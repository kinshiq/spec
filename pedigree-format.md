# Kinshiq Pedigree Format v0.3-dev

*Text format for indexed pedigree trees*

Copyright (c) 2026 Stefan Sellman  
Licensed under the MIT License.

---

## Status of this document

This document defines the **Kinshiq Pedigree Format**.

The specification is currently under active development. It is intended to become the canonical shared specification for pedigree-tree text used by Kinshiq tools and related applications.

The canonical version of this document is maintained in the Kinshiq GitHub repository:

https://github.com/kinshiq/spec

Feedback, discussion, and contributions are welcome.

---

## 1. Overview

This specification defines a line-oriented text format for representing a pedigree rooted at one individual.

The format is designed to be:

- human-readable,
- deterministic to parse,
- tolerant of omitted ancestors,
- suitable for interchange, rendering, and validation.

Unlike indentation-only pedigree formats, this version uses an explicit pedigree index. Indentation and visual ordering are presentation only.

This format models pedigree position and line structure only. It does not define canonical identity, merge behavior, or relationship codes.

---

## 2. Terminology

- The **root entry** is the single pedigree entry with no pedigree index.
- A **non-root entry** is any pedigree entry with a pedigree index.
- A **placeholder entry** is a non-root entry that contains an index but omits names.
- The **pedigree index** is the authoritative ancestry-position token on a non-root line.

---

## 3. Structure

- The file MUST represent an indexed pedigree tree.
- Each non-empty line MUST describe exactly one pedigree entry.
- The file MUST contain exactly one root entry.
- The root person MUST NOT have a pedigree index.
- Every non-root person MUST have a pedigree index.
- The pedigree index defines ancestry position.
- A non-root line MAY be a placeholder entry with no name data.
- Indentation and line ordering are presentation only.

Blank lines MAY appear and MUST be ignored by parsers.

---

## 4. Pedigree Index

- The pedigree index appears immediately after any indentation and before any remaining fields on the line.
- The pedigree index is a lowercase string of one or more `f` and `m` characters.
- `f` means father.
- `m` means mother.
- Each additional character moves one generation farther from the root.

Examples:

- `f` = father of the root person
- `m` = mother of the root person
- `ff` = father's father
- `fm` = father's mother
- `mff` = mother's father's father

### 4.1 Syntax

- The index token MUST match `^[fm]+$`.
- If additional fields follow the index token, the index token MUST be followed by at least one whitespace character.
- After indentation, if the first token matches `^[fm]+$`, parsers MUST interpret it as a pedigree index rather than as part of the name.
- Bare leading tokens such as `f`, `m`, `ff`, and `ffmm` are therefore reserved in index position.

### 4.2 Semantics

- The length of the index equals generation depth.
- The index is authoritative.
- Parsers MUST derive pedigree position from the index, not from indentation or nearby lines.
- Missing ancestors MUST NOT affect the meaning of other indexes.
- For example, `fm` always means father's mother, even if `ff` is omitted from the file.

---

## 5. Presentation

- Indentation is optional and is used only for readability.
- If indentation is used, one indentation level is exactly one tab character (`\t`) or four consecutive spaces.
- Tabs and four-space groups MAY be mixed within the same file.
- Spaces used for indentation MUST appear in groups of exactly four.
- Parsers MUST ignore indentation when determining pedigree structure.
- Writers SHOULD indent a line to the same depth as the length of its pedigree index.

---

## 6. Ordering

- Line order does not define pedigree structure.
- Writers SHOULD emit entries in ascending pedigree order.
- The conventional visual order is father above mother:
  - `f` before `m`
  - `ff` before `fm`
  - `mf` before `mm`

---

## 7. Line Syntax

On non-root lines:

```text
[indentation][index] [names] [birth] [death] [comment] [url...]
```

- On non-root lines, all fields after the index are optional.
- A non-root line containing only the index is a valid placeholder entry.
- A placeholder entry MAY also contain a comment, dates, and/or URLs without a name.

On the root line:

```text
[indentation]names [birth] [death] [comment] [url...]
```

- The root line omits the pedigree index.
- The root line MUST contain at least one name token.

### 7.1 Root Recognition

- A parser MUST recognize at most one root entry.
- A line whose first non-indentation token matches `^[fm]+$` MUST be interpreted as a non-root entry.
- A line whose first non-indentation token does not match `^[fm]+$` MUST be interpreted as a root candidate.
- A valid file MUST contain exactly one root candidate.
- If more than one root candidate appears in a file, the file MUST be rejected as invalid unless an implementation explicitly defines a multi-root extension.

---

## 8. Field Separation

- Fields on a line MUST be separated by one or more whitespace characters.
- Whitespace includes:
  - space
  - tab
- Parsers MUST treat any sequence of one or more whitespace characters as a single separator between fields.

### 8.1 Constraints

- Leading whitespace MUST be interpreted as indentation first.
- After indentation has been interpreted, remaining whitespace MUST be treated as field separators.
- Quoted comments (`"..."`) MUST be treated as a single field.
- Inside a quoted comment, a literal `"` MUST be written as `\"`.
- Inside a quoted comment, a literal `\` MUST be written as `\\`.
- URLs MUST appear last.

---

## 9. Placeholder Entries

- A placeholder entry MUST contain a pedigree index.
- A placeholder entry MAY omit names entirely.
- A placeholder entry MAY include comment and URL fields without a name.
- A placeholder entry MAY include dates without a name, but writers SHOULD avoid such usage unless the dates are intentionally attached to an unidentified ancestor slot.
- A placeholder entry MUST NOT be used for the root entry.

---

## 10. Names

- Name tokens are optional on non-root lines.
- The root line always has at least one name token.
- If there is only one plain name token, it is a given name.
- If there are multiple plain name tokens, the last one is the birth surname.
- The emitted birth surname MUST be encoded as exactly one token.
- If a source birth surname contains spaces, writers MUST encode those spaces so that the emitted birth surname remains one token.
- `()` encloses surnames that are not birth surnames.
- `[]` encloses alternative spellings, nicknames, or other variants.
- `_` inside a name token represents a literal space.
- Parsers MUST interpret `_` inside name tokens as a literal space.
- The characters `~`, `(`, `)`, `[`, `]`, `_`, and `\` are reserved within name tokens.
- A writer MUST escape a literal reserved character inside a name token with backslash syntax.
- Parsers MUST interpret `\~`, `\(`, `\)`, `\[`, `\]`, `\_`, and `\\` as literal characters inside the token.

Examples:

- `af_Silversköld`
- `de_la_Cruz`
- `Svensson_Berg`
- `Berg\_ström`

---

## 11. Dates

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

## 12. Validation Rules

- The root person MUST NOT have a pedigree index.
- Every non-root person MUST have a pedigree index.
- A valid file MUST contain exactly one root entry.
- The pedigree index MUST use only lowercase `f` and `m`.
- Root lines MUST contain at least one name token.
- Non-root lines MAY omit the name entirely.
- Duplicate pedigree indexes MUST be rejected.
- No more than one pedigree entry MAY appear on a line.
- Unquoted comment text is invalid.
- Dates MUST use allowed formats.

---

## 13. Example

```text
Karl Erik Andersson 1947-06-18 "Stora Tuna, Dalarna, Sverige"
    f Erik Gunnar Andersson 1917-12-15 d.1985-11-11 "Norr Amsberg, Dalarna, Sverige"
        ff Johan August Andersson 1893 d.1949
            fff Anders Olof Andersson 1872 d.1913
        fm Märta Kristina Larsson 1884-01-08 d.1927-07-02 "Stora Tuna, Dalarna, Sverige"
    m Dagmar Anna Märta Jansson 1923-02-26 "Aspeboda, Dalarna, Sverige"
        mf Albert Teodor Jansson 1891 d.1984
        mm
            mmf "Father unknown. No name for father in birth records"
```

This example is illustrative.

---

## 14. Scope and Non-Goals

This specification defines:

- indexed pedigree positioning,
- optional presentation indentation,
- field structure,
- placeholder ancestor entries,
- name and date encoding rules.

This specification does not define:

- person identifiers,
- merge behavior,
- relationship codes,
- snapshot semantics,
- generalized family graphs.

---

## 15. Versioning Note

`v0.3-dev` introduces the indexed pedigree model as the current Kinshiq draft publication shape. It corresponds conceptually to the latest indexed pedigree draft previously maintained in KinLab.
