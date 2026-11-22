# iOS call-history redial investigation

## Reproduction (reported)
- iOS 27.2.1, Telegram 13.3
- Calling a person inside Telegram works.
- Tapping an earlier Telegram call in the iOS Phone recents opens Telegram at the last viewed screen, but does not start a call.
- Reproduces even when Telegram remains running in the background.

## Confirmed source observations
- `Telegram/SiriIntents/IntentHandler.swift`: `DefaultIntentHandler` implements `INStartCallIntentHandling`.
- `handle(intent: INStartCallIntent, completion:)` requires the first intent contact's `customIdentifier` to start with `tg` and parse into a cloud-user `PeerId`.
- On success, the handler returns `.continueInApp` with `NSUserActivity.userInfo["handle"] = "TGCA<peer-id>"`.
- On failed resolution it returns `.failureRequiringAppLaunch` with no target handle.
- These paths are hypotheses for investigation, not proof of the bug.

## Next trace points
1. Identify where the app receives `NSUserActivity` and parses `TGCA`.
2. Locate CallKit `CXProviderDelegate.provider(_:perform: CXStartCallAction)` and how its handle is resolved.
3. Check whether call-history redial invokes SiriKit, CallKit, or another entry point on the target iOS version.
4. Add privacy-preserving debug traces to distinguish: callback absent, invalid handle, account mismatch, and call initiation failure.
5. Only then implement a behavior change and run build/tests.

## Build and installation constraints
- Official README requires own Telegram API ID and app identity.
- iOS device IPA signing/provisioning remains unsolved without a suitable signing workflow.
- Keep the custom test app separate from the official Telegram installation.
