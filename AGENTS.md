# AGENTS.md

> **Note:** This is a training repository for the Spec-Driven BA workshop, not a production system. There is no persistent MongoDB. `npm run dev` starts an in-memory database with demo data, and that database is reset on every start. Data changes do not survive a restart.

## Project

This is a training copy of the Liatoshynsky Foundation public website. A visitor reads about the foundation, opens a few content pages, and can send a contact request. The donation screen is visible, but payment is disabled in this demo. The app is Next.js. Copy lives in `i18n/messages`; pages live in `app/[lang]`.

## Pages

Opening `/` redirects to `/uk`. English pages are under `/en/...`.

The header menu is narrower than this table. It shows About us, Biography, Research, Cooperation, and Support us. Contacts is a “Contact us” button. Artistry, archive, news, and the war page are not in the header.

| Route | What the site shows now |
| --- | --- |
| `/` | Home: foundation intro, biography teaser, mock events, an empty news block, an archive link, and partners |
| `/about-us` | Mission, goals, what the foundation does, and founders. Copy is seeded demo text; the founder is “Demo Founder” |
| `/biography` | Error screen: “Sorry, something went wrong”. No biography is shown |
| `/artistry` | A quote and filters. No list of compositions is shown |
| `/research` | Title, a demo quote, and filters. No list of works is shown |
| `/archive` | Search and a note that originals are kept at the Cabinet-Museum in Kyiv. No funds are loaded |
| `/archive/[fund]` | Meant for cases inside a fund. The demo database has no funds, so nothing opens |
| `/archive/[fund]/[case]` | Meant for a case card (code, dates, sheets, PDF). There is no case to open |
| `/news` | Tabs News, Events, and Media. Tab labels stay Ukrainian on `/en`. The news tab says “No news yet” |
| `/cooperation` | Partnership formats, donations, partners, and a collaboration form. The form does not send email |
| `/support-us` | Amount, currency, and bank details. “Make a donation” is disabled. Volunteer help and the FAQ stay in Ukrainian |
| `/contacts` | Phone, email, and a contact form. The form says a reply comes within a few business days |
| `/war-in-ukraine` | “Russia's war against Ukrainians” and a collection for cars for the AFU |
| `/privacy-policy` | How data is collected and used, plus cookies and related sections |
| `/terms` | Site rules: music library and archive, UAASP copyright, registration and account, behavior on the site |

An unknown route shows the “page not found” screen.

## Conventions

- Write code in TypeScript.
- Do not use `export default`. Export named functions and types.
- Ukrainian (`uk`) is the default locale. Add new message keys to both `i18n/messages/uk.json` and `i18n/messages/en.json`.
