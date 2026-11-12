# Authentik TOTP MFA Lab

## What it does

Adds a second login factor to Authentik. After the password, users have to type a 6-digit code from an authenticator app (Google Authenticator, Microsoft Authenticator, Authy, etc.). Users who haven't set up MFA yet get sent to a QR-code enrollment screen before they can finish logging in.

## What it runs on

- **Identity provider:** Authentik
- **MFA method:** TOTP (time-based one-time passwords, RFC 6238), 6 digits, 30-second codes
- **Authenticator app:** Authenticator app

## How I built it

1. **Learned how Authentik's flows and stages fit together.** A *flow* is an ordered list of *stages*, and *stage bindings* set the order. The default login flow goes Identification → Password → Authenticator Validation → User Login.
2. **Set up the TOTP Authenticator Setup stage.** This is the stage that shows the QR code and saves the user's device (6 digits).
3. **Configured the Authenticator Validation stage** in the login flow:
   - Device classes: **TOTP**
   - Not configured action: **Force the user to configure an authenticator**
   - Configuration stages: the TOTP setup stage from step 2
4. **Checked the stage bindings** so the code check runs after the password and before the user is logged in.
5. **Created a test user** (`mfa-testuser`) and set a password.
6. **Signed in as that user from a private window.** After the password, Authentik showed the QR code. I scanned it with the authenticator app, entered the code, and reached the user dashboard.
7. **Signed in again** and was asked for a code instead of the QR code. I confirmed that a wrong or expired code is rejected.

## The thing I fixed

**Problem:** Logging in to Authentik only needed a password. If a password was guessed, phished or leaked, that alone was enough to get into the account.

**Fix:** I added a second layer of security. Users now have to enter a 6-digit code from an authenticator app on their phone after their password. A stolen password alone is no longer enough to log in.

## What I'd improve for a real company

- **Phishing-resistant MFA:** offer WebAuthn/passkeys (YubiKey, Windows Hello, Touch ID) and require them for admins. TOTP codes can still be phished.
- **Monitoring:** send Authentik events (failed MFA attempts, new device enrollments) to a SIEM and alert on spikes.

## Where I built it

In a personal lab environment, not in production.

See the process of the steps taken above.

<img width="780" height="570" alt="Picture2" src="https://github.com/user-attachments/assets/1006df37-c948-42f9-896a-edcf91edc101" />

<img width="975" height="484" alt="image" src="https://github.com/user-attachments/assets/99c8c2c2-189b-42f9-a475-0359912020d6" />

<img width="975" height="352" alt="image" src="https://github.com/user-attachments/assets/dc8aa9a1-4781-4ada-a5d7-57fc0be4582a" />

<img width="975" height="262" alt="image" src="https://github.com/user-attachments/assets/a68f54d2-55ed-4308-a5c8-4629e663a82b" />

<img width="975" height="564" alt="image" src="https://github.com/user-attachments/assets/808c7f6a-361c-4f69-ad56-e7b734f49648" />

<img width="975" height="576" alt="image" src="https://github.com/user-attachments/assets/b29ffb94-1ef4-40a1-a2f0-daab7ad4601f" />

<img width="700" height="635" alt="Picture8" src="https://github.com/user-attachments/assets/84fa199d-fb45-4b20-8f83-9eddbb81bc0c" />

<img width="975" height="221" alt="image" src="https://github.com/user-attachments/assets/f912cd3f-2845-4b73-9dee-858fd341bf6c" />

<img width="975" height="187" alt="image" src="https://github.com/user-attachments/assets/31befb3c-bf07-4727-9a96-341695f37eb9" />

<img width="740" height="609" alt="image" src="https://github.com/user-attachments/assets/43f0cd55-e081-4027-beb2-3f3366c44dc2" />

<img width="975" height="289" alt="image" src="https://github.com/user-attachments/assets/1cc65545-2396-47ab-9e95-53d086397ec2" />





