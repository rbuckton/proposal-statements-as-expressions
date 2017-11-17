<!--
Welcome to your new proposal repository. This document will serve as the introduction and 
 strawman for your proposal.

The repository is broken down into the following layout:

  /README.md        # intro/strawman (this file)
  /LICENSE          # ECMA compatible license (BSD-3 Clause)
  /src              # ecmarkup sources for the specification
  /docs             # ecmarkup output

To build the specification, run:

  npm run compile

To preview the specification, run:

  npm run start

It is recommended that you configure GitHub Pages in your GitHub repository to point to the
'/docs' directory after you push these changes to 'master'. That way the specification text
will be updated automatically when you publish.

-->

# Proposal to Explore ECMAScript Statements as Expressions

As a follow up to the `throw` expressions [proposal](https://github.com/tc39/proposal-throw-expressions), 
this strawman seeks to explore the feasability of defining expression-forms for other statements. 
To frame this discussion and establish an overall scope for this proposal, at this time we are only
considering expression forms for a limited set of statements. As such, all declarations that do not
already have an expression form are out of scope for the purposes of this proposal.

## Status

**Stage:** 0  
**Champion:** Ron Buckton (@rbuckton)

_For more information see the [TC39 proposal process](https://tc39.github.io/process-document/)._

## Authors

* Ron Buckton (@rbuckton)

# Leaf Statements

For the purposes of this proposal, a "Leaf Statement" is a _Statement_ that does not 
directly contain other statements.

## `throw` operator

`throw` statements already have a [proposal](https://github.com/tc39/proposal-throw-expressions) 
for an expression form, allowing the following motivating use cases:

- Parameter initializers
    ```js
    function save(filename = throw new TypeError("Argument required")) {
    }
    ```
- Arrow function bodies
    ```js
    lint(ast, { 
      with: () => throw new Error("avoid using 'with' statements.")
    });
    ```
- Conditional expressions
    ```js
    function getEncoder(encoding) {
      const encoder = encoding === "utf8" ? new UTF8Encoder() 
                    : encoding === "utf16le" ? new UTF16Encoder(false) 
                    : encoding === "utf16be" ? new UTF16Encoder(true) 
                    : throw new Error("Unsupported encoding");
    }
    ```
- Logical operations
    ```js
    class Product {
      get id() { return this._id; }
      set id(value) { this._id = value || throw new Error("Invalid value"); }
    }
    ```

## Grammar

```grammarkdown
UnaryExpression[Yield, Await]:
  `throw` UnaryExpression[?Yield, ?Await]
```

## `return` operator

Allowing a `return` in an expression position would allow for the early exit from a function while
in the midst of any expression:

```js
function checkOpts(opts) {
    let x = opts?.x ?? return false;
    /* ...do something with x... */
}
```

### Grammar

```grammarkdown
UnaryExpression[Yield, Await]:
  `return` UnaryExpression[?Yield, ?Await]
```

## `break` operator

Allowing a `break` in an expression position would allow for an immediate jump from within the 
current expression to the end of the target _LabelledStatement_, _SwitchStatement_, or 
_IterationStatement_:

```js
switch (cond) {
  case 0:
    const y = x.y ?? break;
    // ...do something with `y`...
    break;
}
```

### Grammar

```grammarkdown
UnaryExpression[Yield, Await]:
  BreakExpression

BreakExpression:
  `break`
  `break` [no |LineTerminator| here] LabelIdentifier
```

## `continue` operator

Allowing a `continue` in an expression position would allow for an immediate jump to the next
iteration within an _IterationStatement_:

```js
for (const x of array) {
    const y = x.y ?? continue; // continue if `x.y` is not present.
}
```

### Grammar

```grammarkdown
UnaryExpression[Yield, Await]:
  ContinueExpression

ContinueExpression:
  `continue`
  `continue` [no |LineTerminator| here] LabelIdentifier
```

## `debugger` expression

Allowing a `debugger` operation in an expression position would allow for an immediate break
into an attached debugger at the current expression:

```js
const y = x.y ?? debugger; // break into debugger if `x.y` is not present.
```

### Grammar

```grammarkdown
UnaryExpression[Yield, Await]:
  `debugger`
```

## _ExpressionStatement_ restrictions

To avoid grammatical ambiguity, the above expressions are restricted from _ExpressionStatement_:

```grammarkdown
ExpressionStatement[Yield, Await]:
  [lookahead ∉ {`{`, `function`, `async` [no |LineTerminator| here] `function`, `class`, `let [`, 
    `throw`, `return`, `break`, `continue`, `throw`, `debugger` }] Expression[+In, ?Yield, ?Await]
```

# Branching Statements

Branching statements are those statements that directly contain one or more other statement. This 
category includes all control-flow statements, iteration statements, `with` statements, and blocks. 

This category is not yet fully explored and requires further investigation to determine what 
syntax and semantics would best fit each of these kinds of statements if they were to have an 
expression form.

This is an area that closely overlaps with the `do` expression proposal. A more thorough discussion 
regarding what constitutes a usable completion value from these statements would best be handled as
part of that proposal.

# Declaration Statements

Declaration statements are currently out-of-scope for this proposal.

<!-- The following are shared links used throughout the README: -->

[Champion]: #status
[Prose]: #motivations
[Examples]: #examples
[API]: #api
[Specification]: https://rbuckton.github.io/proposal-statements-as-expressions
[Transpiler]: #todo
[Stage3ReviewerSignOff]: #todo
[Stage3EditorSignOff]: #todo
[Test262PullRequest]: #todo
[Implementation1]: #todo
[Implementation2]: #todo
[Ecma262PullRequest]: #todo