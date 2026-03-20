# Kinshiq Descendant Tree Format v0.3-dev

*Text format for indentation-based descendant trees*

Copyright (c) 2026 Stefan Sellman  
Licensed under the MIT License.

---

## Status of this document

This document defines the **Kinshiq Descendant Tree Format**.

The specification is currently under active development. It is intended to become the canonical shared specification for descendant-tree text used by Kinshiq tools and related applications.

The canonical version of this document is maintained in the Kinshiq GitHub repository:

https://github.com/kinshiq/spec

Feedback, discussion, and contributions are welcome.

---

## 1. Overview

This specification defines a line-oriented text format for representing a descendant tree rooted at one individual.

The format is designed to be:

- human-readable,
- easy to edit in plain text,
- deterministic to parse,
- suitable for validation, rendering, and interchange between Kinshiq tools.

Each non-empty line describes one person or partner entry. Indentation expresses descendant-tree depth.

This format models presentation and descendant structure only. It does not define canonical identity, merge behavior, or generalized graph structure.

---

## 2. Terminology

- A **descendant entry** is a non-partner line describing a person in the descendant tree.
- A **partner entry** is a line beginning with `~`.
- The **root entry** is a descendant entry at depth 0.
- A **depth** is the number of indentation levels on a line after indentation normalization.

---

## 3. Structure

- The file MUST represent a descendant tree.
- Each non-empty line MUST describe exactly one entry.
- The file MUST contain exactly one root entry.
- Indentation MUST represent generation depth.
- A line beginning with `~` MUST be interpreted as a partner entry rather than a direct descendant line.
- Children listed after a partner line at the next deeper generation belong to that descendant-partner pair.
- If a child appears with no preceding partner entry at that level, the other parent is unknown or omitted.

Blank lines MAY appear and MUST be ignored by parsers.

---

## 4. Indentation

### 4.1 Indentation Units

- One indentation level is exactly one tab character (`\t`) or four consecutive spaces.
- Tabs and four-space groups MAY be mixed within the same file.

### 4.2 Normalization

- Parsers MUST treat one tab and four spaces as equivalent to one indentation level.
- Parsers MUST interpret indentation in units of indentation levels, not raw characters.

### 4.3 Validation

- Spaces used for indentation MUST appear in groups of exactly four.
- Any number of tabs and/or four-space groups MAY be combined to form indentation.
- Tabs and four-space groups MAY be combined in a single indentation prefix, provided the total resolves to a whole number of indentation levels.

---

## 5. Line Syntax

Each non-empty line has the form:

```text
[indentation][~] names [birth] [death] [comment] [url...]
```

- Leading indentation MUST be interpreted first.
- After indentation is consumed, the optional `~` marker, if present, MUST apply to the entry on that line.
- Remaining fields are separated by one or more whitespace characters.

---

## 6. Field Separation

- Fields on a line MUST be separated by one or more whitespace characters.
- Whitespace includes:
  - space
  - tab
- Parsers MUST treat any sequence of one or more whitespace characters as a single separator between fields.

### 5.1 Constraints

- Leading whitespace MUST be interpreted as indentation first.
- After indentation has been interpreted, any remaining spaces or tabs MUST be treated as field separators.
- Quoted comments (`"..."`) MUST be treated as a single field.
- URLs MUST appear last on a line.

---

## 7. Structure Semantics

- The root entry MUST be a descendant entry.
- A partner entry MUST NOT appear at depth 0.
- A partner entry MUST attach to the nearest preceding descendant entry at the immediately shallower depth.
- A partner entry MUST NOT itself create a new structural parent context for later sibling or child resolution.
- A descendant entry at depth `n + 1` following a partner entry at depth `n` is a child of the owning descendant-partner pair.
- A descendant entry at depth `n + 1` with no active partner entry at depth `n` is a child of the nearest preceding descendant entry at depth `n`.
- A parser MUST reject entries that require jumping more than one depth level deeper than the currently established structural context.
- A valid file MUST contain exactly one descendant entry at depth 0.

---

## 8. Names

- Every non-empty line MUST contain at least one name token.
- If there is only one plain name token, it is a given name.
- If there are multiple plain name tokens, the last plain name token is the birth surname.
- `()` encloses surnames that are not birth surnames, for example married or taken names.
- `[]` encloses alternative spellings, nicknames, or other non-canonical variants.
- `_` inside a name token represents a literal space.
- Parsers MUST interpret `_` inside name tokens as a literal space.

Examples:

- `af_Silversköld`
- `de_la_Cruz`

---

## 9. Dates

- Birth date has no prefix.
- Death date is prefixed with `d.`

Allowed exact forms:

- `YYYY`
- `YYYY-MM`
- `YYYY-MM-DD`

Allowed uncertain forms:

- `YYYY?`
- `YYYY-MM?`

Parsers MUST reject date tokens outside the allowed forms.

---

## 10. Validation Rules

- No more than one entry MAY appear on a line.
- Unquoted free comment text is invalid.
- Dates MUST use the allowed formats.
- A partner line at the root level MUST be rejected.
- A partner line with no preceding descendant entry at the immediately shallower depth MUST be rejected.
- A parser MUST reject indentation that cannot be resolved into whole indentation levels.

This specification does not require parsers to infer missing structural context beyond the rules above.

---

## 11. Example

```text
Johanna Persdotter 1848-12-05
    ~Per Andersson 1850-05-19
    Karl Alfred Persson (Lindberg) 1871-03-02 d.1957-03-31
        ~Anna Maria Forsberg 1872-02-24
        Gustav Hilmer [Hjalmar] Lindberg 1908-10-16 d.1951-04-03
            ~Frida Kristina Jansson 1911-07-31
            Karl Åke Lindberg 1942-02-22
            Anna Margareta Lindberg 1947-03-25
    Emma Charlotta Persson (Svensson Berglund) 1874-11-18
        ~Johan Emil Svensson 1870-04-09
        Nils Verner Svensson 1898-06-14
        ~August Teodor Berglund 1876-01-27
        Elin Maria Berglund 1903-09-02
```

This example is illustrative.

---

## 12. Scope and Non-Goals

This specification defines:

- textual structure,
- indentation rules,
- token separation,
- name and date encoding rules,
- descendant/partner line semantics.

This specification does not define:

- person identifiers,
- cross-file identity,
- merge behavior,
- snapshot semantics,
- generalized family graph modeling.

---

## 13. Versioning Note

`v0.3-dev` introduces the current draft naming and publication shape for Kinshiq. It corresponds conceptually to the latest draft previously maintained in KinLab, while normalizing the name to **Descendant Tree Format**.
