---
always_on: true
description: "Enforces writing clear, self-documenting code and only adding comments when logic is genuinely complicated or hard to read."
---

# Code Commenting Style Rule

**ALWAYS ON**

## Principles

1. **Self-Documenting Code First**: Write code that is clear, succinct, and readable. Variable, function, and class names should express intent.
2. **Minimal Comments**: Only add comments if a method, function, or block of logic is particularly confusing, non-obvious, or hard to read.
3. **Focus on "Why", Not "What"**: Do not write comments that simply restate what the code is doing. If a comment is needed, explain *why* the code is written that way or clarify complex algorithmic steps.

## Examples

### Correct Example:
```typescript
// Good: Self-documenting code needing no comments
export function calculateCartTotal(items: CartItem[]): number {
  return items.reduce((total, item) => total + item.price * item.quantity, 0);
}

// Good: Comment added to explain a complex or non-obvious workaround
export function getLocalizedDate(date: Date, locale: string): string {
  // Safari <14 has a known bug formatting relative times in pt-BR,
  // so we fall back to a manual formatting function if Safari is detected.
  if (isLegacySafari() && locale === 'pt-BR') {
    return formatLegacySafariPortugueseDate(date);
  }
  return new Intl.DateTimeFormat(locale).format(date);
}
```

### Incorrect Example:
```typescript
// ❌ Bad: Redundant comments explaining obvious code
export function calculateCartTotal(items: CartItem[]): number {
  // Initialize total to 0
  let total = 0;
  // Loop through all items
  for (const item of items) {
    // Add item price multiplied by quantity to total
    total += item.price * item.quantity;
  }
  // Return the total
  return total;
}
```
