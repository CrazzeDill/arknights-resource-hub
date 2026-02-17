# Fix TypeScript Error in Layout.astro

## Problem Summary

TypeScript error at line 55 in `src/layouts/Layout.astro`:

```
Property 'dataset' does not exist on type 'Element'. (2339)
```

## Root Cause Analysis

The issue occurs because `document.querySelector()` returns `Element | null`, but the `dataset` property only exists on `HTMLElement` (and its subclasses). TypeScript correctly identifies this type mismatch.

### Affected Lines

1. **Line 49:** `const link = document.querySelector('.sidebar-nav a[href="#' + id + '"]');`
   - Returns: `Element | null`
   - Used at line 55: `link.dataset.parent` ❌

2. **Line 58:** `const parentAnchor = document.querySelector('.sidebar-nav a[data-parent="' + parentId + '"]');`
   - Returns: `Element | null`
   - Potential issue if `dataset` is accessed on this element

## Solution

Cast the `querySelector` results to `HTMLElement` since we know these selectors target anchor elements (`<a>`), which are HTML elements.

### Changes Required

#### Change 1: Line 49

**Before:**

```typescript
const link = document.querySelector('.sidebar-nav a[href="#' + id + '"]');
```

**After:**

```typescript
const link = document.querySelector(
  '.sidebar-nav a[href="#' + id + '"]',
) as HTMLElement | null;
```

#### Change 2: Line 58

**Before:**

```typescript
const parentAnchor = document.querySelector(
  '.sidebar-nav a[data-parent="' + parentId + '"]',
);
```

**After:**

```typescript
const parentAnchor = document.querySelector(
  '.sidebar-nav a[data-parent="' + parentId + '"]',
) as HTMLElement | null;
```

## Why This Fix Works

1. **Type Safety:** The cast tells TypeScript that we expect an `HTMLElement` (which has `dataset`), not just a generic `Element`
2. **Runtime Safety:** The existing null checks (`if (!link) return;` and `if (parentAnchor)`) ensure we don't access properties on null values
3. **Correctness:** Since the selectors target `<a>` elements, they will always be `HTMLElement` instances (specifically `HTMLAnchorElement`) when not null

## Alternative Approaches

### Option 1: Use HTMLAnchorElement (More Specific)

```typescript
const link = document.querySelector(
  '.sidebar-nav a[href="#' + id + '"]',
) as HTMLAnchorElement | null;
```

This is more specific since we know these are anchor elements, but `HTMLElement` is sufficient for accessing `dataset`.

### Option 2: Use Non-null Assertion with Type Guard

```typescript
const link = document.querySelector('.sidebar-nav a[href="#' + id + '"]');
if (!link) return;
const parentId = (link as HTMLElement).dataset.parent;
```

This approach casts only where needed, but is more verbose.

## Recommended Solution

Use `HTMLElement` casting at the `querySelector` call sites (Option from "Changes Required" section above) because:

- It's clean and concise
- It makes the type clear from the variable declaration
- It works with the existing null checks
- `HTMLElement` is appropriate for accessing `dataset`

## Testing Considerations

After applying the fix:

1. Verify TypeScript compilation succeeds with no errors
2. Test the scrollspy functionality in the browser to ensure it still works correctly
3. Check that sidebar navigation links are properly highlighted when scrolling
4. Verify parent link activation works as expected

## Files to Modify

- `src/layouts/Layout.astro` (lines 49 and 58)
