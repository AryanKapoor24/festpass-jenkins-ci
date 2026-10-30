# FestPass Jenkins Challenge: Evidence

**Student (Git author alias):** ARYAN KAPOOR
**Class code:** 001  **Challenge code:** 00138
**Date:** 30 October 2026
**Jenkins:** 2.580.1 LTS on JDK 21.0.13 (Temurin)

## 1. Job configuration

| Setting | Value |
|---|---|
| Job name / type | `festpass-ci`, Pipeline |
| Definition | Pipeline script from SCM, Git |
| Repository URL | `git://127.0.0.1:9418/festpass-java` (no credentials) |
| Branch / script path | `*/main` / `Jenkinsfile` |
| Agent | `student-laptop` (permanent agent, 1 executor, WebSocket); built-in node has 0 executors |

## 2. Committed polling trigger

The only Jenkinsfile change, added inside `pipeline` beside `options` and `stages`:

```groovy
triggers {
    pollSCM('* * * * *')
}
```

It was committed together with `submission.properties` in `438f394`. No UI trigger or "Build periodically" was configured.

## 3. Build trail

| Build | Cause | Commit | Result | Tests | JAR |
|---|---|---|---|---|---|
| #1 | Started by user (Build Now, bootstrap only) | `438f394` Configure my Jenkins assignment | FAILURE | 28 run, 3 failed | NOT_PUBLISHED |
| #2 | **Started by an SCM change** | `46f32d6` Personalize my help introduction | FAILURE | 28 run, 3 failed | NOT_PUBLISHED |
| #3 | **Started by an SCM change** | `a0ba479` Fix cancel so it removes the booking and frees the seat | **SUCCESS** | **28 run, 0 failed** | Published |

In each case, the commit printed in the Jenkins console matched `git log -1 --oneline` in my repository.

**Red builds (#1, #2):** Checkout, Submission setup and compilation succeeded. The Test stage failed, and
**Build JAR, Smoke test JAR and Save artifact were skipped** ("skipped due to earlier failure(s)").
Only `submission-evidence.csv` was archived.

**Green build (#3):** all stages passed. 7 real-JAR smoke checks passed, and the archived files were
`festpass.jar`, `build-info.txt`, `smoke-test.txt` and `submission-evidence.csv`.
JAR SHA-256: `7a6ec7bb238aca9a3fc1c1fb9d576b922834063d4e8a2d801f5bc37deecc05c4`

## 4. Diagnosis

Failed tests in builds #1 and #2:

```
FAIL cancellationFreesSeat: SOLD OUT. Even the developer needs a ticket.
FAIL cancellationsSurviveReload: Cancellation was lost
FAIL cliWorkflowPersistsAcrossCommands: List does not reflect persisted cancellation
```

**Failing assertion explained (`cancellationFreesSeat`):** the "music" event has 1 seat. Student S1 books it
and then cancels. Student S2 should then be able to book that freed seat, and the test expects the roster
to be `[S2]`. Instead, S2 was rejected with "SOLD OUT".

**Violated business rule:** cancelling an existing booking must remove it and free its seat.
In `BookingService.cancel()`, the code checked that the booking existed but never removed the
student from the event, so the seat stayed occupied. That one cause explains all three failures:
the in-memory roster, the saved file and the CLI `list` output all still showed the cancelled booking.

## 5. Fix

A minimal one-line change in `src/main/java/BookingService.java`:

```diff
         if (!event.students.contains(studentId)) {
             throw new IllegalArgumentException("No booking for student: " + studentId);
         }
+        event.students.remove(studentId);
     }
```

No tests, assertions, build helpers, evidence code or pipeline stages were changed. Locally,
`java ci/Build.java verify` gave 28 tests and 0 failures, and `java ci/SmokeTest.java` passed.
I committed the fix without clicking Build Now, and polling started build #3, which was green.

## 6. Final JAR output

From the download folder of build #3:

```
> java -jar festpass.jar help
FestPass - campus event registration, configured by ARYAN KAPOOR
Usage: java -jar build/festpass.jar [--data FILE] COMMAND
  add-event ID "TITLE" CAPACITY
  list
  book EVENT_ID STUDENT_ID
  cancel EVENT_ID STUDENT_ID
  roster EVENT_ID
  summary
  help
...
```

## 7. Submitted files

- Repository root: source and Git history
- [`red-build-2/submission-evidence.csv`](red-build-2/submission-evidence.csv): red evidence (the automatic failed build)
- [`green-build-3/submission-evidence.csv`](green-build-3/submission-evidence.csv): green evidence (the automatic successful build)
- [`green-build-3/festpass.jar`](green-build-3/festpass.jar): the final JAR
- [`red-build-1-bootstrap/submission-evidence.csv`](red-build-1-bootstrap/submission-evidence.csv): the bootstrap run (extra)
