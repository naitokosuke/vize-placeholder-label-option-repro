# vize-placeholder-label-option-repro

Minimal reproduction for a false positive in [vize](https://github.com/ubugeeei-prod/vize)'s `a11y/placeholder-label-option`.

The rule flags every `<select>` whose first `<option>` has `value=""` and asks for `disabled` or `hidden` on it, whether or not the select is `required`. Here the select is not `required`, and "Any" is a real choice. With `disabled`, a value once picked could never be cleared.

## Steps

```sh
pnpm install
pnpm lint
```

The rule is not in the default preset, so the `lint` script passes `--preset opinionated`.

[`src/FruitSelect.vue`](./src/FruitSelect.vue):

```vue
<template>
  <select aria-label="Fruit">
    <option value="">Any</option>
    <option value="apple">Apple</option>
  </select>
</template>
```

## Expected

No warning. The select is not `required`, so it has no placeholder label option.

## Actual

```
Patina lint report: 1 warning in 1 file

src/FruitSelect.vue
  src/FruitSelect.vue:3:5 warning a11y/placeholder-label-option Placeholder <option> should have disabled or hidden attribute
    Reference: docs/content/rules/accessibility.md
```

## Notes

- The rule says it is [based on markuplint's `placeholder-label-option`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/opinionated/a11y/placeholder_label_option.rs#L1-L6).
  - [markuplint's rule](https://markuplint.dev/docs/rules/placeholder-label-option) applies only to a `required`, non-`multiple` `<select>` with a display size of 1.
  - It asks that the first option have `value=""`. It doesn't ask for `disabled`.
- The HTML spec's [placeholder label option](https://html.spec.whatwg.org/multipage/form-elements.html#placeholder-label-option) likewise exists only for such a `required` select.
- [`check_element`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/opinionated/a11y/placeholder_label_option.rs#L96-L122) looks at neither `required` nor `multiple`; a `multiple` select is flagged too.

## Environment

vize 0.476.0, Node v26.10.0, pnpm 12.9.0, macOS (arm64)
