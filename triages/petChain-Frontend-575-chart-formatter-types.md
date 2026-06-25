# 🔒 Triage Record — DogStark/petChain-Frontend #575

> **Date:** June 25, 2026 | **By:** [SNTL84](https://github.com/SNTL84)

## Issue Details

| Field | Value |
|-------|-------|
| **Repo** | [DogStark/petChain-Frontend](https://github.com/DogStark/petChain-Frontend) |
| **Issue** | [#575 — Type Chart Formatter Props (Remove `any`)](https://github.com/DogStark/petChain-Frontend/issues/575) |
| **Label** | `good first issue` · `frontend` · `Stellar Wave` |
| **Priority** | Low |
| **Effort** | ~2 hours |
| **Category** | Type Safety / Code Quality |
| **My Comment** | [View Live →](https://github.com/DogStark/petChain-Frontend/issues/575#issuecomment-4797528640) |

---

## Problem Statement

Two Recharts callback props were typed as `any`:

1. `FinancialReportChart.tsx` (line ~42): `formatter={(value: any) => ...}`
2. `PetHealthChart.tsx` (line ~13): `renderCustomizedLabel = (props: any)` + `eslint-disable` comment

**Impact:**
- Silent runtime type mismatches on currency formatting
- `eslint-disable` masking structural misuse
- Inconsistent type discipline across analytics components

---

## Solution Delivered

### File 1: `FinancialReportChart.tsx`

```tsx
// Option A — Official Recharts types
import type { ValueType, NameType } from 'recharts/types/component/DefaultTooltipContent';
formatter={(value: ValueType, _name: NameType) => [`$${formatCurrency(value as number)}`, undefined]}

// Option B — Safe inline (no extra import)
formatter={(value: number | string) => [`$${formatCurrency(Number(value))}`, undefined]}
```

### File 2: `PetHealthChart.tsx`

```tsx
interface PieLabelRenderProps {
  cx: number; cy: number; midAngle: number;
  innerRadius: number; outerRadius: number; percent: number;
}

const renderCustomizedLabel = ({ cx, cy, midAngle, innerRadius, outerRadius, percent }: PieLabelRenderProps) => {
  // ... same logic, no any, no eslint-disable
};
```

---

## Acceptance Criteria Status

- [x] No `any` in `FinancialReportChart.tsx` formatter
- [x] No `any` in `PetHealthChart.tsx` `renderCustomizedLabel`
- [x] `eslint-disable` comment removed
- [x] Types scoped to fields actually used

---

## Extras Offered

- Offered to extend typing pattern to `UserEngagementChart.tsx` and `VaccinationChart.tsx`
- Comparison table of 3 approaches (any / Recharts import / local interface)
- Reasoning for why local interface is more stable across Recharts version bumps

---

## Contributor Signature

> | **SNTL 84** | Agentic AI Workflow Professional |
> Lead Generation · Fulfillment Automation · Full-Stack Builds · AI Workflows · Supply Chain BI
>
> 🌐 [desidevloper.com](https://desidevloper.com) · 💬 [WhatsApp](https://wa.me/919727413309) · 🔗 [LinkedIn](https://linkedin.com/in/sntl2784) · 💻 [GitHub](https://github.com/SNTL84) · 📸 [@desibiztrade](https://www.instagram.com/desibiztrade) · 🔴 [YouTube @SNTL84](https://youtube.com/@SNTL84)
>
> 🚀 *Follow for practical AI automation insights & founder systems.*
