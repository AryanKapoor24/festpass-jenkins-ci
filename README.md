# FestPass Jenkins Challenge: Submission

**Student (Git author alias):** ARYAN KAPOOR
**Class code:** 001 · **Challenge code:** 00138

## Where everything is

| Required item | Link |
|---|---|
| 📄 Evidence page | [evidence/EVIDENCE.md](evidence/EVIDENCE.md) |
| 🔴 Red evidence CSV: build #2 | [evidence/red-build-2/submission-evidence.csv](evidence/red-build-2/submission-evidence.csv) |
| 🟢 Green evidence CSV: build #3 | [evidence/green-build-3/submission-evidence.csv](evidence/green-build-3/submission-evidence.csv) |
| 📦 Final JAR (build #3) | [evidence/green-build-3/festpass.jar](evidence/green-build-3/festpass.jar) |
| ⏱️ Polling trigger | [Jenkinsfile](Jenkinsfile) |
| 🔧 The fix | [src/main/java/BookingService.java](src/main/java/BookingService.java) |

## Summary

| Build | Cause | Commit | Result |
|---|---|---|---|
| #1 | Build Now (bootstrap) | `438f394` Configure my Jenkins assignment | ❌ 3/28 failed |
| #2 | Started by an SCM change | `46f32d6` Personalize my help introduction | ❌ 3/28 failed, no JAR |
| #3 | Started by an SCM change | `a0ba479` Fix cancel so it removes the booking and frees the seat | ✅ 28/28 passed, JAR published |

**Bug:** `BookingService.cancel()` checked that the booking existed but never removed the student,
so a cancelled seat stayed taken and the next booking was rejected as SOLD OUT.
**Fix:** added `event.students.remove(studentId);`. No tests or pipeline stages were changed.

Full details are in the [evidence page](evidence/EVIDENCE.md).
