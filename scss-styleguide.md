# SCSS Style Guide

## Table of Contents

1. [Basic Rules](#basic-rules)
2. [Spacing](#spacing)
3. [Formatting](#formatting)
4. [Variable Naming](#variable-naming)

## Basic Rules

- Be consistant with indentation
- Be consistant with spacing
- One selector per line
- One rule per line
- Use `!important` only when you can't overwrite a rule the other way
- Use flex boxes
- Use camelCase for rule naming
- Always care about mobile display
- Put `@media-` queries inside the element and not at the end of the `.scss` file

  > Why? Cause it's much easier to maintain

  ```scss
  // bad
  .theHeader {
    background: #FFF;
    height: 80px;
  }
  ...a lot of other rules
  @media all and (max-width: $break-point-sm) {
    .theHeader {
      height: 50px;
    }
  }

  // good
  .theHeader {
    background: #FFF;
    height: 80px;
    @media all and (max-width: $break-point-sm) {
      height: 50px;
    }
  }
  ```

## Spacing

- Use soft-tabs with a two space indent. Spaces are the only way to guarantee code renders the same in any person's environment.
- Put spaces after `:` in property declarations.
- Put spaces before `{` in rule declarations.
- Put line breaks between rulesets.
- When grouping selectors, keep individual selectors to a single line.
- Place closing braces of declaration blocks on a new line.
- Each declaration should appear on its own line for more accurate error reporting.

## Formatting

- Use hex color codes `#000` unless using `rgba()` in raw CSS (the SCSS `rgba()` function is overloaded to accept hex colors, as in `rgba(#000, .5)`).
- Use `//` for comment blocks (instead of `/* */`).
- Avoid specifying units for zero values, e.g., `margin: 0;` instead of `margin: 0px;`.
- Strive to limit use of shorthand declarations to instances where you must explicitly set all the available values.

## Variable Naming

Name variables not by a place where they used or their current value but by their main purpose

```scss
// bad
$lightPink: #ce6799;

// good
$primaryColor: #ce6799;

```

---

Credits: this guide is based on [Primer SCSS Principles](https://primer.style/css/principles/scss) + our own changes according to principles we use in our company