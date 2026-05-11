---
name: bdd-scenario-design
description: Use when a new feature request, enhancement request, user story, acceptance criteria request, behavior specification, regression behavior, or test-first planning task is detected. Converts feature ideas and bug regressions into BDD-style Gherkin scenarios, acceptance criteria, edge cases, and testable behavior.
---

# BDD Scenario Design Skill

Use this skill when the task involves:
- planning a new feature
- enhancing existing functionality
- converting a user story into acceptance criteria
- defining expected behavior before implementation
- writing Gherkin scenarios
- preparing test-first development guidance
- defining regression behavior for a bug fix
- clarifying edge cases and validation behavior

Do not use this skill for:
- pure cleanup or refactoring with no behavior change
- CI infrastructure-only changes
- dependency-only upgrades
- documentation-only changes unless documenting behavior
- formatting or style-only changes

## Purpose

Convert feature requests or regression descriptions into clear, testable behavior using Behavior-Driven Development practices.

Focus on:
- actor
- goal
- business value
- user-visible behavior
- happy path
- validation failures
- empty-result cases
- not-found cases
- persistence behavior
- UI behavior
- regression scenarios

BDD scenarios must describe observable behavior, not implementation details.

## Spring PetClinic Domain Language

Prefer Spring PetClinic domain terms:
- owner
- pet
- veterinarian
- visit
- specialty
- clinic staff
- receptionist
- owner details page
- pet details
- visit description
- search result
- validation message

Avoid technical terms inside Gherkin steps:
- controller method names
- repository method names
- database table names
- Java class names
- model attribute names
- internal service names
- HTTP status codes, unless the feature is explicitly API-level

## Good Scenario Example

```gherkin
Scenario: Add a valid visit for an existing pet
  Given an owner named "George Franklin" has a pet named "Leo"
  When clinic staff records a visit for "Leo" with description "Annual vaccination"
  Then the visit should be saved for "Leo"
  And the owner details page should show "Annual vaccination"
```

## Bad Scenario Example

```gherkin
Scenario: Controller saves visit
  Given VisitController is initialized
  When processNewVisitForm is called
  Then visitRepository.save is invoked
```

## BDD Workflow

When a feature or regression request is detected:

1. Identify the actor.
2. Identify the user goal.
3. Identify the business value.
4. Define in-scope and out-of-scope behavior.
5. Search the repository for similar existing behavior.
6. Write Gherkin scenarios.
7. Derive acceptance criteria from the scenarios.
8. Map each scenario to a test type.
9. Recommend an implementation sequence.
10. Identify risks, assumptions, and open questions.

## Gherkin Rules

Use this structure:

```gherkin
Feature: <feature name>
  As a <actor>
  I want <capability>
  So that <business value>

  Scenario: <happy path>
    Given ...
    When ...
    Then ...

  Scenario: <validation failure>
    Given ...
    When ...
    Then ...

  Scenario: <empty or not-found case>
    Given ...
    When ...
    Then ...
```

Use `Scenario Outline` when the same behavior should be validated with multiple input examples.

Example:

```gherkin
Scenario Outline: Reject invalid owner phone numbers
  Given clinic staff is creating an owner
  When the phone number is "<phone>"
  Then the system should show a validation message for the phone number

  Examples:
    | phone     |
    | abc       |
    | 12        |
    | !@#$%     |
```

Use `Background` only when it reduces repeated setup across multiple scenarios.

Example:

```gherkin
Background:
  Given an owner named "George Franklin" exists
  And George has a pet named "Leo"
```

## Scenario Quality Rules

Each scenario must:
- describe behavior from the actor's point of view
- be independently understandable
- avoid technical method, class, or database names
- use concrete examples where useful
- have a clear observable outcome
- be testable
- avoid vague outcomes such as "the system works correctly"

Prefer:

```gherkin
Then the owner details page should show "Annual vaccination"
```

Avoid:

```gherkin
Then the visit should be processed correctly
```

## Acceptance Criteria Rules

Every acceptance criterion must map to at least one scenario.

Avoid vague criteria:
- works correctly
- is user friendly
- handles errors properly
- supports the feature
- validates input

Prefer concrete criteria:
- The system shows a validation message when the visit description is empty.
- The owner details page shows the newly added visit after save.
- The system does not create a visit when the pet ID does not exist.
- The search results page shows a clear empty state when no owners match the search term.

Use this format:

| ID | Acceptance criterion | Covered by scenario |
|---|---|---|
| AC-1 |  |  |

## Test Mapping

Map scenarios to the smallest suitable test type.

| Scenario type | Recommended test |
|---|---|
| Controller form behavior | MockMvc / Spring MVC test |
| Domain validation | Unit or validation test |
| Repository query behavior | Repository / JPA test |
| End-to-end Spring behavior | Spring Boot integration test |
| Template rendering | MVC test with rendered view assertions where practical |
| Bug regression | Smallest regression test that fails before the fix and passes after it |

Do not require Cucumber unless the repository already uses it or the user explicitly asks for Cucumber.

BDD scenarios can guide:
- JUnit tests
- Spring Boot tests
- MockMvc tests
- repository tests
- integration tests
- manual acceptance checks

## Regression Scenario Rules

When the task is a bug fix, write regression behavior as a scenario.

Use this structure:

```gherkin
Scenario: <bug no longer occurs>
  Given <precondition that exposes the bug>
  When <user action or system event>
  Then <expected behavior after the fix>
```

A regression scenario should clearly separate:
- the expected behavior
- the broken behavior
- the user-visible impact
- the test that should fail before the fix and pass after the fix

Example:

```gherkin
Scenario: Do not create a visit when the description is empty
  Given an owner named "George Franklin" has a pet named "Leo"
  When clinic staff submits a new visit for "Leo" with an empty description
  Then the visit should not be saved
  And the form should show a validation message for the description
```

## Implementation Guidance Rules

When producing implementation guidance:
- recommend a test-first or outside-in sequence
- start with the simplest scenario
- keep each implementation slice small
- avoid unrelated cleanup
- avoid changing behavior not covered by the scenarios
- call out open questions instead of silently inventing behavior

Recommended validation command for Spring PetClinic:

```bash
./mvnw test
```

Use broader validation only when justified:

```bash
./mvnw verify
```

## Output Format

Use this structure:

# BDD Scenario Pack: <feature or regression name>

## 1. Feature / Regression Intent

- Actor:
- Goal:
- Business value:

## 2. Scope

### In scope

-

### Out of scope

-

## 3. Domain Language

| Term | Meaning | Notes |
|---|---|---|
|  |  |  |

## 4. Gherkin Scenarios

```gherkin
Feature: <feature or regression name>
  As a <actor>
  I want <capability>
  So that <business value>

  Scenario: <happy path or regression path>
    Given ...
    When ...
    Then ...

  Scenario: <validation or edge case>
    Given ...
    When ...
    Then ...

  Scenario: <empty or not-found case>
    Given ...
    When ...
    Then ...
```

## 5. Acceptance Criteria

| ID | Acceptance criterion | Covered by scenario |
|---|---|---|
| AC-1 |  |  |

## 6. Test Strategy

| Scenario | Test type | Candidate test area | Notes |
|---|---|---|---|
|  |  |  |  |

## 7. Implementation Guidance

- Recommended first test:
- Recommended first production change:
- Suggested implementation sequence:
  1. Add or update the test for the first scenario.
  2. Implement the smallest production change needed.
  3. Run the narrowest relevant test.
  4. Repeat for edge cases.
  5. Run full validation.

Recommended validation command:

```bash
./mvnw test
```

## 8. Open Questions

| Question | Why it matters | Suggested default |
|---|---|---|
|  |  |  |
