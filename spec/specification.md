# TENC Specification — Version 0.3

> Public documentation/specification of the TENC format. Released by Vorna Group LLC. Conceived and implemented by Felix Orlov and Nick Medved.

**Token Efficient Nested Codec**
A compact, reversible, DOM-like markup for LLM prompt compression and lightweight document representation.

---

## 1. Introduction

TENC (Token Efficient Nested Codec) is a compact, single-line, DOM-like notation for:

* LLM prompt compression,
* lightweight, machine-readable documents as a reversible alternative to HTML/XML.

TENC 0.2 extends the 0.1 grammar with **dictionary definitions and references** for repeated attribute values.

TENC 0.3 formalizes the `@primary` mapping across tags and introduces the `data-primary` fallback for tags without a natural primary attribute.

> Non-goal: TENC does **not** define translation/NLP behavior. Those are application/data-layer concerns.

---

## 2. Design Goals

1. Compactness with minimal token overhead.
2. Deterministic parsing and edit stability.
3. Full, lossless HTML round-trip at the DOM/tree level; canonical, dictionary-free TENC remains 1:1 with HTML.
4. Simple, line-oriented representation.
5. Semantic neutrality (no built-in types).
6. Strict superset of Mini-DOM.
7. Support reuse/compression of repeated attribute and primary values through **dictionary macros**.

---

## 3. Relationship to Mini-DOM

* Every valid Mini-DOM string is valid TENC.
* TENC clarifies: escaping (TEXT and attributes), head segment ordering, and closure rules.
* Mini-DOM is the syntactic substrate; TENC is the formal spec.

---

## 4. Core Concepts

A TENC document is a sequence of **dictionary definitions** (optional, non-DOM, compression-oriented metadata) followed by **elements** (DOM-like nodes).

Canonical element pattern:

```tenc
<tag[.class...][#id][@primary](attr=value ... ) TEXT <child> ... >
```

Components may be omitted as needed. Elements can contain inline TEXT and/or nested child elements.

---

## 5. Element Structure

### 5.1 Tag

* Immediately after `<`.
* Characters: `a–z`, `A–Z`, `0–9`, `-`.
* Examples: `<p>`, `<div>`, `<span.highlight>`.

### 5.2 Classes

* Zero or more classes, each prefixed by `.` and attached to the element’s head:

```tenc
<p.lead>
<div.card.primary>
```

* No intrinsic semantics. `.test` is literally class `test`.

### 5.3 ID

* Optional, at most once, prefixed by `#`: `<div#root>`.

### 5.4 Primary attributes

In TENC, the `@` segment encodes the **primary attribute** of an element:

```tenc
<a@https://example.com Docs>
<img@/logo.svg(alt="Logo")>
```

On the HTML side, the primary attribute is a concrete HTML attribute such as `href`, `src`, `name`, etc. For example:

* `<a@…>` ↔ `<a href="…">`
* `<img@…>` ↔ `<img src="…">`

TENC 0.3 formalizes this mapping, defines which tags have a built-in primary attribute, and introduces a consistent fallback using `data-primary` for tags that do not.

---

#### 1. Primary attribute map (TENC 0.3)

The following HTML tags have a **built-in primary attribute** in TENC 0.3.
For each tag `T`, the primary attribute name is `P`:

|        Tag | Primary attribute | Notes                                                  |
| ---------: | ----------------- | ------------------------------------------------------ |
|        `a` | `href`            | Target URL of the link.                                |
|     `abbr` | `title`           | Expanded version of the abbreviation.                  |
|     `area` | `href`            | Target URL for the image map area.                     |
|    `audio` | `src`             | Source URL (if not using nested `<source>`).           |
|     `base` | `href`            | Base URL for relative links in the document.           |
|   `button` | `type`            | Behavioral type: `submit`, `button`, `reset`, etc.     |
|     `data` | `value`           | Machine-readable value associated with the text.       |
|    `embed` | `src`             | Resource URL to embed.                                 |
|     `form` | `action`          | Submission URL of the form.                            |
|     `html` | `lang`            | Language code of the document.                         |
|   `iframe` | `src`             | Embedded document URL.                                 |
|      `img` | `src`             | Image source URL.                                      |
|    `input` | `name`            | Field name in form submission.                         |
|    `label` | `for`             | ID of the associated form control.                     |
|     `link` | `href`            | Linked resource (CSS, icon, etc.).                     |
|      `map` | `name`            | Name referenced by `usemap`.                           |
|    `meter` | `value`           | Measured value within a range.                         |
|     `meta` | `name`            | Name of the metadata entry (for `name/content` pairs). |
|   `object` | `data`            | Resource URL/data to load.                             |
|   `option` | `value`           | Value submitted when this option is selected.          |
| `progress` | `value`           | Progress value (relative to `max`).                    |
|   `script` | `src`             | External script URL.                                   |
|   `select` | `name`            | Field name in form submission.                         |
|   `source` | `src`             | Media source URL (video/audio/picture).                |
| `textarea` | `name`            | Field name in form submission.                         |
|     `time` | `datetime`        | Machine-readable date/time.                            |
|    `track` | `src`             | Text track (subtitles, captions, etc.) resource URL.   |
|    `video` | `src`             | Video source URL (if not using nested `<source>`).     |

For these tags:

* TENC: `<T@VALUE …>`
* HTML: `<T P="VALUE" …>`

where `P` is the primary attribute from the table above.

---

#### 2. Fallback primary for tags without a mapping: `data-primary`

For any tag **not** listed in the table above, TENC 0.3 uses the synthetic attribute name `data-primary` as its primary key.

Let `T` be a tag **without** a built-in primary attribute:

* **TENC → HTML**

  * `<T@VALUE …>` MUST be serialized as:
    `<T data-primary="VALUE" …>`.
* **HTML → TENC**

  * If `<T data-primary="VALUE" …>` is encountered, then:

    * `data-primary` MUST be encoded as the TENC primary segment:
      `<T@VALUE (…) …>`
      and MUST be removed from the attribute block.

This guarantees that **every tag can have a primary value** in TENC, even if HTML does not define a natural primary attribute for that element.

> Summary:
>
> * Tags in the table above: primary is the concrete attribute (e.g. `href`, `src`, `name`).
> * All other tags: primary is `data-primary`.

---

#### 3. Relationship between `@primary` and the attribute block

Each element head in TENC has at most one `@primary` segment and at most one attribute block:

```tenc
<tag@PRIMARY (key1=val1 key2="val2") TEXT>
```

Let `P` be the primary attribute name for tag `T`:

* If `T` is in the primary map (section 1), `P =` that mapped attribute.
* Otherwise, `P = "data-primary"` (section 2).

If **the same attribute `P`** appears both as the primary (`@…`) and in the attribute block, **the primary wins**.

Formally, for an element:

```tenc
<T@VALUE (P=OTHER_VALUE other=… ) …>
```

the resolver MUST behave as follows:

1. **TENC → HTML**

   * The effective value of attribute `P` MUST be `VALUE` (from the `@` segment).
   * The conflicting `P=OTHER_VALUE` inside the attribute block:

     * MUST NOT override `VALUE`;
     * MAY be ignored, dropped, or flagged as a validation/authoring error, depending on the implementation.

2. **HTML → TENC**

   * Implementations SHOULD avoid generating such conflicts.
   * If a conflict exists in input TENC (e.g. handwritten code), `@VALUE` is considered the canonical source of truth.

In other words:

> If a tag has a primary attribute and the same attribute appears inside the parentheses, the value from `@primary` takes precedence over the one in the attribute block.

This rule applies both to **mapped primaries** (e.g. `href` on `<a>`) and the **fallback `data-primary`**.

---

#### 4. Parsing rules (HTML → TENC) with primary attributes

For each HTML element `<T …>`:

1. **Determine the primary attribute name**:

   * If `T` is in the primary map (section 1), `P =` mapped primary (e.g. `href`, `src`, `name`).
   * Otherwise, `P = "data-primary"`.

2. **Check whether `P` is present**:

   * If `<T … P="VALUE" …>` is present:

     * Emit `@VALUE` in the TENC element head:
       `<T@VALUE (…) …>`.
     * Remove attribute `P` from the attribute block portion (`(…)`), leaving only non-primary attributes there.
   * If `P` is absent:

     * Do not emit a primary segment:
       `<T(key1=val1 key2="val 2") …>`
     * All attributes remain inside the attribute block.

These rules ensure a **stable, reversible mapping** between HTML and TENC for:

* tags with a well-defined primary attribute, and
* tags relying on the generic `data-primary` fallback.

### 5.5 Attribute Block

* Optional parenthesized key/value pairs:

```tenc
(role=user data-id=42 title="hello world")
```

* Pairs separated by a single space.
* Values may be quoted (see Escaping).

### 5.6 TEXT

* Unstructured text after the head (tag/classes/id/primary/attrs).
* Special characters follow Escaping rules.
* Head/TEXT boundary: encoders emit exactly one space after the head as a separator. Decoders MUST treat only one trailing space as part of the head; any additional leading spaces belong to TEXT and are permitted (they are not considered head mutation).

**Head segment ordering.** After the tag, any number of head segments may appear in **any order** until TEXT begins: (unchanged)

* `.class` (repeatable),
* `#id` (≤1),
* `@primary` (≤1),
* `(attributes)` (≤1).

---

## 6. Dictionary Definitions

### 6.1 Syntax

A **dictionary definition** is a top-level, non-DOM construct:

```tenc
<%dictname(key1=value1 key2=value2 ... )>
```

Examples:

```tenc
<%ln(g=https://google.com no="noopener noreferrer" bl=_blank)>
<%x(y=5 z=9)>
```

Where:

* `dictname` is an identifier (same `ident` rules as tags/attrs).
* Inside `()` are **dictionary pairs**: `key=value` separated by a single space.
* `key` is a dictionary key (identifier).
* `value` follows the same lexical form and escaping rules as attribute values (§8).

All dictionary definitions MUST appear **before** the first normal element in the document.

Dictionaries are not HTML elements and do not correspond to DOM nodes; they are metadata for compression and re-use of values.

### 6.2 Semantics

For each dictionary definition `<%dictname(...)>`:

* A logical dictionary **Dict[dictname]** is created.
* For each pair `key=value`:

  * The lexical `value` is parsed as a normal TENC attribute value (including escapes and optional quotes).
  * The **semantic value** of the dictionary entry is the unescaped text content of the value, without enclosing quotes if present.

Example:

```tenc
<%ln(g=https://google.com no="noopener noreferrer" bl=_blank)>
```

Semantically:

* `Dict["ln"]["g"]   = "https://google.com"`
* `Dict["ln"]["no"]  = "noopener noreferrer"`
* `Dict["ln"]["bl"]  = "_blank"`

Dictionaries MUST NOT reference other dictionaries in their own values in 0.2 (no `%dict.key` inside dictionary values).

---

## 7. Dictionary References

### 7.1 Syntax

A **dictionary reference** is a macro-like token:

```tenc
%dictname.key
```

Where:

* `dictname` is the dictionary name defined by `<%dictname(...)>`.
* `key` is a key defined inside that dictionary.
* The reference starts at a non-escaped `%` and continues through:

  * `dictname` (`ident`),
  * a literal `.`,
  * `key` (`ident`).
* The reference ends at the first character that is **not** `[a-zA-Z0-9-]` after the `key`.

Examples:

```tenc
%ln.g          // refers to Dict["ln"]["g"]
%ln.no         // refers to Dict["ln"]["no"]
%ln.g%ln.test  // two adjacent references (concatenation)
```

Dictionary references MAY appear only inside attribute values and primary attribute values in TENC 0.2. They MUST NOT appear in element TEXT in 0.2.

### 7.2 Semantics

At the semantic (expansion) level:

* `%dictname.key` is replaced by the stored semantic value `Dict[dictname][key]`.
* If several dictionary references are adjacent with no other characters in between, their expansions are **concatenated**:

  ```tenc
  <%ln(g=https://google.com test=/search)>
  @%ln.g%ln.test
  ```

  Expands to:

  ```tenc
  @https://google.com/search
  ```

* If a dictionary or key is missing, the document is **semantically invalid** (error).

---

## 8. Children and Closures

* Every element ends with exactly one `>`.
* Child elements end with `>`; if a parent also ends immediately after a child, you will observe adjacent `>` characters. These are simply consecutive single closers.

Patterns:

```tenc
<p Hello>
<p Intro <a@u One> End>
<p See <a@u Link>>
<p Intro <a@u One> <a@v Two> End>
<p Intro <a@u One> <a@v Two>>
```

---

## 9. Special Symbols

| Symbol    | Meaning                                         |
| --------- | ----------------------------------------------- |
| `<`       | Start of an element                             |
| `>`       | End of the current element                      |
| `.`       | Class prefix                                    |
| `#`       | ID prefix                                       |
| `@`       | Primary attribute prefix                        |
| `( )`     | Attribute block                                 |
| `"`       | String literal delimiter for attribute values   |
| `\`       | Escape prefix                                   |
| `%`       | Dictionary reference prefix and special literal |

---

## 10. Escaping Rules (TEXT and Attributes)

### 10.1 Required escapes (literal usage)

When these characters must appear **literally** in TEXT **or** in attribute values (quoted or unquoted), they MUST be escaped with a leading backslash:

```tenc
\<  \>  \(  \)  \#  \@  \\  \%
```

Additionally, within **quoted** attribute values, `"` MUST be escaped:

```tenc
\"
```

Any unescaped `%` in an attribute value is interpreted as the beginning of a dictionary reference in TENC 0.2. To produce a literal `%`, use `\%`.

10.2 TEXT

    Apply the required escapes above whenever the character would otherwise be parsed as syntax.

10.3 Attribute values

    Unquoted values: MUST escape any of the required characters if used literally; avoid spaces (use quotes if spaces are needed).
    Quoted values: MUST escape "; MUST also escape any of the required characters when they are intended literally (recommended always for consistency). Quoted values may contain spaces.

10.4 Guidance

    Prefer quoting attribute values that contain spaces or multiple escaped characters for readability.
    Backslash escapes itself: \\.
    Dot (.) is NOT special in TEXT or in values; escaping '.' is optional, not required.


---

## 11. Grammar (EBNF) + Well-Formedness Constraints

### 11.1 Grammar (syntax)

```ebnf
document   = dictdef* element* ;

dictdef    = "<%" dictname "(" dictpair ( " " dictpair )* ")" ">" ;

dictname   = ident ;
dictkey    = ident ;
dictpair   = dictkey "=" value ;

element    = "<" head content children? ">" ;

head       = tag headseg* ;

headseg    = class | id | primary | attrs ;

tag        = ident ;
class      = "." ident ;
id         = "#" ident ;
primary    = "@" value ;
attrs      = "(" attrpair ( " " attrpair )* ")" ;
attrpair   = ident "=" value ;

content    = text? ;
children   = element+ ;

text       = { character - "<" - ">" } ;

ident      = letter { letter | digit | "-" } ;

value      = quoted | unquoted ;

quoted     = "\"" { character - "\"" | '\"' | escape } "\"" ;
unquoted   = { character - " " - ")" } ;

escape     = "\\" ( "<" | ">" | "(" | ")" | "#" | "@" | "\\" | "\"" | "%" ) ;

letter     = "a"…"z" | "A"…"Z" ;
digit      = "0"…"9" ;
```

Dictionary references `%dictname.key` are recognized **semantically** inside `value` when a non-escaped `%` is followed by `ident "." ident`.

### 11.2 Well-Formedness constraints (semantics)

In addition to 0.1 constraints:

* At most one `id` segment per element.
* At most one `primary` segment per element.
* At most one `attrs` block per element.
* Any number of `class` segments.
* Head segments may appear in any order prior to TEXT.
* Each element must end with one `>`.

TENC 0.2 adds:

* All dictionary definitions MUST appear before the first element.
* Dictionary names (`dictname`) MUST be unique within a document.
* Dictionary keys (`dictkey`) MUST be unique within each dictionary.
* Dictionary values MUST NOT contain unescaped `%` or dictionary references in 0.2.
* Every dictionary reference `%dictname.key` MUST refer to an existing dictionary and existing key.
* Dictionary references MAY only appear inside attribute and primary values, not in TEXT, in 0.2.

---

## 12. HTML Reversibility

Base mapping (unchanged from 0.1):

* `.c1.c2` → `class="c1 c2"`.
* `#id` → `id="id"`.
* `@value` → tag-dependent primary attribute (e.g., `a[href]`).
* `(k=v …)` → `k="v" …` (after unescaping).
* TEXT → text node (after unescaping).
* `<child>` inside a parent → nested HTML node.
* Adjacent `>` at the end encode consecutive end tags in proper order.

0.2 extension — dictionaries:

* **Dictionary-free TENC** remains a canonical, 1:1 textual representation for HTML.
* **Dictionary-based TENC** is an **optional compression layer**:

  * TENC→HTML:

    * Dictionary definitions are ignored as non-DOM metadata.
    * All dictionary references `%dictname.key` in values are expanded using the dictionaries.
    * Concatenation is simply adjacency of references (and optionally literals, in future versions).
  * HTML→TENC:

    * An encoder MAY output a dictionary-free TENC representation.
    * An encoder MAY optionally build one or more dictionaries and rewrite repeated values as dictionary references.

TENC 0.2 therefore guarantees **lossless round-trip at the tree/DOM level**, while the textual TENC representation is no longer unique when dictionaries are used.

---

## 13. Semantic Rules

* TENC remains semantically neutral. No reserved class or attribute names. `.test` is the literal class `test`.
* Dictionaries are purely a **compression and reuse mechanism** for values; they do not introduce semantics beyond value substitution.

---

## 14. Examples

### 14.1 Simple dictionary reuse

```tenc
<%ln(g=https://google.com no="noopener noreferrer" bl=_blank)>
<nav.menu <a@%ln.g(rel=%ln.no target=%ln.bl) Google><a@https://youtube.com(rel=%ln.no target=%ln.bl) YouTube><a@%ln.g(rel=%ln.no) Docs>>
```

Expands to HTML:

```html
<nav class="menu">
  <a href="https://google.com" rel="noopener noreferrer" target="_blank">Google</a>
  <a href="https://youtube.com" rel="noopener noreferrer" target="_blank">YouTube</a>
  <a href="https://google.com" rel="noopener noreferrer">Docs</a>
</nav>
```

### 14.2 Concatenation of dictionary values
```tenc
<%ln(g=https://google.com test=/search no="noopener noreferrer" bl=_blank)>
<a@%ln.g%ln.test(rel="%ln.no" target=%ln.bl) Google>
```

Semantically:

* `%ln.g` → `https://google.com`
* `%ln.test` → `/search`
* `%ln.g%ln.test` → `https://google.com/search`

HTML:

```html
<a href="https://google.com/search" rel="noopener noreferrer" target="_blank">Google</a>
```

### 14.3 Basic valid examples

Simple

```tenc
<p Hello>
```

Classes, ID, attributes

```tenc
<div.panel#root(role=note data-x=10 title="hello world") Content>
```

Primary attribute (common a[href])

```tenc
<a@https://example.com Visit>
```

Child in the middle, trailing TEXT

```tenc
<p Intro <a@u One> End>
```

Last token is a child (adjacent closures)

```tenc
<p See <a@u Link>>
```

Multiple children and trailing TEXT

```tenc
<p Intro <a@u One> <a@v Two> End>
```

Multiple children, last child ends the parent

```tenc
<p Intro <a@u One> <a@v Two>>
```

Escapes (TEXT and attributes)

```tenc
<p Show symbols: \< \> \( \) \. \# \@ \\ >
<div(data="x\<y" raw=\(literal\)) OK>
```

---

## 15. Invalid Structures

In addition to 0.1 invalid examples (missing closure, unescaped specials, malformed attrs, etc.) , TENC 0.2 considers the following invalid:

**Missing dictionary**

```tenc
<p Link <a@%ln.g Test>>
```

No `<%ln(... )>` defined.

**Missing dictionary key**

```tenc
<%ln(g=https://google.com)>
<a@%ln.unknown Test>
```

Key `unknown` not present in `ln`.

**Dictionary after element**

```tenc
<p Hello>
<%ln(g=https://google.com)>
```

Dictionary appears after first element.

**Unescaped `%` when literal is intended**

```tenc
<p 50% complete>
```

Should be:

```tenc
<p 50\% complete>
```

## 16. Invalid Structures (additional)

**Missing parent closure**

```tenc
<p Text<a@u Link>            ❌ parent not closed
```

**Unescaped specials in TEXT**

```tenc
<p 1 < 2>                    ❌ use \< in TEXT
```

**Malformed attribute pair**

```tenc
<div(role) Content>          ❌ must be key=value
```

**Dangling head segment in TEXT**

```tenc
<p Text .lateClass>          ❌ classes belong to head, before TEXT
```

---

## 16. Versioning

```text
TENC Version: 0.3
Status: Pre-final
Compatibility: Strict superset of Mini-DOM and TENC 0.1 and TENC 0.2 syntax; adds more primary attributes.
```

Backward compatibility at the semantic level is preserved for documents that do not use dictionaries.

---
