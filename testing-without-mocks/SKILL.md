---
name: testing-without-mocks
description: Run a code quality and test architecture review using James Shore's "Testing Without Mocks" (Nullables) pattern language. Focus on eliminating mocks, enforcing logic sandwiches/A-Frame architecture, and utilizing state-based sociable tests.
disable-model-invocation: true
---

# Testing Without Mocks: Nullables & A-Frame Architecture Review

Use this skill to perform a code review specifically targeting test quality, test architecture, and the use of mocking frameworks. The goal is to move the codebase toward James Shore's "Testing Without Mocks" pattern language, heavily utilizing Nullables, A-Frame Architecture, Logic Sandwiches, and State-Based Sociable Tests.

This review should ruthlessly hunt down brittle, interaction-based tests (mocks, spies, stubs) and suggest architectural restructurings that make the code natively testable.

## Core Prompt

Start from this baseline:

> Perform a deep review of the test architecture and the testability of the production code.
> Look for any use of mocking frameworks, spies, or interaction-based assertions (`assert_called_with`).
> Challenge the use of mocks. Push for the code to be refactored using "Logic Sandwiches" (separation of I/O and pure logic) and "Nullables" (infrastructure wrappers with a built-in off switch).
> Actively search for brittle tests and push for "State-Based Sociable Tests" that verify real objects wired together instead of in isolated, mocked environments.

## Non-Negotiable Review Standards

Apply the baseline prompt above, plus these explicit review rules:

0. **No Mocks, Spies, or Stubs**
   - Flag any usage of mocking libraries (`unittest.mock`, `jest.mock`, `Mockito`, `Sinon`, `patch`, `MagicMock`, etc.).
   - Reject interaction-based assertions (e.g., `expect(mock).toHaveBeenCalled()`, `mock.assert_called_once_with()`).
   - Suggest replacing the mock with a "Nullable": an Infrastructure Wrapper that has a `createNull()` method (an embedded stub).

1. **Push for Logic Sandwiches (A-Frame Architecture)**
   - Look for business logic deeply intertwined with I/O (e.g., making an HTTP call, processing the result, then writing to a database all in one function).
   - Suggest refactoring into a **Logic Sandwich**:
     1. Read data (Infrastructure/IO)
     2. Process data (Pure Logic / No IO)
     3. Write data (Infrastructure/IO)

2. **State-Based Sociable Tests**
   - Tests should instantiate the real objects (or their Nullable variants for infrastructure) and test the unit along with its real dependencies.
   - Tests must assert on the **state** or **output data**, not on which methods were called on dependencies.

3. **Output Tracking over Behavioral Assertion**
   - If the test needs to verify that an external system *would* have been called (e.g., a message was published), suggest an "Output Tracker".
   - The Nullable infrastructure object should record its inputs in memory, and the test should assert against that recorded state.

4. **Parameterless Instantiation (Signature Shielding)**
   - Flag tests that do heavy, manual dependency wiring (e.g., passing 5 mocks into a constructor).
   - Suggest adding `createTestInstance()` or `create()` parameterless factories with sensible overridable defaults so sociable tests are trivial to set up.

## Primary Review Questions

For every meaningful change, ask:

- Does this test rely on a mocking framework to isolate the unit?
- Does the test verify *how* a dependency is used (`assert_called_with`) instead of verifying the *state* or the *result*?
- Is infrastructure/I/O mixed directly into pure business logic?
- Can this code be structured as an A-Frame Architecture where Application coordinates, Infrastructure does I/O, and Logic computes cleanly?
- If an external system is used, is it hidden behind an Infrastructure Wrapper that offers a `createNull()` method?
- Does setting up the test require too much brittle setup? Could a `createTestInstance()` factory simplify it?

## What to Flag Aggressively

Escalate findings when you see:

- Any `patch`, `jest.mock()`, or dependency injection used purely to supply dummy mock objects.
- Business logic that performs I/O directly without going through an Infrastructure Wrapper.
- Tests that are exact mirror images of the implementation (brittle interaction tests).
- Broad integration tests being used where a Narrow, Nullable-based sociable test would be faster and more deterministic.
- Setup blocks (`setUp`, `beforeEach`) filled with mock definitions and return-value configurations.

## Preferred Remedies

When you identify a mock-heavy or intertwined implementation, prefer suggestions like:

- "Instead of using `@patch` to mock the HTTP client, can we create an Infrastructure Wrapper for it and implement a `createNull()` method that provides configurable offline responses?"
- "This test is asserting that `db.save()` was called. Let's use Output Tracking on a Nullable `DataStore` to assert that the correct data was queued for saving instead."
- "This function mixes reading from the DB, applying business rules, and sending an email. Can we refactor this into a Logic Sandwich? Let the Application layer read the data, pass it to pure Logic, and then pass the result to the email Infrastructure."
- "Setting up this test requires mocking 4 different dependencies. Let's add a `createTestInstance()` factory to provide zero-impact, sociable defaults."
- "Instead of verifying behavior (`expect(mock).toHaveBeenCalled()`), let's use a state-based test that checks the output of the computation."

## Review Tone

Be encouraging but firm about structural testability. Emphasize that mocks make refactoring painful and lock in implementation details. Focus on how Nullables and Logic Sandwiches make code both natively testable and structurally cleaner.

Good phrases:

- `this test is highly coupled to the implementation details because of the mock. can we use a Nullable infrastructure wrapper here instead?`
- `we're mixing I/O with pure logic here, which is why we needed to mock this. can we extract the I/O to form a logic sandwich?`
- `this assert_called_with makes the test brittle. let's use output tracking on a Nullable to assert on the final state instead.`
- `this test setup is very noisy with mock return values. can we push this into a createNull() configurable response?`
- `let's avoid testing this in total isolation. can we use an overlapping sociable test where this unit talks to its real (but Nulled) dependencies?`

## Output Expectations

Prioritize findings in this order:

1. Presence of mocking frameworks (patches, spies, mocks).
2. Intertwined Logic and Infrastructure (missing Logic Sandwiches).
3. Interaction-based assertions vs State-based assertions.
4. Missing Infrastructure Wrappers.
5. Brittle test setup (missing parameterless test instantiation).

Focus on guiding the developer to restructure the code so it doesn't *need* to be mocked, rather than just suggesting a different way to write the test.