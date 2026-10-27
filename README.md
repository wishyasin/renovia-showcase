<p align="center">
  <img src="docs/logo.svg" width="96" alt="Renovia logo">
</p>

<h1 align="center">Renovia</h1>

<p align="center">
  A mobile platform that brings together the owners of buildings facing urban transformation in Türkiye and the contractors who rebuild them.<br>
  Flutter · Node.js · TypeScript · PostgreSQL · Redis · Socket.IO
</p>

<p align="center">
  <a href="https://wishyasin.github.io/renovia">Website</a>
</p>

> **About this repository.** Renovia is my graduation project. The source code lives in a private repository; this repo
> documents what the product does and how it is built. I'm happy to walk through the code in an interview.

## The problem

In Türkiye's urban transformation (*kentsel dönüşüm*), the owners of an old building must agree on a contractor who will
demolish and rebuild it in exchange for a share of the new flats (*kat karşılığı*). In practice:

- **Owners are scattered.** Neighbours talk to firms one by one, and the two-thirds majority needed for a decision is hard to reach.
- **Offers are hard to compare.** Every firm presents share ratio, number of flats, delivery time and rent support differently.
- **Trust is low.** Unlicensed firms, fake listings and "deposit" fraud are common.

Renovia puts one building in one listing: the owners see every offer and every message together, compare offers in the
same format and vote. Renovia never handles money between the parties.

## Screens

<table>
  <tr>
    <td align="center"><img src="docs/screenshots/01_onboarding.png" width="200"><br><sub>List your building</sub></td>
    <td align="center"><img src="docs/screenshots/02_home.png" width="200"><br><sub>Follow the process on one screen</sub></td>
    <td align="center"><img src="docs/screenshots/03_explore.png" width="200"><br><sub>Find a contractor</sub></td>
    <td align="center"><img src="docs/screenshots/04_compare_firms.png" width="200"><br><sub>Compare firms</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="docs/screenshots/05_compare_offers.png" width="200"><br><sub>Compare offers row by row</sub></td>
    <td align="center"><img src="docs/screenshots/06_chat.png" width="200"><br><sub>One shared chat for all owners</sub></td>
    <td align="center"><img src="docs/screenshots/07_invite.png" width="200"><br><sub>Invite neighbours with a link</sub></td>
    <td align="center"><img src="docs/screenshots/08_checklist.png" width="200"><br><sub>Check the contract before signing</sub></td>
  </tr>
</table>

<sub>All people, firms and numbers in the screenshots are fictional.</sub>

## Features

**Homeowners**
- Create a building listing; the full address stays hidden until the owner replies to a contractor.
- Invite neighbours with a single link; the listing owner approves each join request.
- Receive flat-for-land offers in one format and compare them side by side, with the best value in each row highlighted.
- Vote on offers; every owner is notified when a two-thirds majority is reached.
- A shared conversation with each firm, with meeting proposals and a calendar.
- A 10-point checklist to review before signing a flat-for-land contract.

**Contractors**
- Browse verified listings by region and send structured offers.
- A public profile with reference projects, ratings and reviews from homeowners.

**Admins** (web panel served by the backend)
- Review queues for title deeds and contractor licences, with automated risk findings.
- Two-step login: a 6-digit code is emailed after the password check; only its hash is stored.

## Approach to trust

| Check | How it works |
|---|---|
| Title deed | A listing is not shown to contractors until its deed details and document are reviewed. Automated checks flag the same flat being listed twice and reused photos (a 64-bit perceptual hash, so a resized or re-saved copy still matches). Checks never decide on their own; findings go to an admin as risks. |
| Contractor | Tax number and contractor licence are reviewed; verified firms get a badge. |
| Privacy | The full address is shown only to the chosen firm; documents are visible only to the review team. Accounts can be deleted from the app, and their access tokens are revoked immediately. |

## Architecture

```mermaid
flowchart LR
  subgraph Clients
    M["Flutter app<br/>iOS · Android"]
    A["Admin panel<br/>(web)"]
  end

  subgraph Backend["Node.js · Express · TypeScript"]
    R["REST API<br/>validation · rate limits · i18n"]
    S["Socket.IO<br/>real-time chat"]
    W["Services<br/>deed checks · images · email · push"]
  end

  DB[("PostgreSQL")]
  C[("Redis<br/>cache · token revocation")]
  F["Local / object storage<br/>photos & documents"]
  N["SMTP · FCM"]

  M -- "HTTPS + JWT" --> R
  M <-- "WebSocket" --> S
  A -- "HTTPS + JWT (2FA)" --> R
  R --> W
  S --> W
  W --> DB
  W --> C
  W --> F
  W --> N
```

- **Mobile:** Flutter, about 26k lines of Dart. Turkish and English, built on a small design system
  (Familjen Grotesk and Schibsted Grotesk).
- **Backend:** Express with TypeScript, about 9.5k lines, organised as routes → controllers → services.
  The database schema is managed with 30 plain SQL migrations.
- **Auth:** short-lived JWT access tokens (15 min) with refresh tokens (7 days), bcrypt password hashes,
  token revocation in Redis when an account is deleted.
- **Hardening:** Helmet headers, rate limiting on API and auth routes, request validation, parameterised SQL.
- **Images:** uploads are re-encoded with `sharp` (thumbnails, EXIF/GPS stripped so a photo cannot leak a hidden
  address); deed and licence documents are kept in separate private storage.
- **Testing:** widget tests for the main flows (listings, offers, chat, account), and integration tests that drive the app against a
  real server and a separate demo database. The same integration tests record the product demo videos.

## Status

The app runs end-to-end against a local server: listings, deed and licence review, offers, voting, chat,
notifications and the admin panel all work. Before a public release it still needs production hosting, push
notification credentials, SMS verification and store listings.

## Author

**Yasin Özçelik** · [GitHub](https://github.com/wishyasin)
