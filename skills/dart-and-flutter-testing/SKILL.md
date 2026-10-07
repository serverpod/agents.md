---
name: dart-and-flutter-testing
description: Use when writing or refactoring tests of any kind for Dart or Flutter projects.
---

## Goals

Tests should be:
- readable
- resilient to refactoring
- focused on behavior (not implementation)
- easy to maintain
- effective at catching bugs

Always ensure that edge cases and regressions introduced by the code change are covered by tests.

## Choose the test structure

**Readability governs grouping: split substantial phases and keep short, straightforward phases together.**

- For substantial arrangement and action, use Given → when → read-only then. Given setup establishes the scenario; when setup performs the action and captures results. Several outcomes can share that action through separate then tests.
- Partial grouping is equally valid: Given → short when/then test, or Given/when in shared setup → read-only then tests when the action is simple. Keep all three in one test when the whole scenario is straightforward.
- Judge each phase's readability, not line count or number of tests; even one then can justify separate setup. Hiding phases in a helper does not simplify them.

## Test descriptions

Descriptions must be understandable without unrelated code. Given, when, and then are clauses of the combined description.

- Given describes the precondition or state; when describes the action; then describes the observable outcome.
- Describe behavior from the business perspective and scenario state only in Given.
- Ensure the combined `group` and `test` descriptions form a complete sentence containing exactly one Given, one when, and one then.
- Use literal, searchable scenario names rather than conditional or loop-generated descriptions.
- When a string contains multiple clauses:
  - Place one clause per line, separated by commas.
  - Leave a trailing space after each comma when another string continues the sentence.
  - End the final clause with a period.
  - Never split a string in the middle of a clause solely to satisfy line length.

## Setup scope and lifetime

Distinguish scenario-independent infrastructure from scenario state.

- Put shared infrastructure only in `setUp` or `setUpAll` directly inside `main`. A top-level `withServerpod` is the sole structural-group exception because its API requires a group.
- Harnesses, databases, and containers count as infrastructure only if they do not define the scenario. Keep scenario entities, inputs, actions, and expected values out of global setup.
- Place each phase where its clause is declared, including shared Given/when setup or a combined when/then test. Keep arrangement, action, and assertions easy to distinguish.
- Group setup must implement its clause and apply to every descendant test. Given facts must be locally verifiable.
- Helpers may hide mechanics, but calls must expose scenario-relevant values and preserve the arrangement/action boundary.

> If changing a value would change the Given, when, or then description, treat it as scenario state rather than shared infrastructure.

- Prefer scenario `setUpAll` to share one completed action across read-only then tests; otherwise use `setUp` with fresh state.
- Match setup/teardown lifetimes: shared setup cannot depend on per-test initialization or survive per-test cleanup of its resources. Isolate sibling when groups whose actions affect one another.
- Then-only tests inspect stable results, preferably snapshots, without writes, interactions, sync, or event consumption. Normally they are synchronous assertions. Capture expected failures in when; keep behavioral assertions in then.
- `testWidgets` work requiring its tester stays in that callback, with explicit phases and fresh state. Do not force shared widget setup to satisfy the group shape.

## Group structure

- Use groups that own matching setup or meaningfully organize multiple descendant tests.
- Keep single-test groups that declare setup implementing their description.
- Recursively flatten a group only when it has one descendant test, owns no setup, and merely splits the description.
- A setup-free group may organize multiple tests sharing a setup-free initial state.
- Separate group, test, setup, and teardown declarations with a blank line.

Examples use illustrative application APIs. A shared scenario with read-only outcomes:

```dart
group('Given a blocked user,', () {
  late User user;

  setUpAll(() {
    user = User(blocked: true);
  });

  group('when the user attempts to authenticate,', () {
    late AuthenticationResult result;

    setUpAll(() async {
      result = await authenticate(user);
    });

    test('then authentication fails with reason "blocked".', () {
      expect(result.reason, 'blocked');
    });

    test('then no session is issued.', () {
      expect(result.session, isNull);
    });
  });
});
```

A Given group can also keep a short action and expectation together:

```dart
group('Given a blocked user,', () {
  late User user;

  setUp(() {
    user = User(blocked: true);
  });

  test(
    'when the user attempts to authenticate, '
    'then authentication fails with reason "blocked".',
    () async {
      final result = await authenticate(user);

      expect(result.reason, 'blocked');
    },
  );
});
```

## Explicit scenarios instead of loops

- Strongly discourage loops, parameter tables, and test-registration helpers. Declare distinct business scenarios explicitly with literal names and visible setup, especially when cases need conditional setup/assertions.
- One test's loop prevents selecting a case independently. Loops registering separate tests allow filtering, but require reconstructing names. Prefer explicit cases and mechanical helpers with concrete arguments, not mode flags hiding different scenarios.
- Allow loops within one scenario for fixture collections, collection assertions, or repetitive behavior such as retries. Property/fuzz tests need reproducible cases/seeds and do not replace explicit business regressions.

## Testing principles

- **Independence**:
  - No test may depend on another test's order or side effects.
  - Running one test alone must produce the same outcome as the full suite.
- **Single responsibility**:
  - One test → one behavior or outcome.
  - Split unrelated outcomes into separate then tests; share setup/action safely through groups or repeat with fresh state.
  - Multiple `expect`s are fine for one outcome, such as related fields of one object.
- **Implementation-agnostic (black box)**:
  - Prefer observable behavior; cover business logic, failures, and edge cases.
  - Avoid mocking and side effects in unit tests where possible.
  - Avoid `@visibleForTesting`, private helpers, internal methods, and call-count assertions unless part of the public contract.
- **Preserve the behavioral contract**:
  - Never change expectations to make a production bug pass; change them only when the contract changes or the test is wrong.
  - Structural refactors preserve scenario inputs, production paths, and every distinct behavioral assertion.
- **Simplicity over abstraction**:
  - Keep setup and assertions explicit in the test or matching described groups.
  - Avoid helpers that hide the clauses' meaning. Readability matters as much as correctness.
  - Compare complete strings where possible; use `contains` only for mutable or irrelevant parts.

## Test audit

Before finishing any test change:

1. Inspect longest tests first: split substantial phases; allow partial or full combinations when clearer.
2. Check exactly one Given/when/then per combined description, with literal names.
3. Check visible Given facts, clause-matching group setup, infrastructure-only global setup, and the structural-group rule.
4. Check read-only then-only bodies, assertions outside setup, and appropriate widget lifecycles.
5. Check setup/teardown lifetimes, stable results, and sibling action isolation.
6. Expand looped/conditional business cases; check helper arguments expose scenario values.
7. Flatten description-only single-test groups; retain groups owning matching setup.
8. Preserve all assertions and run affected tests. For newly shared setup, also run a later then alone by its full name to verify independence.
