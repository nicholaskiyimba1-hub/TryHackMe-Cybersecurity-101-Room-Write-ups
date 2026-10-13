# Burp Suite Fundamentals: Personal Learning Notes

## Introduction

I studied the Burp Suite Basics room on TryHackMe to understand how Burp Suite supports web application penetration testing. The main idea I took from the room is that Burp Suite acts as an intercepting proxy. It allows me to inspect HTTP traffic between a browser and a web server, and, when appropriate, modify requests before they reach the server or examine responses before they return to the browser.

The room introduced the main Burp Suite tools, the interface and settings, browser traffic routing, target scope, and a guided cross-site scripting example. Some of the screenshots and interface descriptions in the lesson were from an older version of Burp Suite, so the exact layout may differ from the version installed on my machine. I should use the concepts and the current interface together rather than assume every button is in the same place as it appeared in the lesson.

## Burp Suite as an Intercepting Proxy

Burp Suite is a platform used to inspect and test web applications. Its proxy places Burp between the client, such as a browser, and the web server. A browser sends a request through the proxy, Burp can pause or record it, and the request can then be forwarded to the server. The server returns a response through Burp, where it can also be inspected depending on the proxy configuration.

This position between the browser and the server is what makes request manipulation possible. For example, I can inspect a request to understand which path is being requested, which HTTP method is used, what headers are sent, and whether cookies or form data are included. I can then change selected parts of the request and observe how the application responds. These tests should only be performed against systems I own or have explicit permission to assess.

Burp Suite has different editions. The Community Edition provides a useful set of manual testing tools and is suitable for learning the fundamentals. The Professional Edition adds capabilities such as automated web vulnerability scanning and other testing features, with some tools and operations subject to different limits in Community Edition. The Enterprise Edition is designed for organizations that need scheduled or continuous scanning across web applications. Edition features can change between releases, so I should confirm current differences in PortSwigger's official documentation instead of relying on an older comparison.

## The Proxy Tool and HTTP History

The Proxy tool is central to Burp Suite because it provides the traffic interception and inspection workflow. When interception is enabled, a request that matches the active interception rules can pause in the Intercept view. I can inspect it, forward it to its destination, or drop it so that it is not sent onward. Interception can be turned off when I want requests to pass through without stopping, while Burp can still record traffic in HTTP history.

HTTP history stores the HTTP requests and responses observed by Burp. I can select an entry to inspect its request and response, and I can sort or filter the history to find traffic by host, method, URL, or other available fields. This is useful when a website makes many requests in the background and I need to isolate the traffic related to a particular action.

The Proxy area can also show WebSocket messages. WebSockets support ongoing, two-way communication between a client and a server, rather than only the usual request-and-response exchange used by ordinary HTTP interactions. The WebSocket history allows me to inspect messages that Burp has observed over that connection.

The Proxy tool also provides Match and Replace rules. These rules can automatically look for a configured part of a request or response and replace it with another value as traffic passes through Burp. Depending on the configuration and version, rules can also apply to WebSocket messages. This can help with repeatable testing, but I need to understand exactly what a rule changes so that I do not accidentally alter unrelated traffic.

## Repeater for Manual Request Testing

Repeater lets me take one HTTP request and send modified versions of it repeatedly. The workflow shown in the room was to select a request in HTTP history, use the context menu to send it to Repeater, and then work with the copied request in the Repeater tab.

I can change a request's path, headers, cookies, query parameters, or body and send it again. Burp displays the server's response so that I can compare the result with the original behavior. For example, changing a path from a login endpoint to the site's root path can show whether the server returns a different page. Removing required request information can produce an error such as HTTP `400 Bad Request`, which indicates that the server considered the request malformed or incomplete.

Repeater is a manual testing tool. It is useful when I want to change one or a few values at a time and carefully observe the effect. The response can be viewed in different formats, including a readable representation, raw text, hexadecimal data, and a rendered view when supported. The rendered response is helpful for a quick visual check, but it is not a replacement for a full browser and may not display every page exactly as the browser would.

## Intruder for Automated Payload Testing

Intruder automates the process of sending requests with different payload values. Instead of manually editing and sending the same request many times, I can mark a position in the request where a value should be substituted, configure a payload list, and let Burp make the requests.

The room introduced the Sniper attack type as a starting point. In a typical Sniper configuration, Burp tests payload values against one selected payload position at a time. For example, in an authorized lab, a path segment can be marked as the payload position and replaced with candidate directory names. The responses can then be reviewed to identify differences that may indicate which paths exist. A similar workflow can test input handling in a login form, but attempts to guess usernames or passwords should only be conducted in an explicitly authorized environment and within the agreed limits.

The marked payload position matters because it tells Burp exactly which part of the request to change. The payload configuration determines the values to insert. A simple list can be entered or loaded from a file, with each value used as a separate payload. Before starting an attack, I should check the target, request, insertion point, payload source, and rate settings. Automated requests can affect an application, create account lockouts, or generate significant traffic, so I should not run them against systems without permission.

Intruder and Repeater serve different purposes. Repeater is better for focused manual experimentation with a single request. Intruder is useful when the same request needs to be tested systematically with multiple values.

## Decoder for Encoded Data

Web requests sometimes contain values that are encoded rather than immediately readable. Burp's Decoder can encode or decode data in formats such as Base64, URL encoding, and hexadecimal representation. Encoding is a transformation of how data is represented; it does not by itself provide confidentiality or encryption.

A useful workflow is to copy a suspicious or unfamiliar value from a request, decode it to understand its representation, and then make a controlled change if the test requires it. If the application expects the value to be encoded, I can encode the changed value again before placing it back into the request. The correct operation depends on the format actually used by the application.

URL encoding represents certain characters using a percent sign followed by hexadecimal digits. For example, the at sign (`@`) can be represented as `%40`. Burp can convert selected characters to URL-encoded form. I should not assume that every character must always be encoded or that encoding alone will bypass a security control. The important point is to preserve the format expected by the request and observe how the server handles the value.

## Comparer for Identifying Differences

Comparer allows me to compare two selected pieces of data, including requests or responses, and identify where they differ. The room demonstrated comparing by words or by bytes. A word-level comparison is often easier to read when comparing text, while a byte-level comparison can reveal differences in the underlying data representation.

This is useful when I have a baseline request and a modified request, or when two server responses look similar but may contain different values. Comparing them can help identify added, removed, or changed content without relying only on visual inspection. Comparer does not explain why a difference exists; I still need to interpret the difference in the context of the request and the application's behavior.

## Sequencer and Token Randomness

Sequencer is used to assess the quality of randomness in tokens generated by an application. Tokens can be used for purposes such as session management or other application functions. If a security-sensitive token is predictable, an attacker may be able to guess values that should be difficult to predict. The actual risk depends on what the token controls and how the application uses it.

Burp Sequencer can capture a sample of tokens from repeated responses and analyze their distribution and apparent randomness. The analysis can include measures such as effective entropy and character-level or bit-level patterns. Meaningful conclusions require a suitable sample of the actual token being evaluated. A small sample, or a sample taken from a value that is not intended to be unpredictable, is not enough to establish that a token is secure or insecure.

In the room, the instructor showed where live capture and analysis are available, but the example data was acknowledged as insufficient for a meaningful conclusion. I should therefore treat the demonstration as an introduction to the tool, not as evidence that a real session token was proven secure or predictable.

## Extensions and the BApp Store

Burp can be extended with additional functionality. The BApp Store provides extensions that integrate with Burp, and the room showed that extensions can add features that are not part of the core workflow. Examples mentioned in the lesson included integrations for scanning and tools that help identify parameters that may be relevant to web cache poisoning research.

Extensions can improve a testing workflow, but I should review an extension's purpose, compatibility, permissions, source, and maintenance before installing it. An extension should be selected because it addresses a specific testing need, not simply because it is popular. I have learned about the extension mechanism from the room; the notes do not establish that I installed or used any particular extension during a practical test.

## Installation and Interface Navigation

The room explained that Burp Suite can be downloaded for supported operating systems from PortSwigger's official website. On Windows, installation generally uses an installer. On Linux, the downloaded installer can be launched according to the instructions provided for that release. The exact installation steps and paths can vary by package and version, so I should follow the current official instructions rather than assume a path from an older demonstration.

Burp's interface is organized around tools such as Dashboard, Target, Proxy, Repeater, Intruder, Decoder, Comparer, and Sequencer. Each tool can contain additional views or settings. The position of these views has changed between releases, so the same feature may appear as a subtab beneath a tool or in a different part of the interface.

The room introduced keyboard shortcuts for moving between tools. In the demonstrated interface, `Ctrl+Shift+P` opened Proxy, `Ctrl+Shift+R` opened Repeater, `Ctrl+Shift+I` opened Intruder, and `Ctrl+Shift+D` opened Dashboard. Shortcuts can vary by operating system, version, or user configuration, so I should verify them in my own installation. Burp also allows a request to be sent to another tool through its context menu, which is useful if a tool is not visible where I expect it.

The interface includes contextual help in places marked with a question-mark icon. These help panels can explain the currently displayed area or setting. When the interface differs from a screenshot in a learning room, I can use the current interface's help and settings search to locate the equivalent feature.

## User Settings and Project Settings

Burp has settings that apply globally to the user and settings that apply to a particular project or session. User settings generally persist across launches, while project settings apply to the current project context. The exact project persistence options depend on the Burp edition and current version.

The settings area contains categories for different parts of Burp. The room showed examples involving session settings, updates, hot keys, and TLS client certificates. A search field can help locate a setting by name. Rather than memorizing where every option is located, I can use settings search and the relevant tool's settings link.

The distinction between user and project settings matters because changing a global option can affect later testing sessions, while a project-level option is intended to apply to the current testing context. Before changing a setting, I should check which category it belongs to and understand the effect it may have on other work.

## Sending Browser Traffic Through Burp

Burp can only inspect browser traffic that is routed through it. The room introduced FoxyProxy, a browser extension that can configure a browser to use Burp's proxy listener. The settings shown in the lesson were examples of a typical local proxy configuration, but the correct host and port must match the listener configured in the running Burp instance.

The room also demonstrated Burp's built-in browser, opened from the Proxy tool. When available and working, this is a straightforward way to generate traffic through Burp without separately configuring a browser extension. The built-in browser can behave differently across operating systems and installations. On some Linux configurations, browser sandbox or permission issues may prevent it from launching. The lesson mentioned using a suitable low-privilege user account or changing the sandbox setting as possible troubleshooting approaches, but disabling the sandbox reduces isolation and should not be the first choice without understanding the security implications.

If a browser cannot reach HTTPS sites while traffic is being proxied, Burp's certificate authority certificate may need to be installed and trusted in that browser. This allows the browser to trust certificates generated by Burp when Burp intercepts TLS connections. I should only install and trust the certificate in the testing browser or environment where I intend to inspect traffic. I should not disable certificate validation broadly or install a testing CA in environments where it is not needed.

## Target, Site Map, and Scope

The Target tool helps me understand the application I am testing. Its site map organizes hosts and paths that Burp has observed, giving me a navigable view of the application's structure. By browsing the authorized application and reviewing the resulting site map, I can identify endpoints that I have visited and investigate requests or responses associated with them.

The site map is useful for discovery, but it is not necessarily a complete list of every endpoint in the application. It reflects the traffic Burp has observed and other information available to it. A path that does not appear in the site map may still exist if I have not discovered or requested it.

Issue definitions provide descriptions of issue types that Burp can identify, including explanations and references that may help me understand a vulnerability and its remediation. Automated scanning capabilities depend on the edition and configuration, so the presence of an issue definition does not mean that Community Edition will automatically detect every issue described there.

Scope defines which hosts, URLs, or other target elements are included in the assessment. I can add an authorized target to scope and choose whether relevant subdomains are included. I can also exclude hosts or paths that are outside the permitted assessment. Scope is particularly important during a real penetration test or bug bounty assessment because the rules of engagement determine which systems may be tested.

Scope and interception are related but distinct. Scope describes the target boundary for Burp's target-related functions and filtering. Whether out-of-scope traffic is intercepted or logged depends on the applicable proxy settings and configuration. I should test the behavior in my own version rather than assume that adding a host to scope automatically prevents every out-of-scope request from being intercepted. Before beginning a test, I should configure the scope carefully and confirm that the traffic and actions remain within the authorization I have.

## Guided Cross-Site Scripting Demonstration

The room concluded with a guided example involving a support form and cross-site scripting (XSS). XSS occurs when an application includes untrusted input in a page in a way that causes a browser to interpret the input as executable script. The exact impact depends on the context in which the input is inserted and the browser's handling of the resulting page.

In the demonstration, the form applied a client-side filter that blocked an input before it was submitted normally. The instructor enabled Burp interception, submitted an ordinary-looking form value, and then modified the captured request before forwarding it. This showed why client-side validation is not a sufficient security boundary: a user can alter requests outside the normal page interface, including by using an intercepting proxy.

The request also showed URL encoding in a submitted value. For example, an at sign in an email address appeared as `%40`. Burp can encode selected characters when preparing a request. The lesson then demonstrated that changing the request in Burp caused the lab page to display a JavaScript alert, illustrating the weakness in relying on a client-side filter alone.

This was a guided lab demonstration, not an independent assessment of a production application. A robust defense cannot rely only on browser-side filters. Applications should validate input on the server where appropriate and, most importantly for XSS prevention, apply context-appropriate output encoding or safe rendering methods so that untrusted data is treated as data rather than executable code. A filter bypass in one demonstration does not mean every XSS defense can be bypassed in the same way. I should only reproduce such tests in an authorized lab or other explicitly permitted environment.

## What I Learned

I learned that Burp Suite is most useful when I understand the HTTP traffic behind a browser action. The Proxy lets me intercept and record traffic, HTTP history helps me find the relevant request, and Repeater allows me to change that request manually and observe the response. Intruder automates repeated tests with different payloads, while Decoder and Comparer help me interpret encoded values and identify differences. Sequencer is intended for evaluating token randomness, and extensions can add capabilities to the platform.

I also learned that the Target site map and scope serve different purposes. The site map helps me inspect the structure revealed by observed traffic, while scope defines the systems that belong in the assessment. Correct browser routing and, when necessary, certificate configuration are prerequisites for inspecting the traffic I intend to test.

The room's interface demonstrations included older layouts, and the Sequencer example did not produce enough meaningful data to support a security conclusion. I should keep these limitations in mind when revisiting the notes. Knowing where a feature is located is only the starting point; I still need to understand what its output means and avoid making claims that the evidence does not support.

## Conclusion

This room introduced me to the main Burp Suite workflow for web application testing. I now understand how an intercepting proxy provides visibility into HTTP requests and responses, how the main tools support different forms of analysis, and why target scope matters when testing an application. The guided XSS example also showed that client-side filtering alone cannot protect a web application from a user who can modify the request.

My next step is to repeat the basic workflow in an authorized lab: route browser traffic through Burp, locate a request in HTTP history, send it to Repeater, make one controlled change, and compare the response. I should focus on explaining what I observe rather than simply repeating a sequence of clicks.
