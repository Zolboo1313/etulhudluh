# NOTES.md — Etulhudluh.mn project documentation

These notes describe the repository as it was on 2026-11-23 (latest commit `0810b6b`, "Add listing 35"). Everything below was checked against the files in this repo. Where something can't be confirmed from the repo alone, it is marked **(not verifiable from the repo)**.

---

## 1. Project overview

- **What it is:** Etulhudluh.mn is a real estate listings website for Zolboo, a real estate agent in **Erdenet**, Mongolia. It shows apartments, houses, land, garages and commercial space for sale or rent, with photos, prices and contact buttons (call, Viber, Facebook). All visible text is in Mongolian.
- **How it works:** it is a single static page (`index.html`). There is no backend, no database, no build tool and no framework. All listings are written directly inside `index.html` as a JavaScript array, and plain JavaScript in the same file draws the page in the browser.
- **Hosting:** GitHub Pages, served straight from the repository (the `.nojekyll` file at the root tells Pages to serve the files as they are). The publishing source is believed to be the `main` branch, root folder **(not verifiable from the repo: it is a GitHub setting, see section 6)**.
- **Live URL:** https://zolboo1313.github.io/etulhudluh/
- **Domain:** the name "Etulhudluh.mn" is only the site's brand name at the moment. There is **no `CNAME` file**, so no custom domain is connected; the site lives on the `github.io` address above.
- **Agent contact (as stored in the site):** phone 8836 0522, Viber 97688360522, Facebook https://www.facebook.com/AgentZolboo.

---

## 2. File structure

```
/
├── index.html     The whole website: HTML, CSS, listing data and JavaScript (~670 lines)
├── README.md      Short Mongolian description + live URL
├── .nojekyll      Empty file. Tells GitHub Pages not to run Jekyll. Keep it.
├── NOTES.md       This file
├── zurag/         Full-size listing photos ("zurag" = "photo"), one folder per listing
│   ├── central/1.jpg … 9.jpg
│   ├── denj-haus/1.jpg … 11.jpg
│   └── … 34 folders in total
└── zurag-s/       Small copies of the same photos ("-s" = small), same folder and file names
    ├── central/1.jpg … 9.jpg
    └── … 34 folders in total
```

Details:

- **`index.html`** is laid out in this order:
  1. `<head>` (lines ~1–11): charset, viewport, meta description, a tiny safe-area style, `window.SMALL_PHOTOS=true`, and the GoatCounter visitor counter script.
  2. Top of `<body>`: `<title>`, Google Fonts link (loaded non-blocking), then the main `<style>` block with all CSS.
  3. Page skeleton: header (logo + call button), `<main id="app">` (empty; filled by JavaScript), footer.
  4. `<script>` with, in order:
     - **Section 1, `AGENT`**: name, title, phone, Viber, Facebook, email.
     - **Section 2, `LISTINGS`**: the array of all listings (with a Mongolian comment explaining the fields).
     - Helper code: filters, price formatting, placeholder photos, card and detail rendering, router.
- **`zurag/<slug>/N.jpg`**: full-size photos, 1400 px wide (checked: e.g. `zurag/central/1.jpg` is 1400×1050). Used for the big photo on a listing's detail page.
- **`zurag-s/<slug>/N.jpg`**: 560 px wide copies (e.g. 560×420). Used for listing cards and detail-page thumbnails so the first load is fast. Every photo in `zurag/` that a listing uses has a matching file in `zurag-s/` (checked: none missing).
- Total size: `zurag/` ≈ 36 MB, `zurag-s/` ≈ 6.5 MB.
- There is **no** `package.json`, no `.github/workflows/`, no `CNAME`, no `404.html`, and no separate CSS/JS/JSON files.

---

## 3. How listings are stored and rendered

### Where the data lives
All listing data is in **`index.html`**, inside `<script>`, in the array `const LISTINGS = [ … ];` (starts around line 210). Each listing is one JavaScript object. The contact details used everywhere are in `const AGENT = { … }` just above it.

On 2026-11-23 there are **34 listings**, ids 35, 33, 32, …, 1 (id 34 was deleted). They are written **newest first** (highest id at the top).

### How it is rendered (all client-side JavaScript in `index.html`)
- `route()` reads the URL hash. `#zar-N` (e.g. `#zar-12`) shows the detail page of listing id N; anything else shows the list page. It runs on load and on every `hashchange`.
- `renderList()` draws the intro text, filters (deal type, property type, location, price range, number of rooms), the sort menu, and the grid. `drawGrid()` and `filtered()` apply the filters; `card(l)` builds each listing card.
- `renderDetail(l)` draws one listing: photo gallery with arrows, thumbnails and swipe, description, optional units table, features list, facts table, price box and contact box.
- Filter options are generated **automatically** from the data: `TYPES` = every distinct `type`, `DISTRICTS` = every distinct `district`. Price ranges are fixed in `SALE_PRICES` and `RENT_PRICES`.
- Default sort is "newest" = **by `id`, highest first** (the order in the array does not matter for display, but keep it newest-first for readability).
- **Price display** (`priceLabel` + detail price box):
  - `price_m2` set → shows "X сая₮ / м²" and "1м² нь … · нийт үнийг утсаар лавлана уу" (no total is calculated or shown).
  - else `price` set → shows the total price.
  - both empty (`price: null`, no `price_m2`) → shows "Үнэ тохиролцоно".
  - `price_note` (optional) is shown as an extra line under the price.
  - Rentals (`deal: "Түрээслэх"`) get " / сар" (per month) appended.
- **Photos:** the card and thumbnails use `sm(path)`, which swaps `zurag/` → `zurag-s/` **only because `window.SMALL_PHOTOS=true` is set in `<head>`**. The big gallery photo always uses the full-size `zurag/` file. If a listing has no photos, generated SVG placeholder drawings are shown instead.
- Every card shows `#id`, and the detail page tells callers to say "зарын дугаар #id", so Zolboo can find the listing a caller means.
- **Visitor counting:** GoatCounter (`https://etulhudluh.goatcounter.com`). Page loads are counted by the script in `<head>`; opening a listing is counted as the path `/zar/N` by the small `hashchange` script next to it.

---

## 4. HOW TO ADD A NEW LISTING

### Step-by-step
1. **Pick the next id.** Use the highest id ever used + 1. The highest id so far is **35**, and **34 was deleted, so never reuse 34**. The next new listing is **36**.
2. **Pick a photo folder name (slug).** Short, lowercase Latin, hyphens, no spaces, unique. Existing examples: `6mikr-13`, `5mikr-12`, `denj-haus`, `grand-garden-7`, `6mikr-14-turees` (rental). Pattern used for old apartment blocks: `<микр number>mikr-<building number>`.
3. **Prepare photos.**
   - Name them `1.jpg`, `2.jpg`, `3.jpg`, … in display order. **`1.jpg` is the cover photo on the card**, so put the best interior photo first.
   - Full size: resize to **1400 px wide**, JPEG, save into `zurag/<slug>/`.
   - Small copy: resize the same photo to **560 px wide**, same file name, save into `zurag-s/<slug>/`.
   - Both folders must contain **exactly the same file names**. If a file is missing from `zurag-s/`, the card/thumbnail image will be broken on the live site.
   - Don't publish photos with another agency's logo, contracts, or commission details.
4. **Add the listing object** at the **top** of the `LISTINGS` array in `index.html` (right after `const LISTINGS = [`), copying the format below. Every entry ends with `},` (comma) except the very last one in the array (id 1).
5. **Check it locally** (see section 7), then commit both photo folders and `index.html` together, and push to `main`.

### Fields

| Field | Required? | Meaning / allowed values |
|---|---|---|
| `id` | **required** | Unique whole number. Never reuse a deleted id. Used in the URL `#zar-<id>` and for sorting. |
| `deal` | **required** | Exactly `"Худалдах"` (sale) or `"Түрээслэх"` (rent). The deal filter matches these exact strings. |
| `type` | **required** | Property type. Values in use: `"Орон сууц"`, `"Хаус"`, `"Газар"`, `"Барилга, талбай"`, `"Гараж, зогсоол"`. Reuse an existing spelling exactly, or a new filter option appears. `"Газар"` makes size show as "м² газар". |
| `district` | **required** | City. All current listings use `"Эрдэнэт"`. Becomes the "Байршил" filter. |
| `area_name` | **required** | Neighbourhood/building, e.g. `"6-р микр, 6-р байр"`, `"Сувд хотхон"`. |
| `title` | **required** | Headline shown on the card and detail page. |
| `price` | **required key** (may be `null`) | Total price as a plain number in ₮, e.g. `130000000`. For rentals: monthly rent. Use `null` if no total is known. |
| `price_m2` | optional | Price per m² as a number, e.g. `4000000`. When set, the site shows the per-m² price instead of a total. Use with `price: null`. |
| `price_note` | optional | Extra price line, e.g. `"Үнэ тохиролцоно"`, `"Бартер оролцуулж болно"`. |
| `rooms` | **required key** | Number of rooms. `0` = unknown/not applicable (hidden). `4` or more counts as "4+" in the filter. |
| `size` | **required key** (may be `null`) | Area in m² as a number, e.g. `41.31`. `null` = unknown (hidden). |
| `floor` | **required key** | Text, e.g. `"3-р давхар"` or `"11/12"`. `""` or `"-"` = hidden. If the text doesn't contain "давхар", the card adds " давхар". |
| `year` | **required key** (may be `null`) | Year put into use, e.g. `2023`, or `null`. |
| `payment` | **required key** (may be `""`) | Payment terms text, e.g. `"Бэлэн, бартер"`. |
| `features` | **required key** | Array of short strings (bullet list). Can be `[]`. |
| `description` | **required** | Full text. Use `\n` for a line break and `\n\n` for a new paragraph (line breaks are kept on the page). |
| `photos` | **required key** | Array of paths starting with `zurag/`, e.g. `["zurag/abc/1.jpg", "zurag/abc/2.jpg"]`. Shortcut used in the file: `[1,2,3].map(n => "zurag/abc/" + n + ".jpg")`. Never write `zurag-s/` here. |
| `units` | optional | For buildings with several flats for sale: array of groups `{ name, rows: [ { no, floor, size, kind } ] }`. `size` must be a number in every row. Shows a "Зарагдах айлууд" table, and the card shows the count and m² range. |
| `units_head` | optional | Two column titles for the units table (1st and 4th columns), e.g. `["Өрөө", "1м² үнэ"]`. Default is `["Тоот", "Төрөл"]`. |
| `sample` | optional | `true` marks a demo listing ("Жишээ"). Not used by any real listing; don't set it. |

"Required key" means the code reads it, so keep the key present even when the value is empty (all 34 current listings include every one of these keys).

### Full example entry (copy this)

```js
  { id: 36, deal: "Худалдах", type: "Орон сууц", district: "Эрдэнэт", area_name: "4-р микр, 9-р байр",
    title: "4-р микрийн 9-р байранд 2 өрөө байр, 3-р давхар", price: 150000000, rooms: 2, size: 44.5, floor: "3-р давхар", year: null,
    price_note: "Үнэ тохиролцоно",
    payment: "", features: ["Цонх урагшаа", "Засвартай", "Тагттай"],
    description: "4-р микрийн 9-р байрны 3-р давхарт 2 өрөө байр зарна. Засвартай, цонх урагшаа.\n\nҮнэ: 150 сая₮, тохиролцоно.",
    photos: [1,2,3,4].map(n => "zurag/4mikr-9-b/" + n + ".jpg") },
```

(This is an illustration only: the address, price and folder are made up.) For a **per-m² price** instead of a total, write `price: null, price_m2: 4000000,` and do **not** put a computed total anywhere (see section 8).

---

## 5. HOW TO EDIT OR REMOVE A LISTING

### Edit
1. In `index.html`, search for `id: <number>,` (e.g. `id: 28,`) to find the listing.
2. Change only the values you need. Keep quotes, commas and brackets intact.
3. **Price cut:** change `price` (or `price_m2`) and, if you want the cut visible, say so in `price_note`, e.g. `"Үнэ буурсан: 125 сая₮ → 120 сая₮"` (this is how listing 30 does it).
4. **Photos:** to add photos, put the new files in **both** `zurag/<slug>/` and `zurag-s/<slug>/` and add their paths to `photos`. To change the cover, change which file is first in `photos` (or rename files). To remove a photo, remove its path from `photos`; you can also delete the file from both folders.
5. Do not change the `id`: people may have saved or shared the link `#zar-<id>`, and GoatCounter stats are counted per id.

### Remove (sold / withdrawn)
1. Delete the whole object for that listing, from `{ id: N, …` through its closing `},`.
2. If you removed the **last** item in the array (currently id 1), make sure the new last item doesn't leave a broken comma/bracket. (A trailing comma before `]` is allowed in JavaScript, so this is not fatal.)
3. Optionally delete its folders in both `zurag/<slug>/` and `zurag-s/<slug>/` (check that no other listing uses the same folder first; e.g. search `index.html` for the slug).
4. **Never reuse that id** for a new listing. Example from history: commit `5268c9a` removed listing 34, and the next listing became 35.
5. Old links `#zar-<removed id>` will simply show the list page (no error).

There is no "sold" flag in the code. Marking a listing as sold without deleting it would require a code change.

---

## 6. Settings and config that must never be broken

- **Single file + relative paths.** All photo paths are relative (`zurag/…`, `zurag-s/…`). The site lives under the sub-path `/etulhudluh/`, so **never start a path with `/`** (e.g. `/zurag/a.jpg` would point to `zolboo1313.github.io/zurag/…` and break). Keep `index.html`, `zurag/` and `zurag-s/` together at the repo root.
- **Folder names `zurag` and `zurag-s`** are hard-coded in the `sm()` function (it swaps `zurag/` for `zurag-s/`). Renaming either folder breaks photos.
- **`window.SMALL_PHOTOS=true;`** in `<head>`: turns on the small photos for cards. If it is removed, the site still works but loads the 1400 px photos everywhere (slow on phones). If it stays on, `zurag-s/` must have a copy of every photo.
- **`.nojekyll`**: keep it. Without it GitHub Pages runs Jekyll on the repo.
- **GitHub Pages settings** (in GitHub → repo Settings → Pages, not in the code): source should be the `main` branch, `/ (root)` folder **(not verifiable from the repo)**. Pushing to another branch does not change the live site. The repo must stay **public** for free Pages.
- **No custom domain / no `CNAME`.** If a CNAME file is added later (for a real Etulhudluh.mn domain), the domain's DNS must be set up too, or the site will stop loading. Don't add or delete a `CNAME` casually.
- **Listing ids and `#zar-<id>` URLs** are used for links, routing and GoatCounter statistics (`/zar/<id>`). Ids must be unique whole numbers.
- **Exact strings used in code logic:** `deal` must be exactly `"Худалдах"` or `"Түрээслэх"`; `type: "Газар"` changes the size label; `floor` values `""`/`"-"` are hidden. Typos create broken filters.
- **`AGENT.phone` and `AGENT.viber`** build the call/Viber links (`tel:` removes spaces; Viber uses `+` + the number). Keep Viber as country code + number, digits only (`97688360522`).
- **GoatCounter script** (`data-goatcounter="https://etulhudluh.goatcounter.com/count"`): changing or removing it stops visitor statistics.
- **Fonts line**: the Google Fonts `<link>` uses `media="print" onload="this.media='all'"` so it doesn't block the page. Don't turn it back into a normal blocking stylesheet: previously the Facebook in-app browser showed a blank page while it waited for the font (commit `8952c8b`).
- **JavaScript syntax**: because all data is JavaScript, **one missing comma, quote or bracket in `LISTINGS` makes the whole site blank** (no listings at all). Always check after editing (section 7).
- **No build or deploy pipeline** exists in the repo (no workflows, no package.json). Do not expect one to run.

---

## 7. How to deploy and how to verify

### Deploy
1. Commit the changed files (`index.html` and any new `zurag/<slug>/` and `zurag-s/<slug>/` photos) to the **`main`** branch.
2. Push to GitHub (`git push origin main`), or upload/edit the files on github.com directly on `main`.
3. GitHub Pages republishes automatically, usually within 1–2 minutes. Progress is visible in the repo's **Actions** tab ("pages build and deployment") **(this tab is provided by GitHub, not by files in the repo)**.

### Verify before pushing (local check)
- Open `index.html` directly in a browser (double-click it), or run a local server in the repo folder, e.g. `python3 -m http.server 8000` and open http://localhost:8000/.
- Check: the list shows, the counter "N зар олдлоо" went up/down as expected, the new card has its cover photo, clicking it opens `#zar-<id>` with all photos and thumbnails.
- If the page is empty, open the browser console (F12 → Console): a red "SyntaxError" means a comma/quote/bracket mistake in `LISTINGS`.
- Quick data check from a terminal (optional, needs Node.js): this prints the number of listings and any photo files missing from `zurag/` or `zurag-s/`:
  ```bash
  node -e 'const fs=require("fs"),h=fs.readFileSync("index.html","utf8"),a=h.indexOf("const LISTINGS");
  const L=new Function(h.slice(a,h.indexOf("/* =====",a)).replace("const LISTINGS =","return "))();
  const ids=L.map(l=>l.id);console.log(L.length,"listings; duplicate ids:",ids.filter((x,i)=>ids.indexOf(x)!==i));
  L.forEach(l=>l.photos.forEach(p=>[p,p.replace(/^zurag\//,"zurag-s/")].forEach(f=>fs.existsSync(f)||console.log("MISSING",l.id,f))))'
  ```

### Verify after deploy (live site)
- Open https://zolboo1313.github.io/etulhudluh/ (on a phone too; hard-refresh or add `?v=2` to the URL if you see an old version because of caching).
- Open the new/edited listing directly: https://zolboo1313.github.io/etulhudluh/#zar-<id>
- Check photos load on the card (small photo) **and** in the gallery (big photo).
- Optionally open it inside the Facebook app, since many visitors come from Facebook.
- GoatCounter (https://etulhudluh.goatcounter.com) should show a visit after you open the page.

---

## 8. Known issues and things to be careful about

**Business rules (from Zolboo's instructions):**
- **Never calculate a total price as m² × per-m² price.** In old buildings the kitchen area is not included in the m² figure, so the multiplied total would be wrong. If Zolboo gives a per-m² price, use `price: null, price_m2: N` and show only the per-m² price; only show a total price when Zolboo gives one. (Note: the code's `sortPrice()` multiplies `price_m2 × size` internally, but **only** for sorting and the price-range filter; it is never shown to visitors. Because of this, per-m² listings may land in a slightly "wrong" price filter range.)
- **Hidden floor prices ("доод үнэ").** The lowest price a seller will accept is kept private and must not be written on the site, not even in `description` or `price_note` (everything in the file is public; anyone can read `index.html`). Show the asking price, or "Үнэ тохиролцоно" (negotiable). If the floor price is the *only* price Zolboo gives, it is shown as the price.
- **Wording:** write **"микр"** (e.g. "6-р микр"), never "бичил хороолол". This is how people in Erdenet say it.
- **Privacy:** the repo is public, so every line of `index.html` (including comments) can be read by anyone. Don't put private notes, seller names/phones, floor prices, contract pages or commission percentages in the file or in photos.
- Photos with another agency's logo should be replaced with Zolboo's own photos when possible.

**Technical issues seen in the repo:**
- The `<title>`, Google Fonts `<link>`, an extra `<meta name="claude:viewport-fit-sides">` and the main `<style>` block are inside `<body>` instead of `<head>`. Browsers handle this fine, but some link-preview tools (e.g. Facebook link sharing) may not pick up the title; there is also no `og:image` / Open Graph tags, so shared links have no preview image.
- The `<meta name="claude:viewport-fit-sides">` tag is leftover from the claude.ai artifact version of the page; it has no effect on GitHub Pages.
- **Two copies of the site exist** **(not verifiable from the repo; from project history):** the page was originally built as a claude.ai artifact, and the repo `index.html` is a "build" of it (with added `<!doctype>`, `<head>`, `SMALL_PHOTOS`, GoatCounter, non-blocking fonts, and the generated `zurag-s/` copies). If someone edits the artifact copy and re-builds, edits made directly in the repo can be overwritten, and vice versa. Pick one source of truth and keep them in sync.
- `zurag-s/` copies are made by hand/script; there is no automatic step that creates them. Forgetting them = broken card images.
- All listings use `district: "Эрдэнэт"`, so the "Байршил" (location) filter currently has only one option. The neighbourhood is in `area_name`, which is not filterable.
- `rooms: 0` is used for "unknown" and for multi-unit buildings, so those listings never match the 1/2/3/4+ room filters.
- Sorting by "Том талбайтай" (largest) puts listings with `size: null` at the bottom.
- There is no "sold" status; sold listings must be deleted (section 5).
- Some listing details are still unconfirmed (e.g. m² unknown for several listings, one exterior-only photo for #35); check with Zolboo before "fixing" them.
- The repo is ~43 MB of photos. GitHub Pages has a 1 GB site limit, so there is room, but always resize photos (1400 px / 560 px) before adding them.
- Call (`tel:`) and Viber links work on the GitHub Pages site but did not work reliably inside the claude.ai artifact viewer; that's why the phone number is also shown with a "Хуулах" (copy) button.
