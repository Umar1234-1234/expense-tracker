# Smart Expense Tracker PWA

An installable Firebase-backed companion for the Android tracker. The PWA never requests notification-reading permission and does not import notifications from banks or wallets. Android remains the only notification-import client.

## Account setup and devices

Registration asks for a username, password, and recovery email. The recovery email is verified before a device receives access to synced records. Password reset is initiated with the username and delivered to that recovery email. Firebase Authentication keeps the recovery email as its email/password identifier; trusted callable Functions verify username/password with Firebase Identity Toolkit and return a Firebase custom token without revealing the recovery address. The first completed account registration receives Admin status through a transaction in trusted Cloud Functions, not client code.

Each browser installation receives a separate device identity. A new device signs in with its account and consumes an expiring, one-use pairing code from the Admin device. Admin actions to create a pairing code, list linked devices, and revoke a device are callable backend operations. Revocation disables the device record checked by Firestore Security Rules and removes its saved push tokens.

## Sync, offline use, and pushes

Transactions live under a shared Firestore group and update in real time across active linked devices. The web client enables Firestore's persistent local cache so offline edits can queue while the application is available; use this only on a device/browser profile you control because cached finance data remains on that device. Concurrent edits to the same transaction use Firestore's last accepted write. Deletions use tombstones so they propagate to offline Android clients; push text contains no amount or transaction description. Notification permission is requested only after tapping **Enable notifications**.

On iPhone/iPad, install the PWA with Share → Add to Home Screen, open the Home Screen app, then grant notifications from inside it. Apple web push requires a supported iOS/iPadOS version (16.4 or newer). iOS PWA push is for this tracker's own synced alerts only; iOS does not let this PWA read notifications from other apps.

## Firebase deployment (not performed here)

The PWA contains the provided public Firebase Web configuration and VAPID public key. `functions/index.js`, `firestore.rules`, `firebase.json`, and `firestore.indexes.json` provide trusted username authentication, first-Admin assignment, device pairing/revocation, FCM fan-out, Hosting, and Firestore access rules. After installing Firebase CLI and authenticating to the intended account, run `pnpm install && pnpm build`, then `firebase deploy --only hosting,functions,firestore --project my-expanse-tracker-75dfe`. Function invocation and live push require suitable Firebase project plan/quotas, enabled Auth providers, valid Cloud Messaging setup, and an API key usable by Identity Toolkit server requests. Cloud Functions deployment can incur charges; review billing before deployment.

The original Android package ID is `com.umar.expensetracker`; the corrected Android-specific `google-services.json` is stored under the Android app module in the separate Android source project. Android Room-to-Firestore synchronization is wired in the source, but it still requires live Firebase deployment and real-device multi-device validation before treating it as production-ready.

## Development

Node 22 and pnpm 11 are pinned/expected. Run `pnpm install` and `pnpm build`. The PWA project preview runs via the managed WebDev runtime. Publishing is a separate step and has not been performed.
