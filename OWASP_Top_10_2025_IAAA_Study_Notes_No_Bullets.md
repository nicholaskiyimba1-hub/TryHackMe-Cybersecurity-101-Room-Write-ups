# OWASP Top 10:2025 - IAAA and Common Web Application Failures

## Introduction

I completed the TryHackMe OWASP Top 10:2025 room that focuses on three categories related to how applications implement Identity, Authentication, Authorisation, and Accountability, commonly referred to as IAAA.

The three OWASP Top 10:2025 categories covered in this room are A01: Broken Access Control, A07: Authentication Failures, and A09: Logging & Alerting Failures.

I completed practical challenges for these categories. These notes contain the main concepts I learned and are written as a personal study reference for future revision and preparation for the Cybersecurity 101 exam.

## IAAA

IAAA is a simple way for me to understand how applications manage users and their actions. IAAA stands for Identity, Authentication, Authorisation, and Accountability.

These four areas are connected. An application first needs to know which account represents a user or service. It then needs to verify that identity. After authentication, it needs to determine what that identity is allowed to do. Finally, it needs to keep records of important actions so that activity can be investigated later.

## Identity

Identity is the unique account that represents a person or service within an application.

A username, user ID, or email address can be used to represent an identity. For example, an account named `nicholas` can represent my identity within an application.

## Authentication

Authentication is the process of proving that I am the person associated with an identity.

A password, one-time password, passkey, or multi-factor authentication can be used to prove an identity.

The username identifies the account, while the authentication method proves that I should be allowed to use that account.

## Authorisation

Authorisation determines what an authenticated identity is allowed to access or perform.

For example, a normal user may be allowed to view their own profile, while an administrator may be allowed to manage other users.

Authentication answers the question, "Who am I?"

Authorisation answers the question, "What am I allowed to do?"

## Accountability

Accountability means keeping records that allow an organisation to determine who performed an action, what action was performed, when it happened, and where the activity came from.

Logging is an important part of accountability.

For example, an application could record that the user `admin` logged in successfully at a particular time from a particular IP address. This information can later help security teams investigate suspicious activity.

## The IAAA Flow

I can remember the basic flow as Identity, Authentication, Authorisation, and Accountability.

The stages depend on each other. An application cannot properly authorise a user if it has not established and authenticated the user's identity.

# A01: Broken Access Control

## What is Broken Access Control?

Broken Access Control happens when an application does not properly enforce who is allowed to access a resource or perform an action.

The important point I learned is that access control must be enforced by the server.

I should not assume that an application is secure simply because the user interface does not show an option. A user can sometimes manipulate requests directly and attempt to access resources or functions that should not be available to them.

## Insecure Direct Object Reference

A common example of Broken Access Control is an Insecure Direct Object Reference, commonly called IDOR.

For example, suppose an application uses a URL such as `https://example.com/account?accountID=7`.

If I change the value to `https://example.com/account?accountID=6` and the application gives me another user's account information without checking whether I am authorised to access it, the application has an access control problem.

The problem is not simply that an ID appears in the URL. The problem is that the server trusts the supplied identifier without properly checking whether the current user has permission to access the requested object.

## Horizontal Privilege Escalation

Horizontal privilege escalation happens when I remain at the same general privilege level but gain access to another user's data or resources.

For example, User A may be a normal user and then change an account ID in a request. If User A can now view User B's account, User A has not become an administrator. Instead, User A has accessed another user's resources.

If I can access another user's data without gaining a higher role, this is horizontal privilege escalation.

## Vertical Privilege Escalation

Vertical privilege escalation happens when a user gains access to functionality or privileges that belong to a higher-level role.

For example, a normal user may gain access to an administrator-only page or perform an administrator-only action.

In this situation, the user moves from a lower privilege level to a higher privilege level.

## What I Learned From the Practical Challenge

In the practical challenge, I tested the `accountID` value in the URL.

This helped me understand that an application must not trust identifiers supplied by the client.

The server must check authorisation on every protected request.

The main lesson I learned from A01 is that access control must be enforced server-side for every request that requires authorisation.

# A07: Authentication Failures

## What are Authentication Failures?

Authentication Failures happen when an application cannot reliably verify or maintain the identity of a user.

If authentication is weak, an attacker may be able to log in as another user or cause the application to associate a session with the wrong account.

## Username Enumeration

Username enumeration happens when an application reveals whether a particular username exists.

For example, an application may respond differently when a username does not exist compared with when a username exists but has the wrong password.

This information can help an attacker identify valid accounts.

## Weak or Guessable Passwords

Weak or guessable passwords can make accounts easier to compromise.

Applications should also have protections against repeated login attempts. Rate limiting and account lockout mechanisms can make brute-force attacks more difficult.

## Authentication Logic Flaws

Authentication logic can contain mistakes that allow an attacker to bypass part of the intended authentication or registration process.

A problem can occur when different parts of an application process usernames or account information differently.

## Insecure Sessions and Cookies

Authentication does not end when a user enters a password.

The application must also securely manage the user's session.

Problems with session handling can allow an attacker to hijack a session, reuse an old session, access another user's session, or continue using privileges after an important account change.

## The Authentication Challenge

In the practical challenge, I was given the username `admin`.

The challenge involved registering another username using a different combination of uppercase and lowercase characters, such as `aDmiN`.

The purpose of the challenge was to demonstrate how inconsistent handling of usernames can create an authentication problem.

If different parts of an application handle `admin` and `aDmiN` differently, an attacker may be able to interfere with account handling or access the wrong account.

## Authentication Protection

I learned that applications should use a consistent canonical form for usernames and enforce uniqueness using that canonical form.

Applications should also rate-limit repeated login attempts and use appropriate protections against brute-force attacks.

Session management should be secure, and sessions should be rotated after important security changes such as password or privilege changes.

The main lesson I learned from A07 is that authentication must reliably establish and maintain the correct identity throughout the user's session.

# A09: Logging & Alerting Failures

## What are Logging & Alerting Failures?

Logging & Alerting Failures occur when an application does not properly record security-relevant events or does not generate useful alerts when suspicious activity occurs.

This is closely connected to accountability.

If an attack happens but important events were not recorded, investigators may not be able to determine who performed the action, what happened, when it happened, where it came from, or how the attacker moved through the application.

## Logging Failures

Logging failures can occur when authentication events are missing, failed login attempts are not recorded, error messages are too vague, privilege changes are not logged, logs are retained for too short a period, or attackers can modify or delete the logs.

Important security events should be recorded in a way that provides enough information to understand what happened.

## Security Events

A useful security log can contain successful authentication events, failed authentication attempts, password changes, multi-factor authentication changes, account changes, role or privilege changes, and important administrative actions.

The purpose is to create enough evidence to reconstruct security activity.

A useful way for me to remember this is to ask who performed the action, what they did, when they did it, and where the activity came from.

## Centralised Logging

Security logs should not simply remain on the same system that is being attacked.

Centralised logging can help protect evidence and make it easier for security teams to monitor activity across multiple systems.

If an attacker compromises a system and can modify local logs, important evidence may be lost. Centralising logs can reduce this problem and make investigations more reliable.

## Alerting

Logging records events, while alerting helps bring important events to the attention of defenders.

An application or security monitoring system can generate alerts when it detects suspicious behaviour such as a large number of failed login attempts, possible brute-force activity, sudden privilege elevation, suspicious administrator activity, or unusual authentication behaviour.

## Investigation

In the practical challenge, I investigated application logs to understand an attack.

The exercise showed me why complete logs are important.

If important parts of a log are missing, it becomes much harder to reconstruct an attack and determine what actually happened.

The main lesson I learned from A09 is that logging provides evidence, while alerting helps defenders detect and respond to suspicious activity.

# Relationship Between the Three Categories

The three categories covered in this room are closely connected.

A07 Authentication Failures relates to establishing and maintaining the correct answer to the question, "Who is this user?"

A01 Broken Access Control relates to enforcing the correct answer to the question, "What is this user allowed to access or do?"

A09 Logging & Alerting Failures relates to answering the question, "What did this user do, when did they do it, and can we detect and investigate it?"

I can remember the relationship as A07 asking who the user is, A01 determining what the user is allowed to do, and A09 providing evidence about what the user did.

# Important Security Concepts

## Client-Side Trust

I should not trust security decisions that are made only by the client.

A user can manipulate requests before they reach the server.

Important security checks therefore need to be performed on the server.

## Server-Side Authorisation

The server should verify that the authenticated user has permission to access the requested resource or perform the requested action.

This check should happen for every protected request.

## Canonicalisation

Canonicalisation means converting input into a consistent form before making security decisions.

This is important when applications process usernames and other identifiers.

For example, an application needs a consistent policy for how it treats `admin` and `aDmiN`.

If different parts of the application handle them differently, security problems can occur.

## Session Management

Authentication is not only about checking a password.

The application must also securely manage the authenticated session.

Session security includes protecting sessions from hijacking, managing session expiration, rotating sessions after important security changes, and securely handling authentication cookies.

## Security Monitoring

A secure application should not only attempt to prevent attacks.

It should also provide useful evidence when attacks occur.

This requires effective logging, monitoring, alerting, and investigation.

# Quick Revision

## IAAA

Identity answers the question of which account represents the user or service.

Authentication answers the question of how the application proves that the user is the owner of that identity.

Authorisation answers the question of what the authenticated identity is allowed to access or perform.

Accountability answers the question of whether the organisation can determine who did what, when, and from where.

## A01: Broken Access Control

Broken Access Control occurs when an application fails to properly enforce what a user is allowed to access or perform.

IDOR is a common example where changing an object identifier can expose another user's resource because the server fails to perform a proper authorisation check.

Horizontal privilege escalation occurs when a user accesses another user's resources while remaining at the same general privilege level.

Vertical privilege escalation occurs when a user gains access to higher-level privileges or functionality.

The key question for A01 is whether the user can access something they are not authorised to access.

## A07: Authentication Failures

Authentication Failures occur when an application cannot reliably establish or maintain a user's identity.

Username enumeration can reveal which usernames exist.

Weak passwords can make accounts easier to compromise.

Missing rate limiting can make repeated login attempts easier for attackers.

Authentication logic flaws can cause the application to process identities incorrectly.

Session management problems can allow attackers to misuse authenticated sessions.

Username canonicalisation is important because different representations of the same logical username should be handled consistently.

The key question for A07 is whether the application can reliably prove and maintain the user's identity.

## A09: Logging & Alerting Failures

Logging & Alerting Failures occur when security-relevant activity is not properly recorded or suspicious activity is not detected and reported.

Authentication events, failed authentication attempts, password changes, MFA changes, role changes, privilege changes, and important administrative actions should be logged where appropriate.

Logs should be protected against tampering and retained for an appropriate period.

Centralised logging can make monitoring and investigation more effective.

Alerting helps defenders identify suspicious activity that requires attention.

The key question for A09 is whether there is enough reliable evidence to detect and investigate an attack.

# Exam Preparation

For my Cybersecurity 101 revision, I should be able to explain what IAAA stands for and describe the difference between Identity, Authentication, Authorisation, and Accountability.

I should be able to explain the difference between authentication and authorisation.

I should be able to explain what Broken Access Control is and why access control must be enforced server-side.

I should be able to explain what an IDOR is and understand how changing an object identifier can result in unauthorised access.

I should be able to distinguish horizontal privilege escalation from vertical privilege escalation.

I should be able to explain Authentication Failures and identify problems such as username enumeration, weak passwords, brute-force attacks, authentication logic flaws, and insecure session handling.

I should understand why rate limiting and account lockout or equivalent protections are useful against repeated authentication attempts.

I should understand why username canonicalisation is important.

I should understand why sessions need to be securely managed and why sessions may need to be rotated after important security changes.

I should be able to explain Logging & Alerting Failures and understand why security events need to be recorded.

I should understand the difference between logging and alerting.

I should understand why logs need to be protected from tampering and why centralised logging can help during an investigation.

# Final Takeaways

After completing this room, I understand that secure user management is more than simply creating a login page.

An application needs to correctly establish a user's identity, verify that identity through authentication, enforce the correct authorisation, and maintain accountability through logging and monitoring.

A01 Broken Access Control can allow users to access resources or functions they should not access.

A07 Authentication Failures can allow attackers to authenticate as another user or exploit weaknesses in identity and session handling.

A09 Logging & Alerting Failures can make attacks difficult to detect, investigate, and reconstruct.

The most important principles I want to remember are that I should never trust the client with important security decisions, access control should always be enforced on the server, authentication must reliably establish the correct identity, and security-relevant activity must be logged and monitored.

I should also remember that logs need to contain enough information to understand who performed an action, what they did, when they did it, and where the activity came from.

## Source

These notes are based on the TryHackMe OWASP Top 10:2025 room that I completed.

This document is my personal study reference. It is written in my own words and is intended for future revision and Cybersecurity 101 preparation.
