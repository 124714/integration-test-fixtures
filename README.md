# Integration test fixtures

Public, synthetic HTTP fixtures shared by local development projects. They let a
device test observe an externally changed source without embedding a mutable test
stand into an application.

## Appointment availability

`appointment-availability/current.json` is the mutable state consumed by debug
builds. Every entity uses a synthetic stable identifier; no patient data, API
keys, credentials or copied production responses are allowed.

## JSON format

The complete contract is in [`appointment-availability/schema-v1.json`](appointment-availability/schema-v1.json).
The hierarchy follows the page traversal: department → speciality → doctor →
slots. A minimal document with one available appointment is:

```json
{
  "schemaVersion": 1,
  "revision": 2,
  "updatedAt": "2026-09-28T00:05:00Z",
  "departments": [
    {
      "id": 990001,
      "name": "Тестовая поликлиника",
      "specialities": [
        {
          "id": 990002,
          "name": "Невролог",
          "doctors": [
            {
              "id": 990003,
              "name": "Тестовый врач",
              "bookingType": "1",
              "slots": [{ "date": "2030-01-15", "time": "10:30" }]
            }
          ]
        }
      ]
    }
  ]
}
```

Each slot has exactly two fields: `date` in `YYYY-MM-DD` format and `time` in
24-hour `HH:mm` format. Use `"slots": []` when a doctor has no appointments.
Do not repeat the same date/time for one doctor, use positive IDs, and keep
`bookingType` a non-empty string. Unknown fields are rejected. `revision` and
`updatedAt` describe the fixture version; changing either one alone does not
create a new-slot alert. The first successful observation establishes a silent
baseline; only a later newly added slot can alert.

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
schedule UI. Only `current.json` is read by the app; `scenarios/*.json` are
templates that must be copied and published. Before the baseline-first
notification sequence, publish `scenarios/empty.json`; publishing a populated
scenario can otherwise create a new-slot alert for an existing watch.
