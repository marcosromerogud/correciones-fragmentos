# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not a software application** — it's a versioned collection of HTML email fragments ("componentes") for BCP (Banco de Crédito del Perú) marketing campaigns, built for **Adobe Campaign**. There is no build system, package manager, linter, or test suite. All work is manual editing of raw HTML/CSS files that get pasted into Adobe Campaign's WYSIWYG editor as custom HTML components.

Every fragment must render correctly across desktop and mobile email clients (Gmail, Outlook desktop/VML, Apple Mail), which is why the markup relies heavily on nested `<table>` layouts, MSO conditional comments (`<!--[if mso]>`/`<!--[if (gte mso 9)|(lte ie 8)]>`), and `!important`-heavy media queries rather than modern CSS.

## Repository structure

- **`[Temporal] BCP - <Component> - <Variant>.html`** — the markup fragment for one banner variant.
- **`[Temporal] BCP - <Component> - <Variant> - CSS.html`** — the companion responsive CSS for that exact variant, wrapped in Adobe Campaign's `acr-tmp-component` block.
- **A markup file and its `- CSS` counterpart are always deployed together**, pasted into the *same stacked container* in Adobe Campaign. The markup file's leading HTML comment states this explicitly (e.g. "Su CSS responsive ... esta en el archivo ... que debe ir en el mismo contenedor apilado"). Never edit one half without checking whether the other half needs the matching change.
  - **This split is a required workaround, not a style choice.** When a fragment that mixes a `<style>` block with heavy markup is "broken up" by AEM's editor, AEM discards the `<style>` block entirely — the markup and `bd-*` classes survive but end up with zero matching rules, so the banner silently loses its responsive behavior and freezes at a fixed 600px. This was confirmed by comparing `banners-roto/ejemplo.html` (1 `<style>`, 3 `@media`) against `banners-roto/roto.html` (0 and 0) after going through that flow. Since responsive behavior here depends on `@media` queries — which cannot be expressed as inline styles — a single self-contained file isn't a viable alternative; the CSS has to live in its own file/component alongside the markup instead of inside it. A shared base CSS file across variants was also tried and rejected because AEM's editor doesn't allow nesting one component as a child of another.
- **`htmlbase1.html` / `htmlbase2.html`** — global/base CSS blocks used across an entire campaign email (resets, shared mobile classes like `.container`, `.col-mobile-1`, `.detect-desktop/.detect-mobile`, dark-mode overrides). Per-banner CSS classes must be scoped/prefixed so they don't collide with these globals (see "Aislar banners del CSS global de htmlbase1/2" in history) — this is why each component family uses its own class prefix instead of generic names.
- **`banners-roto/`** — the known-good vs. known-broken pair (`ejemplo.html` vs `roto.html`) referenced above, proving the "AEM drops `<style>` on split" failure mode. Use it as the reference when validating that a new markup/CSS pair will survive being pasted into AEM.
- **`otros/`** — additional one-off fragments (e.g. "Lista con check" checklist items/titles) with client-segment variants: `Temporal` (generic/testing), `Credicorp`, `Enalta`. These differ only in icon asset URL/size and copy — structure is otherwise identical.

## Naming and class conventions

- Component variants share a structural skeleton but each gets its **own CSS class prefix** to avoid cross-variant collisions when multiple variants are stacked in the same email. E.g. for "Banner destacado": `bd-full` (shared container class) plus variant-specific icon/text classes — `bd-icon-h`/`bd-text-h` (horizontal/"Sin bordes"), `bd-icon-hf`/`bd-text-hf` (Full width), `bd-icon-v`/`bd-text-v` (Vertical). When adding a new variant, mint a new suffix rather than reusing another variant's classes.
- `-dj` suffixed classes (`title-dj`, `subtitle-dj`, `paragraph-dj`, `paragraph-small-dj`, `paragraph-blue-dj`) are shared typographic helper classes from `htmlbase2.html`.
- `mobile-*` / `mobile-width-*` / `mobile-font-size-*` classes are global responsive helpers defined in the base files; `bd-*`-style classes are per-component responsive overrides defined in each fragment's own `- CSS.html` file.
- `acr-*` prefixed ids/classes (`acr-tmp-component`, `acr-tmp-component-id`, auto-generated ids like `acr-1347211898130824`) are Adobe Campaign editor artifacts — don't rename or "clean up" these; they're either required by the platform or auto-generated per-instance.
- The standard responsive breakpoint for full-width/fluid layouts collapsing to mobile is **599px**; a secondary breakpoint at **480px** is used for finer mobile-only tweaks (see history: "Apilar en el mismo breakpoint que el ancho fluido (599px)").

## Working on a fragment: things to check

- **MSO/Outlook parity**: any visual fix (background color, border, radius, spacing) applied to the standard markup usually needs a matching fix inside the `<!--[if mso]>` / `<!--[if (gte mso 9)|(lte ie 8)]>` conditional blocks, since Outlook desktop renders the VML/table-based fallback instead of the CSS. Check commit history patterns like "Corregir el fondo en Outlook" and "Sincronizar el fondo de Outlook" — Outlook drift is a recurring bug class here.
- **Mobile class collisions**: when copying a variant to create a new one, rename its `bd-*`-style classes (icon/text/wrapper) to a variant-specific suffix — do not leave duplicate class names across two variants that might end up stacked in the same email.
- **Never merge a variant's markup and `- CSS` file back into one file**: doing so recreates the exact failure documented in `banners-roto/` (AEM drops the `<style>` block, banner loses responsive and freezes at 600px). Keep the pair split even if it looks redundant.
- **Border vs. background sizing**: when a variant has no visible border ("Sin bordes"/"Sin bordes" but shares a background), the effective padding/fill must still visually align with the bordered variant's edge (see "Igualar el borde al relleno en las variantes sin borde visible").
- **Radius consistency**: shared `table-radius` wrapper class should use one unified `border-radius` value across variants (see "Unificar radius del wrapper table-radius") — don't hardcode a one-off radius on a new variant.
- There is no automated way to preview these fragments; validate changes by checking against sibling variants for structural consistency (same tag depth, same conditional-comment placement) and, when possible, by pasting into an actual Adobe Campaign stacked container or an email-rendering test tool before considering the change done.

## Language

Comments, commit messages, and copy in these fragments are in Spanish — match that convention when adding comments or commit messages.
