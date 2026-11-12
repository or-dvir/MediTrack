# MediTrack mockup style

All mockups in `docs/design` share this look. **The whole palette (every color below, including success), the icon and the copy are placeholders** until the palette decision (SPEC Section 9). To change the look, change the tokens here, not screen by screen. Reference images: `login-screen-options.png`, `register-screen-options.png`.

## Tokens

- Screen: 390x844, side padding 24, gap between form items 14.
- Colors (all placeholders): background `#F6FAFA`, field/surface fill `#FFFFFF`, text `#191C1C`, muted text `#4A5757`, primary `#006A6A`, on-primary `#FFFFFF`, tonal container `#CCE8E7`, outline `#6F7979`, divider `#C4CCCB`, error `#B3261C`, success `#2E7D4F` (use as a fill or icon color with white on top; not for text on the tonal container).
- Type: Roboto (system-ui fallback). Screen heading 28/500, top bar title 22/28 regular, body 15-16, buttons 16/500, field label and helper text 12, links 14/500 in primary.

## Components

- **Text field:** outlined, white fill, 60 high, radius 12, 1px outline border, 12px muted label inside at the top, 16px value below. Password fields have a show/hide eye icon (line icon, stroke 2). Helper text: 12 muted, below the field. Error: 2px error border, error-colored label, error text replaces the helper.
- **Primary button:** pill, 52 high, full width, primary fill, white 16/500 text. **Secondary button:** same shape, white fill, 1px outline border, dark text.
- **Text link:** 14/500, primary color, no underline. A disabled link (for example a resend countdown) is muted instead of primary.
- **Top app bar** (register, verification, forgot password): Material 3 small bar, transparent, 24 of status-bar space above. Standard back icon button (48x48, no background, arrow stroke 2) with the title right next to it. The title is shown in the bar on all three screens. Center-aligned bar is the backup (SPEC Section 10.1).
- **Icon badges and illustrations:** a tonal container (rounded square radius 20, or circle) with a simple primary-colored line icon (stroke 2). No gradients, no shadows, no decoration. App icon placeholder: 64 rounded square, radius 20. A status overlay on a badge is a small circle in the error or success color with a white check or X.
- Errors use the error color. Success uses the success color. If a screen needs another accent, add it here first.

## Presentation

- One 390x844 board per screen state, radius 24 with a soft shadow, on a light page (`#EBF0F0`). No phone frames or notches.
- Above each board: an uppercase 13 bold label and a 13 muted one-line description. Page title 26 bold, a footer line 13 muted listing the placeholders. The option to implement comes first, backups after, labeled.
- Embed one PNG per screen in the spec (renders everywhere). Keep the editable source in `docs/design/src` (HTML/CSS rendered with headless Chromium preferred; SVG acceptable).

## Rules

- Stay inside these tokens. If something new is needed, ask Or before adding it here.
- Simple over flashy (SPEC Section 5).
