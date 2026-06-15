---
name: parse-dont-validate
description: Run a code quality review focused on type-driven design. Look for opportunities to "parse, don't validate", use domain-specific types, and make invalid states unrepresentable. Best used in languages with type systems (Python, TypeScript, C#, Java) to encourage functional programming lessons.
---

# Parse, Don't Validate: Type-Driven Design Review

Use this skill to perform a code review specifically targeting how the type system is utilized. The goal is to move the codebase toward type-driven design principles like "Parse, don't validate" (from Alexis King's article) and "Make Invalid States Unrepresentable" (from Domain Modeling Made Functional). 

This is particularly useful in codebases like Python, TypeScript, or Java where developers might not naturally leverage the type system to its full potential compared to developers in functional languages like Haskell, F#, or Rust.

## Core Prompt

Start from this baseline:

> Perform a code review focused on type-driven design and domain modeling.
> Look for opportunities to apply the "Parse, don't validate" philosophy: instead of checking if data is valid and continuing to use the raw primitive, functions should parse raw data into a strictly typed domain object.
> Actively search for ways to "Make Invalid States Unrepresentable" using discriminated unions, enums, or algebraic data types.
> Flag primitive obsession and scattered validation (shotgun parsing). Suggest refactoring toward robust domain types.

## Non-Negotiable Review Standards

Apply the baseline prompt above, plus these explicit review rules:

0. **Parse, Don't Validate**
   - If a function takes a `string`, validates it (e.g., checks if it's a valid email or UUID), and returns the same `string` or `bool`, flag it.
   - Suggest returning a specialized type (e.g., `EmailAddress`, `UserId`) so the type system preserves the proof of validation.
   - Look for "shotgun parsing" (validation checks scattered deeply into business logic). Push for parsing raw input at the system boundaries and passing only validated domain types inward.

1. **Make Invalid States Unrepresentable (MISU)**
   - Look for structs, classes, or dictionaries with mutually exclusive fields or boolean flags that can result in nonsensical combinations.
   - Example: An object with `status: "error"` and `data: [...]`. If there's an error, data shouldn't exist!
   - Push for discriminated unions, algebraic data types (ADTs), or mutually exclusive subclasses to guarantee at compile-time that illegal states cannot be instantiated.

2. **Cure Primitive Obsession**
   - Flag the overuse of strings, integers, or generic dictionaries to represent domain concepts.
   - Suggest wrapping them in semantic types (e.g., `NewType` in Python, tagged types in TypeScript, or Value Objects in Java/C#).
   - This prevents accidental swapping of arguments (e.g., passing a `CustomerId` into a `ProductId` parameter).

3. **Replace Booleans with Enums / State Machines**
   - Be highly suspicious of boolean flags (`is_active`, `is_admin`, `has_error`).
   - If multiple booleans are combined to determine a state, suggest an explicit `Enum` or State Machine.

4. **Address Optional/Null Abuse**
   - Flag properties that are `Optional` or nullable just because they aren't available in *some* phases of an object's lifecycle.
   - Suggest splitting the type into multiple phases (e.g., `UnregisteredUser` vs. `RegisteredUser`) so that fields are guaranteed to be present when required.

## Primary Review Questions

For every meaningful change, ask:

- Does this code validate data and throw away the proof, or does it parse data into a type that proves its validity?
- Are raw primitives (`string`, `int`, `dict`, `any`) used where a specific Domain Type would add safety?
- Is validation scattered throughout the function, or pushed to the system boundary?
- Can a developer accidentally instantiate an invalid combination of fields in this data structure?
- Are multiple booleans used to track state that would be better represented as an Enum or Discriminated Union?
- Is `Optional`, `None`, or `null` being used to paper over the fact that we have distinct states?

## What to Flag Aggressively

Escalate findings when you see:

- Validation functions returning `void`, `bool`, or the original unrefined type.
- "Shotgun parsing": spreading checks for invalid data throughout the business logic.
- Primitive obsession: passing generic types like `dict` or `str` deep into the core domain.
- State representations where invalid combinations are syntactically valid (e.g., `success: bool`, `error_message: Optional[str]`).
- "God objects" that have all fields marked as optional to represent different stages of a process.

## Preferred Remedies

When you identify a type-design problem, prefer suggestions like:

- "Instead of `validate_email(email: str) -> void`, how about `parse_email(email: str) -> EmailAddress`?"
- "We have `status` and `error_reason`. Let's use a discriminated union / variant type so `error_reason` only exists when `status == 'ERROR'`."
- "Let's create a `UserId` type instead of using a raw `int` to prevent accidentally mixing it up with `OrderId`."
- "Move all these validation checks to the boundary layer (e.g., the API route or CLI entrypoint). Once inside the core business logic, only accept fully parsed Domain Objects."
- "Instead of a single `Order` class with optional `shipped_at` fields, let's create `PendingOrder` and `ShippedOrder` types."

## Review Tone

Be direct, educational, and constructive. Keep in mind that developers coming from dynamic languages might not be familiar with these functional programming idioms. Explain *why* the type-driven approach prevents bugs (e.g., "by using a specific type, the type-checker guarantees this check was performed").

Good phrases:

- `this validates the string but throws away the proof. can we return a specific type (like EmailAddress) instead?`
- `these two fields can create an invalid state if both are populated. can we use a discriminated union here?`
- `we're passing a raw string here. can we use a NewType / value object to prevent accidental mix-ups with other string IDs?`
- `this validation is happening deep in the business logic. can we parse this at the boundary instead?`
- `this class has many optional fields depending on its lifecycle. can we split this into separate types representing each explicit state?`

## Output Expectations

Prioritize findings in this order:

1. Structural invalid states that can be represented (MISU).
2. Missing opportunities to parse into domain types (Parse, don't validate).
3. Primitive obsession and lack of semantic typing.
4. Shotgun parsing vs boundary parsing.
5. Enum vs Boolean flag cleanups.

Focus on structural improvements to the types rather than low-value stylistic nits.
