---
name: dart-test-coverage-review
description: Audit Dart or Flutter test coverage through a nested description artifact, then validate apparent gaps against production paths before proposing or adding tests. Use for coverage reviews, not ordinary test writing.
---

# Dart Test Coverage Review

Also apply repository instructions and the Dart/Flutter testing skill.

## Workflow

1. Create `/tmp/<scope>_test_cases.md` from every `*_test.dart` file in scope.
   The artifact contains only test descriptions as nested bullets:
   - preserve source order and exact description text;
   - indent each group or group-like wrapper by two spaces;
   - include test registrations as leaves;
   - omit filenames, headings, counts, setup, and commentary.

   Prefer syntax-aware extraction. Account for repository-specific wrappers and
   dynamic test registrations. Verify that every leaf test appears exactly
   once and that the artifact contains no non-bullet content.

2. Review coverage using only the artifact. Do not inspect test bodies,
   production code, schemas, or documentation yet. Propose apparent missing
   behaviors as concise Given/when/then test descriptions.

3. Validate each proposal against production. Trace it from a public entry
   point through valid inputs and invariants to an observable result, then
   classify it as:
   - a real uncovered production path;
   - already covered;
   - unreachable under production invariants;
   - an implementation detail; or
   - a contract mismatch or dead guard.

   Retain only real uncovered paths as missing tests. Do not manufacture
   impossible database state or bypass production contracts to reach a branch.
   If no real gaps remain, say so without inventing tests.

4. A coverage-review request alone does not authorize repository edits. When
   the user also asks to add the missing tests, implement only the validated
   gaps. Keep Given/when/then data visible in each test or its matching groups;
   do not hide scenario facts in external fixtures or abstractions. Fix an
   exposed production defect only when that work is within the request.

5. Run appropriate focused and complete tests plus static analysis. Regenerate
   the artifact from the final test source rather than editing it by hand, and
   repeat its structural and leaf-count validation.

Keep artifact-only hypotheses distinct from production-validated findings.
