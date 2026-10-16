# Implementation Notes

## 1. What I changed


- **Task 1 — Bug Fixes:**

  - **Diff calculation (`diff.util.ts`):** Fixed `computeDiff` so it actually checks _both_ `unitPrice` and `quantity` changes.
  - **Permission gating (`cr-detail.component.ts`):** Updated `canApprove` and `canReject` to verify both the `PENDING_APPROVAL` status and whether the user's policies include action rights (`cr_a_...`), locking read-only viewers (`cr_r_o`) from actions.

- **Task 2 — List UI & Filtering (`cr-list.component.ts`):**

  - Completed the `visibleRows` getter, so it dynamically filter the list with the selcted status dropdown.

- **Task 3 & 4 — Detail UI, Timeline, Actions, & Validation (`cr-detail.component.ts` & Template):**

  - **Detail View Updates:** Added `ngOnChanges` to `CrDetailComponent` so that when a user clicks a different change request in the list, the detail panel actually re-fetches and displays the newly selected CR instead of staying stuck on the initial one.

  - **Timeline sorting:** Added a chronological sort (`oldest first`) to the audit entries so reviewers can trace the history naturally.
  - **API integration:** Wired up `approve()` and `reject()` to call the `CrApiService` with proper loading states (`submitting`), error handling (`actionError`), and automatic data refreshing.
  - **Rejection validation:** Added `Validators.required` to `rejectControl`, ensuring the UI blocks empty reason submissions.

- **Task 5 — Testing:**
  - Added targeted component/DOM tests covering the status filter, chronological timeline ordering, and rejection text validation.

## 2. Component & state model

- **Architecture:** The application uses Angular 15 standalone components and reactive forms
- **View states:** I learned from the provided `ViewState` pattern (`idle`, `loading`, `loaded`, `empty`, `error`) so the UI always tells the user what is going on (like showing a loading text or an error message with a retry button) instead of freezing or showing a blank pag
- **Data flow:** \* Data flows from the mock service into the component, gets saved in state, and is then displayed in the HTML template using simple component getters.

## 3. Invariants I keep

| Invariant                                                                           | How / where                                                                                                                 |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Read-only security:** Viewers shouldn't be able to approve or reject.             | I fixed the permission checks in the component so read-only accounts (`cr_r_o`) have their action buttons locked down.      |
| **Chronological timeline:** The history log has to show oldest events first.        | I added a date-sorting function to the component getter so the audit trail always orders things from oldest to newest.      |
| **No empty rejections:** You can't reject a change request without typing a reason. | I added a required validator to the reactive form so the reject button physically stays disabled until you write something. |

## 4. Testing strategy

- I focused on writing simple component and DOM tests using Jest and Angular's TestBed to make sure the key features work properly for the user.
- Instead of worrying about a high test coverage number, I tested the main behaviors: the list filter, timeline sorting, read-only permissions, and form validation.

## 5. Assumptions

- The instructions clearly explained the permission-string layout (`cr_{action}_{scope}`), like how `cr_a_o` means approving and `cr_r_o` means read-only. I used that exact logic to make sure the UI only lets users perform actions they are actually permitted to do.
- Generating timestamps When clicking approve or reject, the mock API needed a time string for the history log. The prompt didn't specify how to format it, so I assumed using `new Date().toISOString()` right when the button is clicked was the best way to do it.

## 6. Where I used AI

- I used AI to help me wire up the API calls for the `approve()` and `reject()` actions in Task 3, and help me write some tests.

## 7. What I'd improve with more time

- Add more unit tests for tricky edge cases like network errors during approval.
