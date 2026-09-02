# NextCore Technologies — Company Site

Static one-page company site for **NextCore Technologies Limited Liability Company**
(Delaware, US), deployed via GitHub Pages on nextcore-technologies.com.

## Wording

The business description on this page mirrors, word for word, the description used
in every account application. Do not change one without the other — inconsistent
descriptions across providers are the most common cause of account review.

## Company details are deliberately not published

Legal name, registered state, EIN, business address and managing member were
removed from the page on 2026-09-02. The business address in particular was
blocking Google and Meta verification. Anything a provider needs is supplied
directly out of the filings, not from this page.

Consequence to be aware of: this page no longer works on its own as the
"verifiable company presence" some banks and PSPs ask for. If an onboarding
process requires visible entity details, they have to go back in.

## Languages

The page ships English and German. The visitor's language is picked from
`navigator.languages`; an unsupported locale falls back to English. A manual
choice via the EN/DE switch always wins and is remembered in `localStorage`
(`nc-lang`). Static hosting has no server-side geo lookup — this follows the
browser's language preference, not the visitor's IP location.

To add a language: add one entry to the `I18N` object and one to `LANG_LABEL`,
both at the top of the inline script. Every translatable node already carries a
`data-i18n` key; `data-i18n-html` marks the two values that contain markup.

## Files

- `index.html` — the entire site: markup, styles and behaviour in one file
- `favicon.svg` — brand mark
- Only external dependency: Google Fonts (Newsreader / Instrument Sans / IBM Plex Mono)
- A print stylesheet renders the page as a clean white document. Test with Cmd-P
  before changing colours.

`service/` and `legal/` are local working documents and are deliberately NOT
committed — anything in this repo is publicly reachable once pushed.
