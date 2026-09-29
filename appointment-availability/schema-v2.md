# Appointment availability schema v2

Version 2 follows the same four stages as the production NMCS source. All
documents reject unknown fields. Every child document repeats `schemaVersion`
and `snapshotId`; the consumer must reject a mismatch.

`current.json` is the only mutable pointer:

```json
{
  "schemaVersion": 2,
  "snapshotId": "20260929t180701z-a1b2c3d4",
  "updatedAt": "2026-09-29T18:07:01Z",
  "departments": [
    {
      "id": 990001,
      "name": "Тестовое подразделение № 1",
      "specialitiesHref": "appointment-availability/snapshots/20260929t180701z-a1b2c3d4/departments/990001/specialities.json"
    }
  ]
}
```

The root always contains exactly the three departments from `catalog-v2.json`.
Each `specialitiesHref` document has this shape:

```json
{
  "schemaVersion": 2,
  "snapshotId": "20260929t180701z-a1b2c3d4",
  "departmentId": 990001,
  "specialities": [
    {
      "id": 990011,
      "name": "Невролог",
      "doctorsHref": "appointment-availability/snapshots/20260929t180701z-a1b2c3d4/departments/990001/specialities/990011/doctors.json"
    }
  ]
}
```

Only a speciality with at least one doctor having at least one slot is included.
Its doctors document contains every predefined doctor for that speciality, so a
doctor may remain visible with an empty schedule:

```json
{
  "schemaVersion": 2,
  "snapshotId": "20260929t180701z-a1b2c3d4",
  "departmentId": 990001,
  "specialityId": 990011,
  "doctors": [
    {
      "id": 990012,
      "name": "Тестовый врач Невролог № 1",
      "bookingType": "speciality",
      "slotsHref": "appointment-availability/snapshots/20260929t180701z-a1b2c3d4/departments/990001/specialities/990011/doctors/990012/slots.json"
    }
  ]
}
```

A slots document contains the complete schedule for one doctor:

```json
{
  "schemaVersion": 2,
  "snapshotId": "20260929t180701z-a1b2c3d4",
  "departmentId": 990001,
  "specialityId": 990011,
  "doctorId": 990012,
  "slots": [{"date": "2030-01-15", "time": "10:30"}]
}
```

Every href is a repository-relative path below the same immutable
`appointment-availability/snapshots/{snapshotId}/` directory. Absolute paths,
URLs, traversal segments, encoded separators and links to another snapshot are
invalid. A publisher creates all snapshot files and changes `current.json` in a
single Git commit, then fast-forwards `refs/heads/main` once.

`catalog-v2.json` is publisher input, not an availability endpoint. It retains
the predefined departments, specialities and doctors even when no speciality is
visible in the current snapshot.
