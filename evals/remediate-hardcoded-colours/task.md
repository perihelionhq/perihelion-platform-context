# Fix a conformance finding: hard-coded colours

A conformance sweep reported this finding against the storefront:

> **Colours must come from the theme.** Styled components use hex and rgb() colour literals instead of `theme.colors` tokens from `styles/Theme.ts`. Examples: `components/Button/Button.tsx` uses `#5262a8` (which duplicates the theme's `otelBlue`); `components/Header/Header.styled.ts`, `components/Input/Input.styled.ts`, `components/CurrencySwitcher/CurrencySwitcher.styled.ts`, `components/CartDropdown/CartDropdown.styled.ts` and `styles/Checkout.styled.ts` also contain literals.

Fix this finding. Change only what the finding is about.
