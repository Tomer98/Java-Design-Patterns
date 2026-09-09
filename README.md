# User Validation Library

A small Java library for validating users against tier-specific rules, built around a
composable validator rather than a pile of `if` statements. Individual rules are values
that can be combined with boolean operators, and each user tier declares its own policy by
composing them.

```java
User user = UserFactory.createUser("platinum", "tomerbarel", "tomer@campus.ac.il", "Pa$$w0rd12", 25);

ValidationResult result = user.validate();
if (!result.isValid()) {
    System.out.println(result.getReason().orElse("unknown"));
}
```

## The idea

`UserValidator` extends `Function<User, ValidationResult>`, so a rule is just a function.
That makes rules composable:

```java
UserValidator policy = UserValidator.ageBiggerThan18()
        .and(UserValidator.usernameLengthBiggerThan8())
        .and(UserValidator.passwordLengthBiggerThan8())
        .or(UserValidator.passwordIncludesDollarSign());
```

`and` and `or` short-circuit and propagate the *failing* result outward, so the caller gets
the specific reason a check failed rather than a bare `false`. `xor` requires exactly one
side to pass, and the static `all(...)` and `none(...)` aggregate over any number of rules.

## Design patterns

| Pattern | Where |
|---|---|
| **Template Method** | `User.validate()` is `final` and calls the abstract `buildValidator()` hook, so every tier gets identical validation *mechanics* and defines only its own *policy* |
| **Factory** | `UserFactory.createUser(type, ...)` maps a tier string to the concrete subclass and rejects unknown types with a descriptive exception |
| **Strategy** | Each rule is an interchangeable `UserValidator`, selected and combined at runtime |
| **Result object** | `Valid` / `Invalid` implement `ValidationResult`, carrying the failure reason as an `Optional<String>` instead of returning a boolean or throwing |

## Tiers

Each tier composes a progressively stricter policy:

| Tier | Rules |
|---|---|
| **Basic** | age over 18, username longer than 8 |
| **Premium** | basic rules plus password longer than 8 |
| **Platinum** | premium rules plus password differs from username, and email ends with `il` |

## Available rules

`emailEndsWithIl` · `emailLengthBiggerThan10` · `passwordLengthBiggerThan8` ·
`passwordIncludesLettersNumbersOnly` · `passwordIncludesDollarSign` ·
`passwordIsDifferentFromUsername` · `ageBiggerThan18` · `usernameLengthBiggerThan8`

Adding a rule means adding one static method that returns a `UserValidator` — no existing
class needs to change.

## Layout

```
src/il/ac/hit/validation/    library sources
test/il/ac/hit/validation/   demo exercising each tier
validation.jar               packaged build
code_review.md               annotated walkthrough of every class
```

## Notes

Written for a Java course at the Holon Institute of Technology. Validation is intentionally
rule-based and self-contained — no framework, no annotations, no reflection, and no
dependencies beyond the JDK.
