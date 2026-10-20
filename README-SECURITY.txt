# Royal Hair admin-login build

## Render Environment Variables
Set these in the existing Render web service before deploying:
- `ADMIN_PASSWORD`: a long, unique admin password (recommend 20+ characters).
- `SESSION_SECRET`: a cryptographically random secret (at least 32 random bytes; use a different value from every other credential).
- `SECOND_PHONE_NUMBER_ID`: `1277018182169272` (number ending 6688).
- `SECOND_WABA_ID`: `1361076229487113` (WABA for number ending 6688).
- `SECOND_WHATSAPP_NUMBER`: `+90 507 859 66 88` (display label only).

Generate a secret locally with:
`node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"`

Keep existing variables unchanged, including `DATABASE_URL`, `WHATSAPP_TOKEN`, `PHONE_NUMBER_ID`, `WABA_ID`, `VERIFY_TOKEN`, `BITRIX_WEBHOOK_URL`, `GRAPH_API_VERSION`, and any other existing configuration. Do not put secrets in source files. The existing `WHATSAPP_TOKEN` must have permission to send through both WABAs. The second-number IDs are prefilled as defaults in code, but setting the three `SECOND_*` variables in Render makes the configuration explicit.

## Deployment
1. Save/export the current Render environment configuration securely.
2. Add `ADMIN_PASSWORD` and `SESSION_SECRET` in Render > service > Environment.
3. Deploy this ZIP's files to the existing app repository/service, preserving the project structure: `app.js`, `package.json`, `public/index.html`, `public/login.html`.
4. Confirm the app opens at `/login`, correct password opens dashboard, incorrect password is rejected, and protected `/api/*` routes return 401 when logged out.
5. Verify inbound/outbound WhatsApp and Bitrix sync. Meta `/webhook` remains reachable for platform callbacks.
6. Test logout, then redeploy/rollback if any integration fails.

## Notes
- This implements one shared admin password, not individual users.
- Session lifetime is 12 hours. Cookie is HttpOnly and SameSite=Strict, with Secure enabled when `NODE_ENV=production`.
- The app still needs its existing Meta webhook verification configured. Do not disable webhook validation.
- Changing the Render hostname is a separate Render setting; this package secures the app but does not rename the `onrender.com` hostname.
- The app has not been live-tested against your Render, Meta or Bitrix accounts. Run the checks above before using it with live leads.


## Dual WhatsApp account setup
- The top-right number selector switches between the existing 5788 account (`primary`) and 6688 (`secondary`).
- Outgoing text, templates and media use the selected account's Phone Number ID. Template sync fetches templates from the selected WABA.
- Conversations and message history are scoped by account. Bitrix leads remain shared.
- Incoming webhook events are routed using `metadata.phone_number_id`.
- IMPORTANT: In Meta, subscribe the same app to BOTH WABAs. The existing callback URL and verify token stay the same. Verify each WABA's `/{WABA_ID}/subscribed_apps` includes your app before testing inbound messages.
- If the System User token does not actually have access to the second WABA, API calls for 6688 will fail; do not replace the existing token until permissions are checked.
- Test one outbound template and one inbound reply per number before using broadcasts. The code has passed local syntax checks but has not been live-tested against Meta or Bitrix.
