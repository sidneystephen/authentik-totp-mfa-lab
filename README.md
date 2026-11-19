# Authentik TOTP MFA Lab

In this lab I added a second login factor to Authentik. After entering their password, users must type a 6-digit code from an authenticator app. Users who haven't set up MFA yet are sent to a QR-code enrollment screen before they can finish logging in.

> Built in a personal lab environment, not in production.

## The problem and the fix

**Problem:** Logging in to Authentik only needed a password. If a password was guessed, phished or leaked, that alone was enough to get into the account.

**Fix:** I added a second layer of security. After their password, users now have to enter a 6-digit code from an authenticator app on their phone. A stolen password alone is no longer enough to log in.

## Tech 
| Identity provider | Authentik |
| MFA method | TOTP (time-based one-time password), 6 digits, 30-second codes |
| Authenticator app | Google Authenticator |

## How the login flow works

In Authentik, a *flow* is an ordered list of *stages*, and *stage bindings* set the order. The default login flow runs:

```mermaid
flowchart LR
    A[Identification] --> B[Password]
    B --> C{Authenticator Validation}
    C -- "TOTP device enrolled" --> E[User Login]
    C -- "No device yet" --> D[TOTP Authenticator Setup<br/>QR code]
    D --> E
```

The code check sits after the password and before the user is logged in.

## Build steps

1. **Create the TOTP Authenticator Setup stage.** This stage shows the QR code and saves the user's device (6 digits).
2. **Configured the Authenticator Validation stage** in the login flow:
   - Device classes: **TOTP**
   - Not configured action: **Force the user to configure an authenticator**
   - Configuration stages: the TOTP setup stage from step 1
3. **Check the stage bindings** so the code check runs after the password and before User Login.
4. **Create a test user** (Kate) and set a password.

## Testing

| Test | Result |
|---|---|
| Kate's first sign-in, from a private window | After the password, Authentik showed the QR code. Scanned it with the authenticator app, entered the code and reached the user dashboard. |
| Kate's second sign-in | Asked for a code instead of the QR code. |
| Wrong or expired code | Rejected. |

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

- **Phishing-resistant MFA:** offer WebAuthn/passkeys (YubiKey, Windows Hello, Touch ID) and require them for admins. TOTP codes can still be phished.
- **Monitoring:** send Authentik events (failed MFA attempts, new device enrollments) to a SIEM and alert on spikes.
