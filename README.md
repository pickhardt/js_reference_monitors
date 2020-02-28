# JavaScript Reference Monitors


## Intro

This proposal is to add JavaScript Reference Monitors as part of the JavaScript spec, allowing websites or applications to implement [reference monitors](https://en.wikipedia.org/wiki/Reference_monitor).


## Motivation

[Magecart](https://www.google.com/search?q=magecart) and similar supply chain attacks have become an epidemic, successfully stealing credit card numbers, crypto wallet secrets, and other sensitive data directly from websites and Electron apps. This has had a number of high profile victims including British Airways, Macy's, Ticketmaster, Newegg, and IOTA wallets, among others. These attacks work by getting malicious JavaScript into code that runs directly in a website, Electron app, or React Native app, enabling the attacker to steal sensitive data like credit card numbers.

JavaScript Reference Monitors will allow [reference monitors](https://en.wikipedia.org/wiki/Reference_monitor) over the behavior on a website or application with a JavaScript runtime. This could be used for various security, privacy, and compliance reasons:

- to monitor network requests to track, detect, and attempt to prevent data exfiltration attacks like Magecart.

- to monitor network requests to prevent sensitive data from accidentally being sent to analytics trackers, like accidentally sending social security numbers or credit card numbers to Google Analytics.

- to implement a policy restricting content loaded on the page similar to the Content Security Policy header.

- to prevent cookies from being set before the user has given consent (increasingly important thanks to GDPR).

- to prevent or warn the user before navigating away to an untrusted domain.


## Code vs Configuration

One prominent trade-off in software engineering comes when deciding whether something should be done with configuration or with code.

Currently, Content Security Policy (CSP) Headers act as configuration: the website will set a few configuration parameters (their whitelist), then the browser will carry out the security policy based on that configuration. However, CSP headers are fairly brittle, easy to break, hard to set up properly, and don't actually work all that well in practice. Further, preventing data exfiltration is not the objective of CSP headers, despite that being something companies want to do with CSP headers. For example, nearly everyone will whitelist Google Analytics, but an attacker can create their own Google Analytics account to receive custom events as part of a data exfiltration attack.

The JS Reference Monitor API is code, not configuration, and is therefore more powerful and customizable. It is expected that websites would use them either directly themselves, or through third party script providers.


## How It Works

Reference monitors are a known concept used in operating system security to control subjects' ability to perform operations and access objects. This proposal is a way to extend reference monitors to the browser by allowing websites to specify trusted reference monitors along with their website.

To use JavaScript Reference Monitors, a site will:

- Have a new JavaScript Reference Monitor API to interpose various behavior, with four proposed functions:
  
  ReferenceMonitor.interposeNetworkRequests(referenceMonitor)
  
  ReferenceMonitor.interposeCookies(referenceMonitor)
  
  ReferenceMonitor.interposeDOMWrites(referenceMonitor)
   
    This would be able to monitor not just visual elements like divs, but any addition of scripts, stylesheets, etc. added to the page
  
  ReferenceMonitor.interposeNewLocations(referenceMonitor)
    
    This last one would interpose window.location=new from JS or JS-triggered forms/links, and maybe even user-clicked forms/links too, but wouldn't affect a user navigating away by entering their own url

- Optionally, set a HTTP header, Trusted-Reference-Monitor, with comma-separated nonces. For example, setting "d8aIlsPaaZ3X,oa09e98321cQ" would set two nonces.

- In the HTML, inline script tags may optionally contain a the nonce as an attribute, trusted-reference-monitor="d8aIlsPaaZ3X"  If used, this gives the script access to the JavaScript Reference Monitor API.

- If the Trusted-Reference-Monitor HTTP header is not set, then the Reference Monitor API is available to all JavaScript on the page. However, if the site does set the Trusted-Reference-Monitor HTTP header, then that locks the API down so that it is only accessible by those script tags that have the trusted-reference-monitor attribute set to one of the nonces. Each nonce can only be used once, and the browser will error if used more than once.

- Optionally, the spec could require the Trusted-Reference-Monitor header, and disallow reference monitoring without explicitly declaring which reference monitor script to trust.

- A referenceMonitor function should accept an event parameter, where the event object contains:
  - the function that was called
  - the arguments the function was called with
  - the stack the function was called from

- The referenceMonitor function should have the ability to prevent the function from being carried out, either via a return variable or with a mechanism similar to DOM events' event.stopPropagation() The referenceMonitor function can either:
     - allow the function call or object read to proceed as intended without modification
     - modify the arguments and then proceed (e.g. to remove suspicious scripts from being added tot the DOM, remove sensitive data from being set in cookies, etc)
     - block the function call or object read entirely

- To prevent infinite recursion (such as when an interpose network request function wants to make a network request itself), there should be a mechanism to allow the reference monitors' calls to skip reference monitoring.

## Contributing

This is a work-in-progress. To contribute in any way to the discussion, feel free to open an issue or pull request on the project's Github repo.
