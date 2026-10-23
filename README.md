# Temp Mail — GitHub Pages

A single-file, mobile-first temporary inbox frontend using the **Mail.tm** API. It creates actual provider mailboxes, retrieves messages, refreshes automatically, opens messages, marks them read when supported, and can delete a mailbox. It does not insert sample emails.

## Important status and limitations

- This repository is a static frontend. GitHub Pages does **not** run a backend or receive email by itself.
- The frontend calls `https://api.mail.tm` directly. It needs the provider to permit browser requests from the deployed origin (CORS). If the provider blocks browser requests, the site will show a connection error; do not add an unofficial proxy. Check the current provider policy/docs before deployment.
- Mail.tm is a third-party temporary-mail service. Domains, availability, retention, rate limits, and API policy can change. Account creation can be rate-limited. The app only uses a domain returned by the live `/domains` endpoint.
- No provider account, API key, or secret is hard-coded. The provider-issued bearer token and generated mailbox password are held in the current browser tab's `sessionStorage`; anyone with access to that active browser session may be able to use that mailbox. Closing the tab usually clears session storage.
- The API response does not provide a reliable mailbox-expiry timestamp in the documented account object, so the UI deliberately says “Expiry: provider-dependent” instead of inventing an expiry time.
- A successful API call is not proof that external email delivery works in every case. Complete the end-to-end test below before calling the deployment operational.

## Files

- `index.html` — complete static application.

## Deploy to GitHub Pages

1. Create a new GitHub repository, for example `temp-mail`.
2. Upload `index.html` to the repository root and commit it.
3. Open **Settings → Pages**.
4. Under **Build and deployment**, select **Deploy from a branch**.
5. Choose branch `main` and folder `/(root)`, then save.
6. Wait for GitHub Pages to publish the site. Open the published HTTPS URL on Android Chrome.
7. Click **Generate email**. The app retrieves an available public domain, requests a mailbox from Mail.tm, then requests its access token.
8. If the API request fails with a browser network/CORS error, check the browser console and current Mail.tm API policy. GitHub Pages cannot fix CORS. Do not put API secrets into the public HTML and do not proxy Mail.tm if its current terms prohibit proxying.

No Node.js, bundler, build step, or environment file is required for this direct-API version.

## Provider API used

Base URL: `https://api.mail.tm`

The frontend uses the documented API routes:

- `GET /domains` — discover active public domains.
- `POST /accounts` — create a mailbox with a generated address and password.
- `POST /token` — obtain the mailbox bearer token.
- `GET /messages` — list incoming messages.
- `GET /messages/{id}` — retrieve one message.
- `PATCH /messages/{id}` — mark a message as read.
- `DELETE /messages/{id}` — delete a message.
- `DELETE /accounts/{id}` — delete a mailbox.

Documentation: <https://docs.mail.tm/>  
API explorer: <https://api.mail.tm/index.html>

The browser does not require a provider API key. The account password and bearer token are per-mailbox credentials, not application-wide secrets. Never publish or log them.

## End-to-end test checklist

Run these tests on the actual GitHub Pages URL, not only from a local file:

1. Open the site and click **Generate email**.
2. Confirm the address shown came from a successful `/accounts` response and the UI shows **Connected**.
3. From a separate external mailbox you control, send a simple test email to the generated address. Do not send sensitive information.
4. Wait for polling (about 15 seconds) or tap **Refresh inbox**.
5. Confirm the message appears with its real sender, subject, timestamp, and provider preview.
6. Open the message and confirm its body is readable.
7. Generate a second address and confirm its inbox does not show the first mailbox's messages.
8. Click **Delete this mailbox** and confirm the provider accepts deletion. Then reload the page and confirm the deleted mailbox is no longer restored.
9. Test offline/network failure and provider rate limiting; the UI should show an error rather than sample messages.
10. Test on Android Chrome and a desktop browser.
11. Confirm retention/expiry behavior using current provider policy. The app does not claim an expiry timestamp the API has not supplied.

**Do not mark these tests as passed until you have personally completed them.** This project has not been end-to-end tested from your GitHub account or against an external sender as part of generating these files.

## Security notes

- Email HTML is rendered through a conservative allowlist sanitizer; scripts, forms, remote images, embedded content, event handlers, and links are removed.
- Temporary email is not private or guaranteed to persist. Do not use it for banking, sensitive accounts, private correspondence, or important password recovery.
- This is a direct-to-provider browser integration, not a secure multi-user backend. Browser session tokens are accessible to scripts running on the same origin. Keep the page free of untrusted third-party scripts.
- If a future provider requires secret credentials or server-side mailbox access, use a provider-approved backend with server-side environment variables and authenticated per-mailbox sessions. Do not place secrets in GitHub Pages files.
