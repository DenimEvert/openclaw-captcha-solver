# Usage Examples for OpenClaw-CapSolver Integration

This document provides practical examples of how to instruct your OpenClaw AI agent to interact with web pages protected by CAPTCHAs, leveraging the CapSolver extension without explicit CAPTCHA-related commands.

## The Golden Rule

> **Do not explicitly mention CAPTCHAs or CapSolver to the AI agent.** The extension handles all CAPTCHA resolution in the background. Your primary instruction should be to include a **sufficient wait time** before any form submission or critical interaction, allowing the extension to complete its task.

## Recommended Wait Times

To ensure successful CAPTCHA resolution, consider the typical solve times for various CAPTCHA types and incorporate a buffer into your instructions:

| CAPTCHA Type | Typical Solve Time | Recommended Wait |
|---|---|---|
| reCAPTCHA v2 (Checkbox) | 5-15 seconds | 30-60 seconds |
| reCAPTCHA v2 (Invisible) | 5-15 seconds | 30 seconds |
| reCAPTCHA v3 | 3-10 seconds | 20-30 seconds |
| Cloudflare Turnstile | 3-10 seconds | 20-30 seconds |

**Tip**: When in doubt, a 60-second wait is a safe and reliable choice. It provides ample time for resolution without negatively impacting the overall process.

## Example Scenarios

Here are several common scenarios demonstrating how to phrase your instructions to OpenClaw:

### Example 1: Basic Form Submission with CAPTCHA

Instruct your OpenClaw agent to navigate to a page, wait for CAPTCHA resolution, and then submit a form. The AI will not be explicitly told about the CAPTCHA.

```
Go to https://example.com, wait 60 seconds,
then click Submit and tell me what text appears on the page.
```

**Behind the Scenes:**
1.  The OpenClaw agent navigates to `https://example.com`.
2.  The CapSolver content script detects the reCAPTCHA widget on the page.
3.  The CapSolver extension calls the CapSolver API to solve the challenge (typically within 10-20 seconds).
4.  The resolved token is automatically injected into the hidden form field.
5.  After the instructed 60-second wait, the AI agent proceeds to click the "Submit" button.
6.  The form submits successfully, and the agent reports the page content.

### Example 2: Logging into a Protected Website

This example demonstrates how to log into a website that might have a CAPTCHA on its login page, ensuring the agent waits before attempting to sign in.

```
Go to https://example.com/login, fill in the email field with
"me@example.com" and the password with "mypassword123",
then wait 30 seconds and click the Sign In button.
Tell me what page loads after signing in.
```

### Example 3: Submitting a Contact Form Behind Cloudflare Turnstile

Here, the agent fills out a contact form and waits for the Turnstile challenge to be resolved before sending the message.

```
Open https://example.com/contact, fill in the contact form:
- Name: "John Doe"
- Email: "john@example.com"
- Message: "Hello, I have a question about your services."
Wait 45 seconds, then click Send Message. What confirmation appears?
```

## Effective Natural Language Patterns

These phrasings have proven effective for guiding the AI agent:

-   *"Go to [URL], wait 60 seconds, then submit the form"*
-   *"Navigate to [URL], fill in [fields], wait 30 seconds, then click [button]"*
-   *"Open [URL] and after about a minute, click Submit and tell me the result"*
-   *"Visit [URL], wait a moment for the page to fully load, then submit"*

## What to Avoid Saying

Avoid these types of instructions, as they can confuse the AI or trigger unintended behaviors:

-   ~~"Wait for the CAPTCHA to be solved"~~ (The AI does not have a concept of CAPTCHAs).
-   ~~"Use CapSolver to solve the verification"~~ (The AI cannot directly control browser extensions).
-   ~~"Click the reCAPTCHA checkbox"~~ (The extension handles this automatically; manual interaction by the AI might interfere).
