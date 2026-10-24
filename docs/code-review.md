# Review of the diff

Before a pull request, the author reads its own diff as a reviewer would. The gate's tools read the code for known patterns, and the security review (`docs/security-review.md`) reads the paths across a trust boundary; this read checks that the change does what the plan says, in the plainest form, without making any check easier to pass. It is reading: it adds no test run.

## When

- **Every pull request**, after the security review and before the merge's test runs (ship step 3), so it reads the security fixes too and a fix it brings is checked by those runs once: the author's read, below. Its result is a line in the pull request text (`docs/shipping.md`).
- **A change that is hard to undo** (`HARNESS.md` section 3): a second reader as well (below).
- A change that touches a trust boundary gets the security review for its security; this read does not repeat it.

## Tests first

Read the tests before the code: they show what the change claims. A change in behaviour has a test that was seen failing before the code that makes it pass, and a bug fix has the test that reproduced the bug (`HARNESS.md` section 3). The tests check what a caller or a user sees, not which internal functions were called (`docs/testing.md`, writing tests). A behaviour of the change that no test would notice is a finding: name the missing test.

## Five points

1. **Correct.** It meets the plan's success criteria and the slice's acceptance criteria, on the edges (empty, zero, the limit, the error path) as well as the main path.
2. **Plain.** The simplest form that works: no abstraction before its third use, no option nobody asked for, names that say what a thing is. A condition bolted onto an unrelated flow moves into a function of its own.
3. **In place.** It follows the patterns already in the project, its dependencies point one way, and its logic sits in the part that owns it. A new pattern is explained in the pull request text.
4. **Light.** No query inside a loop, no unbounded list, loop or request, no list endpoint without pages, no heavy work on a hot path. A change made for speed shows a measurement before and after, on the same input.
5. **Clean.** Code the change leaves unused is removed in the same change. Dead code the change did not touch is listed in the report, not removed. No debug output and no code left commented out.

## The bar is not lowered

The diff is read for any check made easier to pass:

- a test skipped, deleted or weakened, or an assertion removed from a test that stayed (a test removed together with the behaviour it tested is not a loosening);
- a suppression marker without its `Reviewed (code check): <why>` comment, or a blanket disable for a file or a rule (`docs/code-check.md`);
- a threshold or a blocking severity lowered, a linter rule turned off or moved from error to warning beyond the linter policy, a project's stricter setting lowered to the harness's, a gate step left out, a suite removed from the filters file or its paths narrowed;
- an entry added to the list of findings accepted on adoption, or a dependency ignore entry without its reason or its expiry date;
- a stub, an empty error handler or a to-do note where the code should be.

Each one is put back, or the pull request names it with the reason and it waits for the owner's go. A reviewed false positive with its comment, or a dependency ignore entry with its reason and expiry date, is not a loosening; it is the review. A step added to the known-flaky list is one: it carries its date and its log (`docs/testing.md`), and like the others it is named and waits for the owner's go.

## Size

A pull request carries one change a reader can hold in one sitting, about 300 changed lines or fewer; lock files, generated files and a replaced copy of the harness do not count. A larger change is split into pull requests that each work on their own (`docs/planning.md`, slices), unless it is a deletion or a mechanical change such as a rename. A refactor the change does not need is left out. One it needs goes first, in a pull request of its own when it is large; it changes no behaviour, so the existing tests are its proof and it needs no new test.

## A second reader

For a change that is hard to undo, a second reader that did not write it tries to find what is wrong, as the security review's second reader does for a finding: another agent that did not write the change, or, where the tool has none, the author in a later pass without its own notes. Either way the reader works from three things alone: the diff, the plan's success criteria and the slice's acceptance criteria, never the author's conclusion, and is asked for faults, not approval.

Each finding is checked against the code and ends in one of three states: fixed, accepted as a trade-off with the reason in the pull request text, or dismissed with the reason. A finding that came from a missing piece of the plan fixes the plan first. At most three rounds; when the third still finds a real fault, the owner decides.
