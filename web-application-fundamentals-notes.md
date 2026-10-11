# Web Application Fundamentals: Front End and Back End

## Introduction

I started learning how web applications work by looking at their front end and back end. The room introduces HTTP requests and responses, status codes, security headers, and other web application concepts. The material covered in these notes focuses on the basic structure of a web application and the roles of HTML, CSS, JavaScript, and databases.

## Understanding the Front End and Back End

I learned to think of a web application as having two broad parts. The front end is the part that a user sees and interacts with in a browser. The back end contains components that help the application operate, including the database used to store information. The comparison in the lesson was similar to a planet: its surface is visible, while its interior is hidden but helps determine how it functions.

The front end and back end work together. When I visit a website such as TryHackMe, my browser communicates with the website's server to request the resources needed to display a page. The browser then interprets the returned HTML and uses related resources, such as CSS and JavaScript, to present the page and support interactions. The page does not simply appear by magic. It is assembled by the browser from resources supplied by the website.

## HTML: Structuring Web Pages

HTML stands for HyperText Markup Language. I learned that HTML provides the structure and content of a web page. It gives the browser instructions about elements that belong on the page, such as headings, paragraphs, links, and other page components.

When a browser requests a page, the server can return HTML for the browser to interpret. The browser uses that markup to build the page structure that I see. HTML is therefore an important part of the front end, but it is not responsible on its own for every visual style or interactive behaviour on a page.

## CSS: Styling Web Pages

CSS stands for Cascading Style Sheets. I learned that CSS controls much of the presentation of a web page. It can define colours, fonts, text sizes, spacing, and the arrangement of page elements.

HTML establishes the page's structure, while CSS changes how that structure looks. For example, the appearance of buttons, the spacing between sections, and the fonts used for text can be controlled with CSS. This helps explain why a web page can have structured content and also a distinct visual design.

## JavaScript: Adding Interactivity

I learned that JavaScript can make a web page respond to user actions and update what is displayed. It can work with the page's HTML and CSS to create interactive behaviour.

In the lesson's example, selecting a task heading caused the page to open or close content and move the view to the relevant section. JavaScript can be used to implement this kind of behaviour, so the browser can respond to an interaction without requiring the user to perform every step manually.

HTML, CSS, and JavaScript have different but related roles. HTML describes the page structure, CSS controls presentation, and JavaScript adds behaviour and interaction. Together, they form important parts of a web application's front end.

## Inspecting the Front End with Browser Developer Tools

I learned that I can inspect a website's front-end resources using the browser's developer tools. In the lesson, the method shown was to right-click a page and select **Inspect**. The browser then opens its developer tools, where I can examine the page's HTML and CSS and investigate JavaScript associated with the page.

This gives me a way to see parts of how a website is built rather than treating the page as a single finished image. The front-end code and resources delivered to a browser can often be inspected by the person using that browser. However, inspecting the front end does not mean that I can automatically see the website's private back-end code or its database contents.

## Understanding the Back End and Databases

I learned that the back end contains components that help a web application function behind the interface. One example introduced in the lesson is a database, which stores information that the application needs.

The lesson used TryHackMe accounts as an example. A website may need to remember information associated with a user, such as their profile picture, level, or streak. A database can store the information that allows the application to retrieve and display those details when the user signs in.

The database is part of the back end, so its contents are not automatically displayed to every person who visits the website. A user may be able to view some information that the website intentionally makes public, but that does not mean the user can browse every account record or see other users' private information, such as their passwords. Access to information depends on how the application is designed and how access is controlled.

## What I Learned

I learned that a web application is made up of connected components rather than just the page displayed in a browser. The front end includes the parts I can see and interact with. HTML provides structure, CSS controls presentation, and JavaScript supports interactive behaviour. Browser developer tools let me inspect much of the front-end material delivered to my browser.

I also learned that the back end supports the application's operation and can include a database that stores user-related information. The interface may display selected information from that database, but the underlying records are not automatically exposed to every user.

These notes cover the introductory material provided so far. The room's wider learning objectives also mention HTTP requests and responses, status codes, and security headers, but the supplied explanation has not yet covered those topics in detail.
