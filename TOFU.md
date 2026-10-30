# League Page on Tofu

League Page is Nicholas Melhado's website for a [Sleeper](https://sleeper.com/) fantasy-football league, built with SvelteKit. It reads the league from Sleeper's public API and shows power rankings, matchups and playoff brackets, standings, rosters, trades and waivers, drafts, a trophy room, all-time records, manager rivalries, a league constitution and fantasy-football news. A copy starts on the league behind upstream's live demo (The Legends League); its owner points it at their own league with one setting.

## Running the app

- **Tofu framework preset:** `sveltekit` (Tofu's Vercel lane). The app has server routes, so it is not a static site.
- **Root folder:** the repository root.
- **Runtime:** Node.js 20 or newer (`engines` in `package.json`); verified with Node.js 22 (2026-10-23) and with 24 and 26 (2026-10-30).
- **Package manager:** npm, with the committed `package-lock.json`.
- **Install:** `npm ci`. The package's `prepare` script runs during the install and compiles the Material theme stylesheets `static/smui.css` and `static/smui-dark.css`, which are not committed, so the install must run lifecycle scripts (the Vercel lane's default install does). On Node.js 26 that script needs `yargs` 17.7.3 or later in the lockfile (item 5 of *Changes and reasons*).
- **Build:** `npm run build` (`vite build` with `@sveltejs/adapter-vercel`, as upstream ships it).
- **Output:** the Vercel Build Output API folder `.vercel/output` (static files and one Node.js function); SvelteKit's own build is in `.svelte-kit/output`. There is no `dist/` and no prebuilt `index.html`.
- **Preview the built app locally:** `npx vite preview --host 127.0.0.1 --port 4173`.
- **Database:** none. League data is read live from Sleeper. The optional blog keeps its posts in Contentful and is off by default.
- **Accounts and secrets:** none needed. No Tofu-owned account or key is used.

## Settings

Public build settings, read when the app is built: deploy again after changing one. Both are compiled into the pages visitors load.

| Name | Required | Purpose |
| --- | --- | --- |
| `VITE_LEAGUE_ID` | Recommended | The owner's Sleeper league ID ([how to find it](https://support.sleeper.app/en/articles/4121798-how-do-i-find-my-league-id)). Unset, a copy shows The Legends League, `1378763026437529600`. Sleeper gives a league a new ID each season (the 2026 league's `previous_league_id` is its 2025 league), so set the new ID after the league renews; the app walks back through earlier seasons by itself. |
| `VITE_LEAGUE_NAME` | With `VITE_LEAGUE_ID` | The league's name, shown on the home page, in the small-screen menu and in the Standings and Managers headings. Unset, it is `The Legends League` while `VITE_LEAGUE_ID` is unset too, and upstream's placeholder `your_league_name` otherwise. |

Optional blog (upstream feature, off by default): set `enableBlog = true` in `src/lib/utils/leagueInfo.js` and provide `VITE_CONTENTFUL_SPACE`, `VITE_CONTENTFUL_CLIENT_ACCESS_TOKEN` (Content Delivery API token) and, for comments, `VITE_CONTENTFUL_ACCESS_TOKEN` (a Content Management token, which can write to the space). The two tokens are secrets: enter them as hidden values on the app's Keys & settings page, never in the code. Upstream reads them only in server routes. Upstream's `TRAINING_WHEELS.md` ("Add a blog") describes the Contentful setup.

Everything else an owner personalises (home page text, managers, dues, dynasty or redraft, the constitution) is edited in `src/lib/utils/leagueInfo.js` and `src/routes/constitution/+page.svelte`, as upstream's `README.md` and `TRAINING_WHEELS.md` describe.

## Changes and reasons

Every adaptation is a commit on the `tofu` branch; the upstream mirror branch `master` is untouched.

1. `src/lib/utils/leagueInfo.js`: `leagueID` and `leagueName` come from `VITE_LEAGUE_ID` and `VITE_LEAGUE_NAME`, so a copy is pointed at another league from Tofu's settings instead of by editing code. Upstream's default is the placeholder `your_league_id`, which loads no league at all, so the fallback is the league behind upstream's live demo (https://www.legendsleagueff.com/, linked from the README): the ID that site's own pages load, confirmed through Sleeper's API as "The Legends League", 2026 season. A fresh copy therefore shows real, live data. Every other value in the file is upstream's.
2. `src/routes/+layout.svelte`: removed upstream's Vercel Web Analytics (`injectAnalytics` from `@vercel/analytics/sveltekit`), which would record the visitors of every copy in the hosting account's analytics. The package stays in `package.json` and `package-lock.json` so the lockfile matches upstream; nothing imports it, and the build contains none of it.
3. Removed the root `Dockerfile`. Tofu's detector reads a root Dockerfile before any framework and proposes the `container` preset, and this one (2021, `node:14-alpine`) cannot run the app's toolchain: Vite 6, SvelteKit 2 and Svelte 5 need Node.js 18 or newer, and `package.json` asks for 20. Without it Tofu detects `sveltekit` and builds with the Vercel adapter upstream already uses. `.dockerignore` and the `build-docker`, `docker-build` and `docker-run` scripts are left as upstream wrote them; the Dockerfile is still on `master`.
4. Added this file.
5. `package-lock.json` (2026-10-30): the two `yargs` entries (`node_modules/yargs` and the lockfile-v2 `dependencies` entry) move from 17.7.2 to 17.7.3 (version, tarball URL and integrity hash from the registry); nothing else in the file changes and `package.json` is untouched (`smui-theme` asks for `yargs` `^17.7.2`). `prepare` runs `smui-theme` 8.0.0-beta.3, which loads `yargs/yargs` with `require()`. In yargs 17.7.2 that is an extensionless file inside a package whose `package.json` says `"type": "module"`; Node.js 26 loads such a file as an ES module even under `require()` (Node.js 24 as CommonJS), so the install stopped with `ReferenceError: require is not defined in ES module scope`, the two theme stylesheets were never written and the build then stopped at prerender with `404 /smui.css`. yargs 17.7.3 maps `./yargs` to `yargs.cjs`; its dependencies, `engines` and `type` are the same as 17.7.2's.

Kept as upstream ships them: the League Page name, logo, favicons and app manifest; the footer's copyright line, "Built by Nicholas Melhado" credit (upstream asks that both stay) and donation link; the update notice; `LICENSE`.

## Public services the app calls

No analytics, advertising or tracking remain. A running copy, from its server routes or from visitors' browsers, contacts:

- **Sleeper API** (`api.sleeper.app`): league, users, rosters, matchups, brackets, transactions, drafts, players and NFL state, all documented and keyless, plus `api.sleeper.app/projections/nfl/...` for the weekly projections behind the power rankings, which Sleeper does not document. See the usage note below.
- **Sleeper's image CDN** (`sleepercdn.com`): avatars, player photos and NFL team logos.
- **Google Fonts** (`fonts.googleapis.com`, `fonts.gstatic.com`): Roboto, Roboto Mono and Material Icons, linked from `src/app.html`.
- **Reddit** (`www.reddit.com/r/DynastyFF/new.json`, or `r/fantasyfootball` for a redraft league): news on the Resources page.
- **News feeds**, read by the server route `/api/fetch_serverside_news`: `thefantasyfootballers.libsyn.com`, `dynastyleaguefootball.com/feed/`, `www.dynastynerds.com/feed/` and FTN (`www.ftnfantasy.com`).
- **Upstream's update check:** the server route `/api/checkVersion` fetches `https://league-page.nmelhado.com/api/checkGlobalVersion`, a plain request carrying no league or visitor data, and compares the version it returns with this copy's (`2.5.1`) to show "There is an update available" in the footer.
- **Contentful** (`cdn.contentful.com`, `api.contentful.com`), only when the owner turns the blog on.
- Links only: the league's page on `sleeper.app`, the resource and news sites, GitHub, Buy Me a Coffee, and the example LeagueSafe link on the constitution page.

## Sleeper API usage note

The Sleeper API is public and read-only and needs no account, key or token. Its documentation (https://docs.sleeper.com/) says it is free to use for non-commercial purposes (for commercial use, contact Sleeper), asks callers to stay under about 1,000 calls a minute or risk an IP block, and asks that the roughly 5 MB player list (`/v1/players/nfl`) be fetched at most once a day. Upstream fetches the player list and the weekly projections through its own route `/api/fetch_players_info`, whose answer visitors' browsers keep for 24 hours; the route itself caches nothing, so each visitor without a fresh copy makes the server fetch the list again. A first visit to the home page also makes about 200 Sleeper requests from the visitor's browser, because it walks back through the league's earlier seasons for awards and records. That is well within the limit for one visitor; a copy expecting a large audience should add server-side caching.

## Upstream and licence

- Upstream: https://github.com/nmelhado/league-page (default branch `master`)
- Base upstream commit: `ea33a685637affc5520a094d251e9b3c7c4ce6dc` (2026-10-07, League Page 2.5.1)
- Gallery fork: https://github.com/tofu-gallery/league-page, adaptations on `tofu`, pinned by the tag `pin/2026-10-23`
- Licence: MIT, "Copyright (c) 2021 [Nicholas Melhado](http://www.nmelhado.com)", in the unchanged `LICENSE`.

Caveats:

- **Real league data.** Until `VITE_LEAGUE_ID` is set, a copy shows The Legends League: real team names, Sleeper display names and avatars, as Sleeper publishes them to anyone and as upstream's demo shows them. Set your own league before sharing a copy; a copy has no connection to that league.
- **Trademarks and artwork.** NFL team logos and player photos come from Sleeper's CDN and belong to their owners. `static/` bundles upstream's own artwork (the League Page logo, awards, banners) and icons of other brands (Sleeper, Discord, WhatsApp, Reddit and the news sites in `static/newsIcons/`), which upstream uses to label contact preferences and news sources; those marks belong to their owners.
- **Sleeper's terms.** The API is free for non-commercial use; a commercial use of a copy needs Sleeper's agreement.
- **Template content.** The home page text is upstream's lorem ipsum and the constitution page is upstream's example, including an example LeagueSafe link. Owners edit both in the repository.
- **Dependencies.** The dependencies are upstream's lockfile, with one change (`yargs` 17.7.2 to 17.7.3, item 5 above). On 2026-10-23 GitHub's Dependabot reported 122 known vulnerabilities in it (2 critical, 46 high, 63 moderate, 11 low; development tools included). This fork does not update them, which keeps the lockfile in step with upstream.

## Verification

Verified locally on 2026-10-23 with Node.js 22.22.3 and npm 10.9.8, from the repository root (local checkout `/private/tmp/gallery-batch7/league-page`). Nothing was deployed.

```sh
npm ci --no-audit --no-fund
npm run build
npx vite preview --host 127.0.0.1 --port 62926 --strictPort
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:62926/   # also /standings, /matchups, /rosters, /api/checkVersion, /api/fetch_serverside_news
VITE_LEAGUE_ID=1201891419585785856 VITE_LEAGUE_NAME="Env Check League" npm run build   # setting check, previewed on another port
```

- `npm ci` installed 353 packages, and its `prepare` step wrote `static/smui.css` and `static/smui-dark.css` (Sass deprecation warnings only).
- `npm run build` passed with no warnings or errors ("Using @sveltejs/adapter-vercel ✔ done"; the function runtime is `nodejs22.x`). The built client contains no Vercel Analytics or `/_vercel/insights` reference.
- The built app answered HTTP 200 on `/`, `/standings`, `/matchups`, `/rosters`, `/api/checkVersion` (`false`, so no update notice) and `/api/fetch_serverside_news`.
- Browser check in a real browser against 127.0.0.1 only: the home page loaded live Sleeper data for The Legends League ("NFL 2026 Season - Week 7", the 2025 Fantasy Champ, Recent Waiver Moves, Rest of Season Power Rankings for all 12 teams), with 22 images from `sleepercdn.com` and none broken, 205 requests to `api.sleeper.app`, no analytics request, no console errors and no failed requests. `/standings` showed "2026 The Legends League Standings" with all 12 teams, again with no console errors or failed requests.
- With `VITE_LEAGUE_ID=1201891419585785856` (the same league's 2025 season) and `VITE_LEAGUE_NAME="Env Check League"`, `/standings` showed "2025 Env Check League Standings" and the Sleeper requests started from the configured ID, walking back through its earlier seasons; the demo league's ID was not requested.
- The preview servers were stopped afterwards.
- A fresh clone of the pushed `tofu` branch (commit `7aa9de7`) installed with `npm ci` (353 packages) and built with `npm run build` with the same results: no `Dockerfile`, both theme stylesheets generated, `.vercel/output` written, no analytics reference in the client.
- Tofu's import, simulated on the pushed branch with `python3 -I simulate_import.py --one tofu-gallery/league-page tofu '' no`, keeps 224 files and drops 2, upstream's `src/.DS_Store` and `static/.DS_Store`, which the app does not use; it finds no container file at the root.
- Size: the branch's `legacy.zip` archive was 3,160,522 bytes before this note was added, and the tree holds 226 files, 4.2 MB unpacked (the importer's limits are 50 MB, 20,000 files and 250 MB).

### Verification of the yargs change (2026-10-30)

On a Mac, with the official Node.js 26.13.0 (npm 11.20.0) and 24.22.2 (npm 11.19.1), each downloaded and checked against nodejs.org's `SHASUMS256.txt`, from a tree made by `git archive` of the commit before this note (no `.git`), with `VERCEL=1`, `CI=1` and `NOW_BUILDER=1` set. Nothing was deployed.

- Before the change, `npm install` on Node.js 26.13.0 stopped in `prepare` (`ReferenceError: require is not defined in ES module scope`, from `yargs/yargs`); the same tree installed and built on Node.js 24.22.2.
- With the change: `npm ci` installed, `prepare` wrote both stylesheets, and `npm run build` ended *Using @sveltejs/adapter-vercel ✔ done* on Node.js 26.13.0 (function runtime `nodejs26.x`) and on 24.22.2 (`nodejs24.x`).
