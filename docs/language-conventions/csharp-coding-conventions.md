# C# Coding Conventions

Offline working copy for Codex agents.

- Source: https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Source title: Common C# code conventions
- Source last updated: 2025-01-18
- Offline copy created: 2026-05-20

This file summarizes the Microsoft Learn C# coding conventions so agents do not need to fetch the page for routine C# planning or implementation. Refresh this file from the source when the Microsoft page changes or when a task depends on the newest C# guidance.

## Use With Repo Rules

Prefer the current repository's `.editorconfig`, formatting tools, analyzer rules, and established local conventions when they are more specific. Use this file as the default C# convention baseline when the repository does not define a stronger rule.

## General Language Guidance

- Use modern C# features when they improve clarity and are compatible with the target framework and project language version.
- Avoid outdated language constructs when a clear modern alternative exists.
- Write straightforward code that favors readability over cleverness.
- Catch only exceptions that can be handled meaningfully. Prefer specific exception types and useful error messages.
- Use LINQ where it improves collection manipulation readability.
- Use `async` and `await` for I/O-bound work. Avoid sync-over-async patterns.
- Be careful with deadlocks; use `ConfigureAwait` where the application model and library context make it appropriate.
- Prefer C# language keywords for built-in types, such as `string`, `int`, and `bool`, instead of runtime type names such as `System.String`.
- Prefer `int` over unsigned numeric types except when the domain or API explicitly requires unsigned values.
- Use `var` only when the type is obvious from the right side of the assignment or when LINQ/query projections make explicit types noisy.

## Strings

- Use string interpolation for short string composition.
- Use `StringBuilder` for repeated string appends in loops or large text construction.
- Prefer raw string literals for multiline strings or text that would otherwise require many escapes.
- Prefer expression-based interpolation over positional formatting.

## Construction And Initialization

- Use PascalCase primary constructor parameters for records.
- Use camelCase primary constructor parameters for classes and structs.
- Use `required` properties when the object must be initialized by callers and that communicates intent better than constructor overloads.
- Use collection expressions when initializing collections if the project language version supports them.
- Use object initializers when they make construction clearer.
- Use concise object creation syntax when the variable type already makes the constructed type clear.

## Delegates And Events

- Prefer `Func<>` and `Action<>` instead of declaring custom delegate types, unless a named delegate adds meaningful domain clarity or matches an existing API.
- If a custom delegate is necessary, use concise method-group assignment syntax.
- For event handlers that do not need to be removed later, a lambda handler is acceptable and often clearer.

## Exceptions And Disposal

- Use `try`/`catch` for exception handling when the code can recover, enrich context, or perform meaningful translation.
- Do not catch broad exceptions unless there is a deliberate boundary, filter, logging, or recovery reason.
- Use `using` or `await using` for disposable resources.
- Prefer the modern using declaration form when it keeps the lifetime clear.

## Operators And Expressions

- Use short-circuit boolean operators `&&` and `||` for boolean comparisons.
- Use parentheses to make complex boolean clauses clear.
- Break long expressions across lines before binary operators when a line break is needed.

## Static Members

- Call static members through the declaring type name so the static access is explicit.
- Do not qualify a base class static member through a derived type.

## LINQ

- Use meaningful query variable names that describe the result.
- Use aliases in anonymous projections when names would otherwise be ambiguous or incorrectly cased.
- Put filtering clauses early so later clauses operate on a smaller data set.
- Align query clauses consistently.
- Use multiple `from` clauses for inner collections when that reads better than a join.
- Use implicit typing for query variables and range variables because LINQ projections often produce anonymous or complex generic types.

## Implicit Local Variables

- Use `var` when the assigned expression makes the type clear, such as literals, `new` expressions, explicit casts, or obvious factory results.
- Use explicit types when the right side does not make the type apparent.
- Do not encode the type into the variable name. Name variables for meaning, not implementation type.
- Use `dynamic` intentionally when runtime binding is required; do not use `var` to hide dynamic behavior.
- Use `var` for `for` loop counters when clear.
- Prefer explicit element types in `foreach` loops when the collection element type is not immediately obvious.

## Namespaces And Usings

- Prefer file-scoped namespace declarations for files that declare one namespace.
- Put `using` directives outside the namespace declaration.
- Keep namespace and dependency direction consistent with the repository architecture.

## Formatting

- Use four spaces for indentation.
- Do not use tab characters for indentation.
- Use one statement per line.
- Use one declaration per line.
- Add blank lines between method and property definitions where they improve readability.
- Use Allman braces: opening and closing braces on their own lines aligned with the current indentation level.
- Break long statements into multiple lines instead of forcing dense code.
- Follow repository formatter output if it differs in a deliberate, enforced way.

## Comments

- Use `//` comments for brief explanations.
- Avoid block comments for long explanations.
- Put comments on their own line, not at the end of a code line.
- Start comment text with an uppercase letter and end complete-sentence comments with a period.
- Use XML documentation comments for public APIs when the repository convention expects public member documentation.
- Prefer self-explanatory names and structure over comments that restate the code.

## Security

- Follow secure coding guidance for input validation, authentication, authorization, secret handling, cryptography, logging, and dependency use.
- Do not log secrets or sensitive personal data.
- Treat external input as untrusted at system boundaries.

## Agent Checklist

When planning or editing C# code:

1. Check repo-specific `.editorconfig`, analyzer settings, and existing nearby style first.
2. Apply this convention baseline for any style question not answered locally.
3. Keep SOLID guidance proportional to the change; avoid creating abstractions only to satisfy a pattern name.
4. Prefer idiomatic C# and framework conventions over generic cross-language style rules.
5. Record any intentional deviation in the architecture plan or implementation summary.
