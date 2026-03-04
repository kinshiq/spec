# Kinshiq Core Specification v0.6.4

*MRCA-based encoding of biological pedigree relationships*

Copyright (c) 2026 Stefan Sellman  
Licensed under the MIT License.

---

## Status of this document

This document defines the **Kinshiq Core Specification**.

The specification is currently under active development. The design is considered stable but may evolve as the system is tested through implementations and real-world usage.

The canonical version of this document is maintained in the Kinshiq GitHub repository:

https://github.com/kinshiq/spec

Feedback, discussion, and contributions are welcome.

---

## 1. Overview

This specification defines **Kinshiq** (/ˈkɪnʃɪk/, *KIN-shik*), a formal language for representing biological relationships between two individuals using structured symbolic expressions called **qodes** (/koʊd/, *code*).

Kinshiq encodes pedigree relationships independently of human kinship terminology by describing the descent structure connecting two individuals relative to shared ancestors within a pedigree directed acyclic graph (DAG).

A **qode** represents an oriented biological relationship. Each qode has a unique canonical form, called a **cqode** (/ˈsiːkoʊd/, *SEE-code*), used for comparison, indexing, and identity.

Multiple distinct qodes MAY represent the same biological relationship. Canonicalization produces a cqode that serves as the unique canonical identity of that relationship.

Kinshiq is designed to have several desirable formal properties. The language is deterministic and produces a unique canonical representation for every relationship. Under the pedigree model defined in this specification, Kinshiq is mathematically complete for biological pedigree relationships, meaning that any relationship between two individuals in a pedigree DAG can be represented. The representation directly encodes the descent paths connecting the endpoints rather than relying on ambiguous natural-language kinship terms.

Because qodes consist of independent MRCA-anchored clauses, Kinshiq naturally supports relationships involving multiple shared ancestors. This includes cases such as pedigree collapse, endogamy, and double-cousin structures, where individuals share more than one ancestral pathway. Each pathway is represented by a separate clause.

Kinshiq Core models biological pedigree topology only. It does not include identifiers, metadata, DNA inheritance behavior, or non-biological relationship types. Such modeling belongs to Kinshiq extensions.

---

## 2. Model Assumptions

Kinshiq operates under the following assumptions:

- Each individual has exactly one biological mother and one biological father.
- Parent–child relationships form a directed acyclic graph (DAG).
- No individual is their own ancestor.
- Relationships are defined relative to two known endpoints.

All qodes describe relationships within such a pedigree structure.

---

## 3. MRCA Anchoring

A qode consists of one or more **clauses**.

In many relationships, the endpoints share two distinct individual MRCAs that form a mating pair (for example, full siblings share both parents, and full first cousins share two grandparents). In such cases, the qode will contain one clause anchored at the female ancestor (X) and one clause anchored at the male ancestor (Y) with matching legs. During canonicalization, each such matching X/Y pair is compressed into a single C clause.

Each clause represents a relationship relative to a Most Recent Common Ancestor (MRCA) and has the form:

```
Leg1 + T + Leg2
```

where:

- `T` denotes the MRCA type,
- `Leg1` describes descent from the MRCA to Endpoint A,
- `Leg2` describes descent from the MRCA to Endpoint B.

A **leg** is a sequence of symbols representing parent→child steps from the MRCA to an endpoint.

Legs are written left-to-right, excluding the MRCA and including the endpoint.

Each symbol encodes one generational step and the biological sex of the child reached by that step.

---

## 4. Alphabet

### 4.1 Descent Symbols

- `S` — son (male child)
- `D` — daughter (female child)

### 4.2 MRCA Symbols

- `X` — female MRCA
- `Y` — male MRCA
- `C` — couple MRCA
  (canonical compression of matching X/Y clauses)

---

## 5. Core Syntax

A qode is defined by the grammar:

```
Clause := Leg? T Leg?
Leg := [SD]*
T := X | Y
Qode := Clause ('.' Clause)*
```

The separator `.` is the literal character U+002E.

### Character Set

Kinshiq qodes are finite strings over:

```
{ S, D, X, Y, . }
```

Whitespace and lowercase letters are invalid inside qodes.

---

## 6. Clause Validity

A clause is valid if and only if:

- If `T ∈ {X, Y}`, then either:
  - both legs are empty (self relationship), or
  - exactly one leg is empty, or
  - both legs are non-empty.

---

## 7. Expression Consistency

All clauses within a qode describe the same two endpoints.

Derived properties (such as endpoint biological sex) MUST be consistent across all clauses.

Parsers MUST reject qodes where endpoint properties conflict.

---

## 8. Representations

Kinshiq distinguishes between two representations of a relationship.

### 8.1 qode (Oriented Form)

A qode preserves endpoint ordering and construction order.

Properties:

- Leg1 corresponds to Endpoint A.
- Leg2 corresponds to Endpoint B.
- Clauses in a qode MUST be written in sorted order.
- Clauses are sorted primarily by their total length (shorter clauses first).
- Clauses of equal length MUST be ordered lexicographically.
- This ordering rule applies to qodes as written and is independent of canonicalization.

qodes are suitable for construction, explanation, and execution.

Symbols `X` and `Y` arise naturally in oriented form.

The symbol `C` MUST NOT appear in qodes. It is introduced only during canonicalization and therefore appears only in cqodes.

### 8.2 cqode (Canonical Form)

A cqode is the unique normalized form of a qode.

Canonicalization:

- removes endpoint directionality,
- normalizes leg ordering,
- deterministically orders clauses,
- may introduce symbol `C`.

cqodes provide a unique canonical identity for relationships and SHOULD be used for:

- comparison,
- hashing,
- indexing,
- identity of relationships.

These operations MAY be performed using qodes, but results are not guaranteed to be canonical or unique.

Every valid qode has exactly one cqode.

### 8.3 Executable qodes

Given:

- a pedigree DAG,
- one known endpoint,
- and a qode,

the second endpoint and the connecting ancestry paths can be deterministically reconstructed by traversing the legs relative to the starting endpoint.

cqodes are symmetric and therefore not directly executable without assigning endpoint orientation.

Execution operates on qodes.

---

## 9. Derived Properties

The following properties can be derived directly from the structure of a qode:

- The final symbol of a non-empty leg determines the biological sex of the endpoint (`S` = male, `D` = female).
- If a leg is empty, the MRCA symbol (`X` or `Y`) determines the biological sex of the endpoint.
- The length of a leg indicates the number of generations between the endpoint and the MRCA.

---

## 10. Canonicalization (Normative)

Canonicalization converts a qode into its cqode.

The input to canonicalization MUST be a valid qode. In particular, the symbol `C` MUST NOT appear in the input.

### Step 1 — Clause Normalization

For each clause:

```
key(Leg1) ≤ key(Leg2)
```

where:

```
key(L) = (len(L), L)
```

Lexicographic ordering uses:

```
D < S
```

Swap legs if violated.

*Non-normative explanation:* This step ensures the two legs inside a clause are always written in the same order. In simple terms, the shortest leg comes first. If both legs are the same length, the order is decided alphabetically (`D` before `S`). This prevents the same relationship from appearing in two different forms just because the legs were swapped.

### Step 2 — Couple Compression

For each `(Leg1, Leg2)` pair:

- let `nX` be count of `(X, Leg1, Leg2)`
- let `nY` be count of `(Y, Leg1, Leg2)`
- let `k = min(nX, nY)`

Replace `k` matching X/Y pairs with:

```
(Leg1 C Leg2)
```

Remaining clauses remain unchanged.

*Non-normative explanation:* If the same ancestor pair appears once as a female (`X`) and once as a male (`Y`) with identical legs, they represent a couple. Instead of keeping both clauses, they are merged into a single `C` clause.

### Step 3 — Expression Ordering

Sort clauses lexicographically by:

```
(T, Leg1, Leg2)
```

with ordering:

```
C < X < Y
```

Join clauses using `.`.

Duplicate clauses are permitted and significant.

Duplicate clauses indicate multiple distinct common ancestors that produce identical descent structures between the endpoints (for example double cousins).

*Non-normative explanation:* Finally, the clauses are sorted into a consistent order so that the final cqode string is always written the same way. This guarantees that two equivalent relationships produce the exact same cqode text.

---

## 11. Clause Independence

Each clause represents an independent MRCA relationship between the endpoints.

Kinshiq Core does not model dependency between shared ancestors. Such modeling belongs to Kinshiq extension specifications.

---

## 12. Examples

| Relationship                         | qode        | cqode  |
| ------------------------------------ | ----------- | ------ |
| Self (male)                          | Y           | Y      |
| Self (female)                        | X           | X      |
| Father–son                           | YS          | YS     |
| Daughter–mother                      | DX          | XD     |
| Full siblings                        | SXS.SYS     | SCS    |
| Half siblings (shared father)        | SYS         | SYS    |
| Grandson-Grandmother (father's side) | SSX         | XSS    |
| Uncle-niece (mother's side)          | SXDD.SYDD   | SCDD   |
| First cousins (example 1)            | DDYSS.DDXSS | DDCSS  |
| First cousins (example 2)            | SSYSS.SSXSS | SSCSS  |
| You figure it out...                 | YS.SXS      | SXS.YS |

---

### Non-Normative Note

Human relationship labels often collapse multiple distinct Kinshiq structures into a single term. For example, “first cousins” can produce ten distinct cqodes depending on the biological sexes of the intermediate ancestors and endpoints:

- DDCDD
- DDCDS
- DDCSD
- DDCSS
- DSCDS
- DSCSD
- DSCSS
- SDCSD
- SDCSS
- SSCSS

All of these relationships are commonly described simply as “first cousins” in natural language, even though they represent structurally distinct descent paths.

---

## 13. Terminology and Naming Conventions

### 13.1 System Name

**Kinshiq** is the proper name of the language and specification.

- “Kinshiq” MUST be capitalized when referring to the system or specification.
- The lowercase form `kinshiq` MAY be used for command names, package identifiers, repositories, and code examples.

### 13.2 Core Object Names

The fundamental representation defined by Kinshiq is a qode.

- The terms qode and cqode are always written in lowercase.
- Capitalized forms MUST NOT be used.

Correct examples:

- a qode
- derive the cqode

Incorrect examples:

- Qode
- Cqode

### 13.3 Redundant Terminology

Because a qode already denotes a coded representation, redundant phrases MUST be avoided.

Incorrect:

- qode code
- Kinshiq qode code

Preferred:

- qode
- Kinshiq qode

### 13.4 Quotation Marks

In running text, qodes SHOULD be written without quotation marks such as ' or ".

Correct:

- The qode SXS.SYS represents full siblings.

Incorrect:

- The qode "SXS.SYS" represents full siblings.
