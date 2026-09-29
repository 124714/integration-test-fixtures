# Integration test fixtures

Public, synthetic HTTP fixtures shared by local development projects. They let a
device test observe an externally changed source without embedding a mutable test
stand into an application.

## Appointment availability

`appointment-availability/current.json` is the mutable state consumed by debug
builds. Every entity uses a synthetic stable identifier; no patient data, API
keys, credentials or copied production responses are allowed.

To run the baseline-first notification scenario:

1. Copy `scenarios/empty.json` to `current.json`, commit and push.
2. Let the application complete one silent baseline poll.
3. Copy `scenarios/slot-appeared.json` to `current.json`, commit and push.
4. Leave the device in the background and wait for its configured polling alarm.
5. Confirm exactly one audible new-slot notification.
6. Leave `current.json` unchanged and confirm later polls stay silent.

Consumers should request the file through the GitHub Contents API with
`Accept: application/vnd.github.raw+json` and `Cache-Control: no-cache`. The
unauthenticated API is intended for short functional checks, not load tests.

The `scenarios/many-slots.json` scenario contains two departments, eight
synthetic doctors and 131 slots for one doctor in `Центр здоровья`. It exercises
the complete department → speciality → doctor → slots traversal and the large
schedule UI. `current.json` currently publishes that scenario. Copy
`scenarios/empty.json` to `current.json` before running the baseline-first
notification sequence above; publishing a populated scenario can otherwise
create a new-slot alert for an existing watch.
