# Authentik TOTP MFA Lab (Beginner)

In this lab I added a second step to logging in to Authentik. After typing their password, users also have to type a 6-digit code from an app on their phone. If someone hasn't set this up yet, Authentik shows them a QR code to scan before they can finish logging in.

> I built this in my own practice lab, not on a real company system.

## Contents

- [Key terms](#key-terms)
- [The problem and the fix](#the-problem-and-the-fix)
- [What I used](#what-i-used)
- [How the login works](#how-the-login-works)
- [Build steps](#build-steps)
- [Testing](#testing)
- [Screenshots](#screenshots)
- [What I'd improve for a real company](#what-id-improve-for-a-real-company)

## Key terms

- **Authentik:** a free tool that handles logins for your apps. It's called an *identity provider*.
- **MFA (multi-factor authentication):** logging in with more than one thing. Here that's something you know (your password) plus something you have (your phone).
- **TOTP (time-based one-time password):** a 6-digit code made by an app on your phone. It changes every 30 seconds, so an old code stops working.
- **Authenticator app:** the phone app that makes the codes. I used Google Authenticator.
- **Flow:** the list of steps Authentik walks a user through, like logging in.
- **Stage:** one step inside a flow, such as "enter your password".
- **Stage binding:** the setting that puts a stage into a flow and decides its order.

## The problem and the fix

**Problem:** Logging in to Authentik only needed a password. If someone guessed, phished or leaked a password, that was enough to get into the account.

**Fix:** I added a second step. After the password, users now type a 6-digit code from the authenticator app on their phone. A stolen password on its own is no longer enough to log in.

## What I used

| What | Choice |
|---|---|
| Identity provider | Authentik |
| MFA method | TOTP: 6-digit codes that change every 30 seconds |
| Authenticator app | Google Authenticator |

## How the login works

Authentik's login flow runs these stages in order:

```mermaid
flowchart LR
    A[Enter username] --> B[Enter password]
    B --> C{Has the user set up MFA?}
    C -- "Yes" --> E[Enter 6-digit code<br/>then logged in]
    C -- "No" --> D[Scan QR code<br/>to set up MFA]
    D --> E
```

The code check happens after the password and before the user is logged in.

## Build steps

1. **Create the TOTP setup stage.** This is the stage that shows the QR code and saves the user's phone as their authenticator. I set it to 6-digit codes.
2. **Set up the Authenticator Validation stage** in the login flow. This is the stage that checks the code. I chose:
   - Device classes: **TOTP** (accept codes from an authenticator app)
   - Not configured action: **Force the user to configure an authenticator** (anyone without MFA has to set it up)
   - Configuration stages: the TOTP setup stage from step 1 (where to send them to set it up)
3. **Check the stage bindings** so the code check comes after the password and before the user is logged in.
4. **Create a test user** (Kate) and give her a password.

## Testing

| What I tested | What happened |
|---|---|
| Kate logs in for the first time, in a private browser window | After her password, Authentik showed a QR code. I scanned it with the authenticator app, typed the code and reached her dashboard. |
| Kate logs in again | Authentik asked for a code instead of showing the QR code. |
| Typing a wrong or expired code | Authentik rejected it. |

## Screenshots

A walkthrough of the build and test, in order.

**Figure 1:** Setting up the Authenticator Validation stage: TOTP as the device class, and users without MFA are forced to set up an authenticator.

<img width="780" height="570" alt="Figure 1" src="https://github.com/user-attachments/assets/1006df37-c948-42f9-896a-edcf91edc101" />

**Figure 2:** The login flow that the validation stage is added to.

<img width="975" height="484" alt="Figure 2" src="https://github.com/user-attachments/assets/99c8c2c2-189b-42f9-a475-0359912020d6" />

**Figure 3:** The stage bindings, showing the code check runs after the password and before the user is logged in.

<img width="975" height="352" alt="Figure 3" src="https://github.com/user-attachments/assets/dc8aa9a1-4781-4ada-a5d7-57fc0be4582a" />

**Figure 4:** The list of users in Authentik.

<img width="975" height="262" alt="Figure 4" src="https://github.com/user-attachments/assets/a68f54d2-55ed-4308-a5c8-4629e663a82b" />

**Figure 5:** The Authentik login page.

<img width="975" height="564" alt="Figure 5" src="https://github.com/user-attachments/assets/808c7f6a-361c-4f69-ad56-e7b734f49648" />

**Figure 6:** Selecting the test user, Kate, to sign in with.

<img width="975" height="576" alt="Figure 6" src="https://github.com/user-attachments/assets/b29ffb94-1ef4-40a1-a2f0-daab7ad4601f" />

**Figure 7:** Because Kate didn't have MFA set up, after her password she's shown a QR code to set up the authentication factor.

<img width="700" height="635" alt="Figure 7" src="https://github.com/user-attachments/assets/84fa199d-fb45-4b20-8f83-9eddbb81bc0c" />

**Figure 8:** Kate's own account (user side) after she logged in, showing her MFA is now set up.

<img width="975" height="221" alt="Figure 8" src="https://github.com/user-attachments/assets/f912cd3f-2845-4b73-9dee-858fd341bf6c" />

**Figure 9:** The admin side, confirming MFA has been set up for Kate, along with the event log.

<img width="975" height="187" alt="Figure 9" src="https://github.com/user-attachments/assets/31befb3c-bf07-4727-9a96-341695f37eb9" />

**Figure 10:** Now that MFA is set up, signing in asks for the one-time password (OTP) instead of the QR code.

<img width="740" height="609" alt="Figure 10" src="https://github.com/user-attachments/assets/43f0cd55-e081-4027-beb2-3f3366c44dc2" />

**Figure 11:** After entering the OTP, Kate is logged in.

<img width="975" height="289" alt="Figure 11" src="https://github.com/user-attachments/assets/1cc65545-2396-47ab-9e95-53d086397ec2" />

## What I'd improve for a real company

- **Stronger MFA:** TOTP codes can still be stolen by a fake login page (phishing). I'd offer passkeys or security keys (YubiKey, Windows Hello, Touch ID), which can't be phished that way, and make admins use them.
- **Monitoring:** send Authentik's events, like failed code attempts and new phones being set up, to a SIEM (a tool that collects security logs) and get alerts when there's a sudden jump.
