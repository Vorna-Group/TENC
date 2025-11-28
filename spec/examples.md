# TENC Examples (0.3)

> Public examples reference for the TENC documentation. Released by Vorna Group LLC. Conceived and implemented by Felix Orlov and Nick Medved.

This document contains practical examples for TENC usage. For the normative rules, see `spec/specification.md`.

---

## 1. Basics

Simple element:

```tenc
<p Hello>
```

Classes, ID, attributes:

```tenc
<div.panel#root(role=note data-x=10 title="hello world") Content>
```

Children and closures:

```tenc
<p Intro <a@u One> End>
<p See <a@u Link>>
<p Intro <a@u One> <a@v Two> End>
<p Intro <a@u One> <a@v Two>>
```

---

## 2. Primary Attribute (`@primary`)

Link (`a[href]`):

```tenc
<a@https://example.com Visit>
```

Image (`img[src]` with extra attrs):

```tenc
<img@/logo.svg(alt="Logo")>
```

For tags without a built-in primary, TENC 0.3 uses `data-primary`:

```tenc
<custom-widget@main(data-role=interactive) Hello>
```

Serializes to HTML with `data-primary="main"`.

---

## 3. Dictionaries and References

Define and reuse values:

```tenc
<%ln(g=https://google.com no="noopener noreferrer" bl=_blank)>
<nav.menu <a@%ln.g(rel=%ln.no target=%ln.bl) Google><a@https://youtube.com(rel=%ln.no target=%ln.bl) YouTube><a@%ln.g(rel=%ln.no) Docs>>
```

Concatenate references:

```tenc
<%ln(g=https://google.com test=/search no="noopener noreferrer" bl=_blank)>
<a@%ln.g%ln.test(rel="%ln.no" target=%ln.bl) Google>
```

Semantics:

- `%ln.g` → `https://google.com`
- `%ln.test` → `/search`
- `%ln.g%ln.test` → `https://google.com/search`

---

## 4. Escaping

TEXT and attribute values require escaping for special characters:

```tenc
<p Show symbols: \< \> \( \) \. \# \@ \\ >
<div(data="x\<y" raw=\(literal\)) OK>
```

Literal `%` in values:

```tenc
<p 50\% complete>
```

---

## 5. HTML Output (illustrative)

From:

```tenc
<%ln(g=https://google.com no="noopener noreferrer" bl=_blank)>
<a@%ln.g(rel=%ln.no target=%ln.bl) Google>
```

HTML:

```html
<a href="https://google.com" rel="noopener noreferrer" target="_blank">Google</a>
```

---

## 6. Invalid (for reference)

These are examples of invalid TENC inputs:

Missing dictionary:

```tenc
<p Link <a@%ln.g Test>>
```

Missing dictionary key:

```tenc
<%ln(g=https://google.com)>
<a@%ln.unknown Test>
```

Malformed attribute pair:

```tenc
<div(role) Content>
```

Unescaped specials in TEXT:

```tenc
<p 1 < 2>
```

Dangling head segment in TEXT:

```tenc
<p Text .lateClass>
```

---

For the full specification and EBNF grammar, see `spec/specification.md`.
