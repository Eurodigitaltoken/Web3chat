# ADR-001: E2EE SDK selection for Web3 Chat

- Status: **Provisional — no production SDK selected**
- Date: 2026-11-02
- Scope: mobile chat E2EE (Android and iOS)
- Existing application shell: Flutter
- Product constraint: commercial app; chat must work without a wallet

## Context

The current v1.5.2 archive contains a Flutter placeholder and a WebSocket service that sends a field named `encryptedContent`. That field name is not evidence of end-to-end encryption. We must not describe the current chat as E2EE until client-side encryption, key custody, device verification, recovery and server-side ciphertext tests pass.

The Matrix ecosystem offers a mature protocol and client SDKs, but SDK license and platform integration both matter.

## Findings checked on 2026-11-02

1. The official Matrix SDK directory lists:
   - Matrix Rust SDK as **Apache-2.0**, stable.
   - Matrix Dart SDK as **AGPL-3.0-only**, stable.
   Source: https://www.matrix.org/ecosystem/sdks/
2. The Matrix Rust SDK repository says it handles encryption, syncing and room state, is used by Element X on Android and iOS, and is licensed Apache-2.0.
   Source: https://github.com/matrix-org/matrix-rust-sdk
3. The Rust SDK repository documents bindings for Swift and Kotlin (among other environments), but does not document an official Dart/Flutter binding.
   Source: https://github.com/matrix-org/matrix-rust-sdk/tree/main/bindings
4. The Matrix Dart SDK says E2EE requires Vodozemac; its Flutter guidance uses `flutter_vodozemac`.
   Source: https://github.com/famedly/matrix-dart-sdk
5. The current `flutter_vodozemac` package is listed as AGPL-3.0.
   Source: https://pub.dev/packages/flutter_vodozemac

## Decision

**Do not add the Matrix Dart SDK or flutter_vodozemac to the production app at this stage.** Their AGPL licensing needs a written legal review for the intended commercial distribution. This is a risk gate, not a claim that use is categorically impossible.

**Do not replace Flutter yet.** The existing Flutter project is only a shell, but changing frameworks before a technical comparison would be premature.

The next engineering investigation is a disposable, isolated proof of concept for using the Apache-2.0 Matrix Rust SDK from a Flutter application through a small native bridge (Android/Kotlin and iOS/Swift/UniFFI). This is only a candidate: there is no official Dart binding confirmed, so estimate and test the bridge effort before committing to it.

In parallel, compare a TypeScript/React Native route using the Apache-2.0 Matrix JavaScript SDK. Do not assume that a web SDK automatically works correctly in React Native; validate crypto/WASM, secure storage, background sync and push handling on real Android and iOS devices.

## Proof-of-concept acceptance criteria

A prototype is not accepted until all of the following are demonstrated:

1. Two disposable test accounts on a dedicated Matrix homeserver can establish a private encrypted one-to-one room.
2. User A sends a unique test message; user B decrypts it; user B replies; user A decrypts it.
3. The homeserver event/API inspection shows ciphertext, not message plaintext.
4. Device verification and cross-signing are exercised; an untrusted/unknown device is not silently trusted.
5. The app survives force-close, restart, reconnect and encrypted-history retrieval.
6. Key backup and restore are tested independently of the wallet recovery phrase.
7. Clean Android and iOS builds succeed on supported toolchains.
8. Logs, crash reports and push payloads contain no message plaintext, private keys, recovery secrets or backup passphrases.
9. Dependency licenses, transitive dependencies and required notices are reviewed before any production adoption.

## Security boundaries

- Wallet private keys, chat E2EE keys and login/session credentials are separate.
- The backend must never receive wallet private keys or sign wallet transactions.
- The backend must not receive plaintext chat messages or E2EE private keys.
- A wallet seed phrase must never be reused as a chat-encryption key or cloud-backup passphrase.
- Chat remains usable without connecting a wallet.
- The existing `encryptedContent` WebSocket field must not be represented as proof of E2EE.

## What this ADR does not authorize

- No production dependency changes.
- No modification of the v1.5.2 application source.
- No deployment or real-user account creation.
- No claim that E2EE is implemented or independently audited.

## Review trigger

Update this ADR after the native Rust bridge and React Native comparison have been tested, and after legal review of the applicable licenses.
