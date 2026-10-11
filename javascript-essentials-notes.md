# JavaScript Essentials: Personal Learning Notes

## Introduction

I studied the JavaScript Essentials room on TryHackMe to understand the basic role of JavaScript in web applications and how JavaScript code can affect browser behaviour. The material introduced variables, data types, functions, loops, conditional statements, the request and response cycle, ways to integrate JavaScript into HTML, browser dialogue functions, client-side authentication weaknesses, minification, obfuscation, and several JavaScript security practices.

These notes document the concepts and demonstrations described in the supplied walkthrough. They distinguish between what I learned conceptually and what the walkthrough demonstrated in its examples.

## JavaScript and the Web Front End

JavaScript is a scripting language commonly used to make web pages interactive. HTML provides the structure of a page, CSS controls its presentation, and JavaScript can respond to user actions and change page behaviour. The three technologies can work together to create the front end of a web application.

The walkthrough describes JavaScript as an interpreted language because browser JavaScript can be executed by the browser's JavaScript engine without the developer first compiling the source into a standalone executable. Modern JavaScript engines may use internal compilation techniques, but the important introductory point is that browser JavaScript can be run directly in the browser environment.

## Variables and Basic Data Types

A variable is a named place in which a program can hold a value. The walkthrough introduced three JavaScript declarations: `var`, `let`, and `const`.

The `var` declaration is function-scoped, while `let` and `const` are block-scoped. A block is a section of code enclosed in braces, such as the body of an `if` statement or a loop. Function scope means that a `var` declaration belongs to the containing function rather than being limited to an individual block. In modern JavaScript, `let` and `const` are generally preferred when their behaviour fits the intended use.

I learned that `let` is appropriate when a variable may need to be reassigned. `const` is used when the variable binding should not be reassigned. A `const` variable can still refer to an object whose properties may change, so `const` does not automatically make every value inside an object immutable.

The walkthrough focused on three basic data types. A string represents text, such as a person's name or a city. A number represents numeric values, such as an age or a year. A Boolean represents one of two values, `true` or `false`. It also mentioned `null`, `undefined`, and objects, but the main introductory focus was on strings, numbers, and Booleans.

## Functions

A function is a reusable block of code that performs a particular task. Instead of writing the same instructions repeatedly, I can define a function once and call it wherever that behaviour is needed.

The example in the walkthrough used a function named `printResult` that accepted a roll number as a parameter and displayed a message containing that number. The parameter is the name used in the function definition to receive an input value. When the function is called, an argument is supplied for that parameter. For example, passing `7` allows the function to use that value in the message it displays.

A function definition establishes the instructions to be performed. Calling the function executes those instructions with the supplied arguments. The walkthrough also showed a loop calling a function repeatedly, connecting the concept of reusable functions with repetition in a program.

## Loops and Repetition

A loop repeats a section of code according to a condition or a defined iteration rule. Loops are useful when the same operation needs to be performed many times, such as processing a set of values. Without a loop, a programmer might have to write the same instruction repeatedly.

The walkthrough introduced a `for` loop with a counter variable named `i`. The example began with `i` set to `0`, continued while `i` was less than `100`, and increased `i` by one after each iteration. This setup produces one hundred iterations, using values from `0` through `99`. During each iteration, the example called a function.

A loop does not necessarily continue forever. Its condition and update expression determine when it stops. A loop such as the one in the example stops when the condition `i < 100` becomes false.

## The HTTP Request and Response Cycle

The request and response cycle describes communication between a client and a server. In the web examples, the browser is the client, and the server hosts the web application or provides the requested resources.

When I use a browser to request a website or a resource, the browser sends a request to the server. The server processes the request and sends back a response. The browser can then use the response to display the page or continue requesting other resources. This back-and-forth communication is a basic part of how websites work.

The walkthrough introduced this concept at a high level and referred to the TryHackMe How Websites Work room for further explanation. The supplied material did not document a separate investigation of HTTP traffic.

## Conditional Statements and Program Flow

Control flow describes the order in which instructions execute. Conditional statements allow a program to choose which instructions to run based on whether a condition is true or false.

The walkthrough used an `if` and `else` example based on an `age` variable. If the age met the condition of being at least 18, the script displayed a message saying that the person was an adult. Otherwise, it displayed a message saying that the person was a minor. The result depended on the value stored in the variable.

The walkthrough also mentioned `switch` statements as another way to control program flow, but the detailed example focused on `if` and `else`. Loops and conditional statements serve different purposes. A loop repeats instructions, while a conditional statement selects a path based on a condition.

## Running JavaScript in Browser Developer Tools

The walkthrough demonstrated using the browser's developer tools to run a small JavaScript example. The console can execute JavaScript and display messages produced by the code. It can be opened in Chrome through the browser's developer tools, such as by pressing `F12` or selecting **Inspect** and then opening the Console panel.

The example declared `x` with a value of `5` and `y` with a value of `10`, added the values, and displayed the result using `console.log`. The output was `15`. When `x` was changed to `10`, the addition produced `20`.

The `console.log()` function writes information to the browser's developer console. It does not automatically insert that information into the visible content of the web page. This is different from code that deliberately updates an HTML element.

The walkthrough also showed comments in JavaScript. Comments are notes for people reading the code and are not executed as JavaScript instructions. They can explain the purpose of code or make examples easier to follow.

## Integrating JavaScript into HTML

The walkthrough introduced two ways to include JavaScript in an HTML page: internal JavaScript and external JavaScript.

Internal JavaScript is written directly inside an HTML document between opening and closing `script` tags. The script can be placed in the document's `head` or `body`, depending on when it needs to run and how the page is structured. The walkthrough used an HTML file containing JavaScript that added two numbers and displayed the result.

In that example, JavaScript also used `document.getElementById()` to find an HTML element with a particular `id`. The code then assigned content to that element's `innerHTML` property, allowing the result to appear in the page itself. This differs from `console.log()`, which sends output to the developer console.

External JavaScript is stored in a separate `.js` file. The HTML document refers to that file through the `src` attribute of a `script` element. The `src` attribute identifies the source file that the browser should load. The walkthrough used an HTML file that referenced a file named `script.js`.

Separating JavaScript into an external file can make HTML easier to maintain and allows the same script file to be reused by multiple pages. The walkthrough recommended external JavaScript for typical reuse, while recognizing that internal JavaScript is also possible when appropriate.

The walkthrough demonstrated checking whether JavaScript was internal or external by inspecting the page source. If the JavaScript code appears between the page's `script` tags, it is internal. If the HTML instead references a separate file through `src`, it is external. The example `external test.html` referenced a file named `THM_external.js`.

## JavaScript Dialogue Functions

The walkthrough introduced three browser dialogue functions: `alert()`, `prompt()`, and `confirm()`.

The `alert()` function displays a message in a dialogue box and requires the user to dismiss it. The example `alert("Hello THM")` displayed the message `Hello THM`. I learned that this function is used to show a message rather than collect a text response.

The `prompt()` function displays a dialogue box that asks the user for input. The example stored the value returned by a prompt in a variable named `name`, then displayed a greeting using that value. If the user entered `Bob`, the later alert displayed a greeting containing `Bob`. A prompt can also be cancelled, in which case its return value is `null`, so code should not assume that the user always entered text.

The `confirm()` function displays a dialogue box with options to confirm or cancel an action. It returns a Boolean value: `true` when the user confirms and `false` when the user cancels.

These functions are built into the browser environment and can be tested from the developer console. The walkthrough also demonstrated that scripts can call an alert repeatedly inside a loop. In the `invoice.html` example, the loop caused an alert containing the message `hacked` to appear three times. Repeated dialogue boxes can disrupt a user's interaction with a page. This was a simple demonstration of how code can misuse browser functionality, not evidence of a serious compromise.

## Client-Side Authentication and Trust Boundaries

The walkthrough demonstrated a deliberately insecure login example in which the page requested a username and password using JavaScript prompts. The script compared the entered username and password against values stored directly in the JavaScript. The example used the username `admin` and the password `complex password`.

Because the credential check was implemented in client-side JavaScript, the relevant logic and hard-coded credential could be found by inspecting the page source. This illustrates why client-side code must not be trusted to enforce authentication or protect secrets. Code and data delivered to a browser can be inspected or manipulated by the person using that browser.

This example should not be taken to mean that all login forms can be bypassed in the same way. It demonstrates a specific design flaw: the application exposed the credential and made the authentication decision on the client side. Real authentication must be enforced by the server, which should validate credentials and control access to protected resources.

The walkthrough connected this issue to the wider principle of validating input on both the client and server. Client-side validation can improve usability by giving immediate feedback, but it can be bypassed. Server-side validation is necessary because the server cannot assume that requests or values coming from the browser are trustworthy.

## Minification and Obfuscation

The walkthrough explained minification as reducing the size of JavaScript source code by removing unnecessary whitespace, line breaks, and comments, and sometimes shortening variable names. Minification is mainly intended to reduce file size and can improve delivery performance. It can make source code less readable, but readability reduction is not its primary security purpose.

Obfuscation deliberately transforms code to make its logic more difficult for a person to understand. Depending on the tool, this may involve changing variable or function names and transforming expressions or program structure. Minification and obfuscation are related in that both can make source code harder to read, but they are not the same process.

The walkthrough demonstrated creating a `hello.html` page and a separate `hello.js` file. The JavaScript displayed the message `Welcome to THM`. The script was then processed with an online tool that minified and obfuscated the code. After the transformed code replaced the original contents of `hello.js`, the page continued to display the same message.

This example showed that transforming the source does not necessarily change the program's intended behaviour. The walkthrough also demonstrated sending obfuscated code to a deobfuscation tool to make it easier to inspect. The example included an `age` variable whose value could be identified as `21` after deobfuscation.

Minification and obfuscation should not be treated as substitutes for security controls. They may increase the effort needed to read code, but they do not reliably protect credentials, API keys, access tokens, or other secrets. Anything delivered to the browser should be treated as potentially visible to the user.

## JavaScript Security Practices

The walkthrough concluded with several security practices. Client-side validation should not be the only validation because users can modify client-side code and requests. The server must independently validate input and enforce authorization.

JavaScript libraries should not be included blindly from untrusted sources. A page that loads an external script is relying on that script's behaviour, so developers need to consider whether the source is trustworthy and whether the library has known security problems. Reusing code can be useful, but it also introduces dependencies that must be maintained and reviewed.

Secrets such as passwords, API keys, and access tokens should not be hard-coded into client-side JavaScript. If a secret is included in code delivered to the browser, users may be able to inspect it. Sensitive credentials and security decisions belong on the server side or in an appropriate protected service.

The walkthrough also recommended minifying and obfuscating JavaScript. These techniques may reduce file size or make casual inspection more difficult, but they should not be relied upon to prevent an attacker from understanding or manipulating client-side logic.

## Practical Work Described in the Walkthrough

The supplied walkthrough demonstrated running arithmetic JavaScript in the browser console, creating HTML files with internal JavaScript, creating an HTML file that loaded an external JavaScript file, and examining page source to distinguish between the two approaches. It also showed the browser dialogue functions and a loop that displayed repeated alerts.

The walkthrough further demonstrated a simplified login page with hard-coded client-side credentials and an example of minifying and obfuscating JavaScript while preserving its displayed output. These are the activities described by the source material. I have not added independent test results, target details, or claims of testing real websites beyond those examples.

## What I Learned

I learned the basic role of JavaScript in interactive web pages and how variables, data types, functions, loops, and conditional statements contribute to script behaviour. I also learned how JavaScript can be executed in a browser console and included in HTML either directly or through an external file.

The security examples helped me understand why browser-side code is not a safe place to store secrets or make final authentication decisions. A user can inspect client-side JavaScript, and client-side validation can be bypassed. Server-side validation and access control are therefore essential.

I also learned the difference between minification and obfuscation. Minification primarily reduces code size, while obfuscation deliberately makes code harder to understand. Neither technique should be treated as a replacement for sound application security.

## Conclusion

The JavaScript Essentials room introduced the programming concepts needed to understand simple browser scripts and showed how those concepts relate to web application security. The main security lesson I took from the examples is that code running in the browser can be inspected and manipulated. Sensitive values and important access-control decisions must therefore be protected by server-side controls rather than relying on client-side JavaScript alone.
