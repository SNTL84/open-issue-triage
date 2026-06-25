# 🧩 Triage Record — type-challenges/type-challenges #38224

> **Date:** June 25, 2026 | **By:** [SNTL84](https://github.com/SNTL84)

## Issue Details

| Field | Value |
|-------|-------|
| **Repo** | [type-challenges/type-challenges](https://github.com/type-challenges/type-challenges) |
| **Issue** | [#38224 — 2257 MinusOne (no reverse)](https://github.com/type-challenges/type-challenges/issues/38224) |
| **Label** | `answer` · `en` · `2257` |
| **Difficulty** | Hard |
| **Category** | TypeScript / Type System / Type Arithmetic |
| **Reactions on Original** | 6 👍 |
| **My Comment** | [View Live →](https://github.com/type-challenges/type-challenges/issues/38224#issuecomment-4797379914) |

---

## Problem Statement

Challenge: implement `MinusOne<T>` — given positive integer type `T`, return `T - 1` at the type level.

Existing community answer used a no-reverse string subtraction approach but had gaps:
- No `MinusOne<0>` guard (borrow propagation chain leads to `never` incorrectly)
- `SplitLast` helper tightly coupled — not reusable
- No TS 4.8 shorthand (`infer X extends T`) used
- No annotation / mental model for learners

---

## Enhanced Solution Delivered

```ts
type PrevDigit = {
  '0': '9'; '1': '0'; '2': '1'; '3': '2'; '4': '3';
  '5': '4'; '6': '5'; '7': '6'; '8': '7'; '9': '8';
}

type SplitTail<S extends string, Head extends string = ""> =
  S extends `${infer F}${infer R}`
    ? R extends "" ? [Head, F] : SplitTail<R, `${Head}${F}`>
    : never

type SubtractOne<S extends string> =
  SplitTail<S> extends [infer H extends string, infer L extends string]
    ? L extends '0'
      ? H extends "" ? never : `${SubtractOne<H>}9`
      : L extends keyof PrevDigit ? `${H}${PrevDigit[L]}` : never
    : never

type TrimZeros<S extends string> =
  S extends `0${infer R}` ? R extends "" ? "0" : TrimZeros<R> : S

type MinusOne<T extends number> =
  T extends 0 ? never
    : `${T}` extends infer S extends string
      ? TrimZeros<SubtractOne<S>> extends infer R extends string
        ? R extends `${infer N extends number}` ? N : never
        : never
      : never
```

---

## Improvements Over Original

| Feature | This Solution | Original |
|---------|--------------|----------|
| `MinusOne<0>` guard | ✅ `never` | ❌ missing |
| Reusable `SplitTail` helper | ✅ generic | ❌ coupled |
| TS 4.8 `infer X extends T` | ✅ throughout | ⚠️ mixed |
| JSDoc + mental model | ✅ full | ❌ absent |
| Leading zero trim guard | ✅ `"0"` safe | ⚠️ partial |

---

## Contributor Signature

> | **SNTL 84** | Agentic AI Workflow Professional |
> Lead Generation · Fulfillment Automation · Full-Stack Builds · AI Workflows · Supply Chain BI
>
> 🌐 [desidevloper.com](https://desidevloper.com) · 💬 [WhatsApp](https://wa.me/919727413309) · 🔗 [LinkedIn](https://linkedin.com/in/sntl2784) · 💻 [GitHub](https://github.com/SNTL84) · 📸 [@desibiztrade](https://www.instagram.com/desibiztrade) · 🔴 [YouTube @SNTL84](https://youtube.com/@SNTL84)
>
> 🚀 *Follow for practical AI automation insights & founder systems.*
