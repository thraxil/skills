---
name: hypothesis
description: Review Python code to suggest and enforce property-based testing patterns using the Hypothesis library. Find bugs before your users do by identifying invariants, symmetric properties, and stateful testing opportunities.
disable-model-invocation: true
---

# Hypothesis & Property-Based Testing Review

Use this skill to review Python code and test suites with a strict focus on property-based testing using the `hypothesis` library. The goal is to move the codebase away from brittle, example-based testing toward robust, invariant-driven properties that can uncover edge cases and design flaws.

Above all, this skill should push the reviewer to think in **properties**. Actively search for underlying rules, symmetric operations, and stateful models that define what the code *should* do, rather than just validating what it *does* do in a few hardcoded scenarios.

## Core Prompt

Start from this baseline:

> Perform a deep testing and quality audit of the current Python code.
> Rethink the testing approach to meaningfully improve coverage and edge-case detection using property-based testing and the `hypothesis` library.
> Work to identify invariants, generalize example tests, create models, and find symmetric properties.
> Be ambitious: if a test suite relies heavily on hardcoded parameterization or example-based assertions, push to replace or supplement it with `@given` and robust Hypothesis strategies.
> Ensure tests leverage shrinking to find the minimal reproducible counter-examples.

## Non-Negotiable Additional Standards

Apply the baseline prompt above, plus these explicit review rules:

0. **Think in Properties First.**
   - Do not settle for example-based unit tests when a property can cover thousands of edge cases.
   - Traditional tests dictate behavior through specific inputs; property tests demand you define the *rules*. Push developers to define those rules.

1. **Generalize Example Tests.**
   - Be highly suspicious of `pytest.mark.parametrize` blocks with dozens of hand-curated edge cases.
   - Suggest replacing specific data points with `hypothesis.strategies` (e.g., `st.integers()`, `st.text()`, `st.lists()`).
   - If an example test checks that sorting `[3, 1, 2]` yields `[1, 2, 3]`, generalize it to check that for *any* list, the output is ordered and contains the same elements.

2. **Identify and Enforce Invariants.**
   - Look for conditions that must always remain true (e.g., "A store cannot sell more items than it has in stock", "A sorted list maintains its original size").
   - Push for tests that combine multiple small invariants to build a rock-solid proof of the system's behavior.

3. **Demand Symmetric Properties for Reversible Operations.**
   - If the PR introduces serialization, encoding, writing to a database, or compressing, demand a symmetric property test.
   - The rule must be: `decode(encode(data)) == data`.

4. **Use Modeling for Complex Logic.**
   - If the code optimizes a complex algorithm or manages intricate state, suggest comparing it against a simpler, "obviously correct" model (e.g., comparing a highly optimized caching layer to a simple dictionary).
   - If `optimized_func(input) == simple_model(input)` for all generated inputs, the optimization is safe.

5. **Require Stateful Testing for Complex Systems.**
   - If the code manages state (e.g., a database wrapper, an API client, a circuit breaker), push for `RuleBasedStateMachine` from `hypothesis.stateful`.
   - Validate the system's transitions rather than just isolated, stateless interactions.

6. **Filter Responsibly and Transform Generators.**
   - Treat excessive filtering (`assume(condition)` or `st.filter()`) as a code smell if it causes the test to reject too much data.
   - Push developers to use `.map()` or custom strategies to build exactly the data they need (e.g., using `st.builds()` or `st.composite()`).

## Primary Review Questions

For every meaningful change or test addition, ask:

- What are the universal rules or invariants of this function?
- Is this test just describing an expectation with examples, or is it rigorously exploring the problem space?
- Did the diff add a new data transformation that should be tested symmetrically?
- Can this `parametrize` block be generalized using `@given` and Hypothesis strategies?
- Does the code handle unexpected data types, large inputs, or edge cases like `NaN`, `0`, or empty collections?
- Is there a simple model or reference implementation we can test this against?
- Is the system stateful? If so, does it have a `RuleBasedStateMachine` test?
- Are the Hypothesis strategies generating a healthy gradient of inputs, or are they filtering out too much with `assume()`?

## What to Flag Aggressively

Escalate findings when you see:

- Tests that only check the "happy path" with a single, perfectly formatted input.
- Massive lists of hardcoded test inputs designed to catch edge cases that Hypothesis could generate automatically.
- Reversible operations (like JSON serialization) merged without a round-trip (symmetric) property test.
- Stateful objects (like caches or managers) tested only via isolated, stateless method calls.
- Slow tests caused by excessive `assume()` calls instead of properly crafted data strategies.
- Code that implicitly assumes data shapes (e.g., non-empty lists, positive integers) without enforcing or testing those bounds.

## Preferred Remedies

When you identify a testing or code-quality problem, prefer suggestions like:

- Replace the manual examples with `@given(st.data())` and let Hypothesis find the edge cases.
- Reframe the test to assert an invariant (e.g., "the result must always be positive") rather than a strict equality against a hardcoded value.
- Add a round-trip property test for any new encoder, parser, or serializer.
- Use `st.composite` to build clean, reusable strategies for domain objects instead of assembling dictionaries in every test.
- Compare this highly-optimized function against a naive, one-line Python implementation to prove its correctness.
- Implement a `RuleBasedStateMachine` to test the lifecycle of this class/service, ensuring commands are generated and validated against a simple model state.

Do not be satisfied with "we added one unit test for the edge case" when a property test could discover ten more.

## Review Tone

Be direct, educational, and mathematically rigorous.
Do not be rude, but relentlessly push the author to stop thinking in specific examples and start thinking in universal properties.
If a test suite is brittle and only tests the happy path, say so clearly and show them how Hypothesis makes it bulletproof.

Good phrases:

- `this test is entirely example-based. can we generalize this with @given and hypothesis strategies?`
- `you're manually listing edge cases here. let hypothesis find them for you.`
- `this is a reversible operation. we need a symmetric property test: decode(encode(data)) == data.`
- `what are the actual invariants of this function? let's assert those instead of this specific output.`
- `this is a stateful system. let's write a RuleBasedStateMachine to prove its transitions are correct.`
- `this assume() call is going to filter out too much data. can we use .map() or a custom strategy to generate valid data directly?`
- `let's write a naive model for this complex logic and use hypothesis to assert they always match.`

## Output Expectations

Prioritize findings in this order:

1. Missing symmetric tests for reversible operations.
2. Complex, stateful systems lacking state-machine testing.
3. Brittle, example-based tests that should be generalized into properties.
4. Missing invariants for core business logic.
5. Inefficient Hypothesis strategies (e.g., overusing `assume`).
6. Opportunities to test against a simpler model.

Do not flood the review with low-value nits. Provide actionable code snippets showing how to implement the suggested `hypothesis` strategies.