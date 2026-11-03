PROMPTVERSE — STATIC FRONTEND + GOOGLE SHEETS CMS
==================================================

PromptVerse is an original beginner-friendly prompt gallery built with HTML, CSS and vanilla JavaScript. Google Apps Script is the backend, Google Sheets is the database, and Google Drive stores uploaded images. It does not use PHP, Node.js, NPM, Composer, a frontend framework or a third-party database.

PROJECT MAP
-----------
Public pages: index.html, prompts.html, search.html, prompt.html, category.html, collection.html, templates.html, page.html, contact.html, 404.html
Public styles/scripts: assets/css/, assets/js/, assets/images/
Reusable component examples: components/
Admin landing page: admin/index.html
Apps Script source files: google-apps-script/
Clean route rules: _redirects (Netlify-compatible) and .htaccess (Apache)
SEO files: robots.txt and sitemap.xml
Sample record themes: data/sample-data.json

The public pages call one small collection of functions in assets/js/main.js. The UI uses shared header, footer, card, preview, copy and pagination functions. Public data comes only from published Sheets records. No demo records are hard-coded in the browser; the setup function inserts editable sample rows into a new spreadsheet.

URL ROUTES
----------
/                         Home
/prompts/                 Searchable, filterable prompt gallery
/prompt/{slug}/           Individual prompt
/category/{slug}/         Category prompt list; /category/ is the directory
/collection/{slug}/       Collection prompt list; /collection/ is the directory
/templates/               Template prompts
/search/?q=...            Search results
/about/, /privacy-policy/, /terms-and-conditions/, /disclaimer/, /dmca/
/contact/                 Contact form
/admin/                   Static link page to the Apps Script hosted admin

SEO AND STATIC HOSTING LIMITS
-----------------------------
Clean paths are served by the host rewrites. Detail pages fetch their metadata and prompt content in the browser after the initial HTML has loaded. The initial prompt.html/category.html/collection.html files are generic shells. Search engines that run JavaScript may index them, but this is not equivalent to server-rendered or prerendered pages; per-prompt search snippets, social cards and indexing are not guaranteed. This project does not claim full SEO rendering for dynamic records.

The checked-in sitemap.xml is a starter file and includes only stable directory/static routes. After publishing content, run buildSitemapXml_ in Apps Script, copy the logged XML into the hosted sitemap.xml, and redeploy it. It excludes unpublished records and admin pages. A sitemap that updates on every Sheets edit requires a host-side build hook or a serverless rewrite that is outside this static-only project.

Update every https://example.com placeholder in config.js, canonical metadata, robots.txt, sitemap.xml, and the Settings sheet after choosing your real domain. The public-site title and description in HTML are useful defaults; dynamic detail metadata is updated client-side.

GOOGLE SHEETS AND APPS SCRIPT SETUP
----------------------------------
1. Create a new, dedicated Google Spreadsheet. Do not point this initializer at a spreadsheet with data you need: it creates and clears tabs with the PromptVerse sheet names on its first run.
2. In that spreadsheet choose Extensions → Apps Script.
3. Create Apps Script files matching the names in google-apps-script/: Code.gs, Database.gs, PublicApi.gs, Authentication.gs, AdminApi.gs and Media.gs. Paste each matching file. Add an HTML file named Admin and paste Admin.html.
4. Save the Apps Script project. Select initializePromptVerse_ in the function picker and run it once. Approve the requested Sheets and Drive permissions. It creates all ten tabs and headers, creates a PromptVerse Media Drive folder, and inserts original sample categories, collections, ten prompts, three pages and initial settings.
5. Open the Apps Script execution log for that run and copy the one-time first-admin setup token. Keep it private. The server stores the token and consumes it after one successful setup.
6. Deploy → New deployment → Web app. Set “Execute as” to your account (the spreadsheet owner). Set access to “Anyone” so the public read API can serve visitors. Workspace administrators may disable public web apps; use an account and host policy that permits this. Keep the spreadsheet itself private.
7. Copy the deployed URL ending in /exec. The public API uses JSONP for read-only browser requests because Apps Script does not let this project set arbitrary CORS response headers.
8. Open the static site’s /admin/ landing page after deployment. For first-time setup, open the Apps Script admin URL directly by appending ?page=admin to the /exec URL, show “First-time admin setup,” paste the one-time token, and create the admin email/password. Use a unique password with 12 or more characters.
9. In assets/js/config.js set apiUrl to the deployed /exec URL and adminUrl to the same URL with ?page=admin. Example values are described in that file. The admin dashboard uses google.script.run within Apps Script HTML Service; do not move Admin.html to static hosting or expose the spreadsheet ID/password to public JavaScript.
10. Deploy the project root to an HTTPS static host. Netlify reads _redirects. Apache hosting can use .htaccess with mod_rewrite enabled. Set the public-site domain and SEO placeholders before submitting a sitemap.
11. Open the hosted home page, explore the seeded records, then sign in at /admin/ and edit one sample prompt. Deploy Apps Script changes as a new web-app version after editing backend files.

The admin dashboard itself is served by Apps Script at the Google deployment URL. /admin/ on the static site is a launch page because a static host cannot map its own /admin/ path to the Apps Script domain without a reverse proxy.

API AND BROWSER SECURITY NOTES
------------------------------
Public reads: getHomepageData, getPrompts, getPromptBySlug, getCategoryBySlug, getCollectionBySlug, getTemplates, searchPrompts, getTrendingPrompts, getFeaturedCollections, getCategories, getCollections, getRelatedPrompts, getPageBySlug and getPublicSettings. The API never returns drafts or the whole workbook. JSONP is used only for these public read calls and counters.

Admin functions: adminLogin/adminLogout, changeAdminPassword, and the token-checked adminCall dispatcher. Dispatcher actions cover dashboard stats, list/save/delete for prompts/categories/collections/tags/pages/settings, newsletter and message lists/deletion, CSV export, media listing/upload/deletion. Every privileged dispatcher request validates the time-limited server-side session token; the UI does not supply an “is admin” flag. Passwords are salted PBKDF2-HMAC-SHA256 hashes (100,000 rounds), not plaintext. Login attempts are limited per email for 15 minutes, and sessions expire after six hours. Apps Script cache eviction can expire a valid session sooner. Use HTTPS and a strong unique admin password.

The first-admin token is one-time and server-checked. The setup and sitemap-builder functions end with an underscore so the browser client cannot call them. Never put the one-time token or password in config.js, a public sheet, or a source repository.

Newsletter/contact submit with a simple form POST so email addresses and message text do not end up in a public URL. Apps Script does not return CORS headers, so the browser uses no-cors and cannot read the server’s response. The UI reports that the request was sent, not that a sheet write was confirmed. Check the Newsletter/Messages tabs during setup. Contact forms have field limits, a honeypot and a per-email throttle. Newsletter duplicate addresses are prevented under a script lock. No email is sent; the records are only saved for an administrator.

Drive uploads accept JPG/JPEG, PNG, WEBP and GIF up to 3 MB. Files are placed in the script-owned PromptVerse Media folder and get “Anyone with the link can view” sharing so public previews can load. Google Workspace policy may prohibit that setting. Delete checks that the image belongs to the PromptVerse folder and is not still referenced. Use only images you have rights to publish.

Public counters and the public GET API can be called outside this website. They contain only published prompt data, but views/copies can be manipulated. For a high-traffic production service, add abuse controls or move the API behind a same-origin serverless proxy. Do not share the spreadsheet publicly.

FEATURES INCLUDED AND LIMITS
----------------------------
Included: responsive original gallery UI; homepage sections; prompt/category/collection/template/search listings; server-side filtering and pagination; quick preview; copy with clipboard fallback; prompt view/copy counters; static-page text; contact/newsletter storage; password login/logout; prompt/category/collection/tag/page/settings CRUD; admin statistics; CSV export; Drive uploads, list and guarded deletion; status fields; SEO metadata fields; JSON-LD WebSite markup; clean-route examples; sitemap generator; sample data.

Advertisement slots are empty by default and show no fake ads. When an administrator sets ads_enabled=true and saves slot snippets, the public site inserts the configured code at the header, home, gallery, category, prompt-detail, sidebar and footer placements. Only paste code from your approved AdSense account: ad slot content is executable site code and is returned publicly only while ads are enabled. Google AdSense approval and real ad delivery depend on your account and domain. Analytics IDs and Search Console values are stored as settings; add their site snippets or verification after configuring your domain. Social URLs can be stored in settings; the demo header intentionally has no invented social accounts.

The category assignment field in the prompt editor stores a category ID. The seeded category IDs are stable labels such as cat-1 through cat-5; for new categories, use the ID column shown in the Categories sheet. Collection IDs and tag names are also stored as pipe-separated text. A beginner can copy the ID from Sheets. Deleting a category or collection still referenced by a prompt is blocked to prevent broken records.

TEST AND DEPLOY CHECKLIST
-------------------------
Before deployment, review these checks after setting up your own spreadsheet and web-app URL:
[ ] Apps Script creates all ten expected tabs and seed records.
[ ] Unauthenticated public calls return only published records.
[ ] Search, category, collection, template and page routes return expected content.
[ ] A visitor can copy prompt text; the counter increases in Sheets.
[ ] The Admin login rejects wrong credentials and accepts the first admin.
[ ] A prompt can be created, edited, published, drafted and deleted.
[ ] Category/collection deletion is blocked when references exist.
[ ] Newsletter duplicates do not create duplicate rows; contact submissions appear in Messages.
[ ] Newsletter CSV export opens correctly.
[ ] Upload a small supported image, verify public display, and verify referenced deletion is blocked.
[ ] Check routes and responsive layout on a phone-sized viewport.
[ ] Replace example.com, refresh the sitemap after publishing, and inspect page metadata.

I could not execute Apps Script or connect to Google Sheets/Drive from this workspace because no Google credentials or deployment are available. No live integration, deployment, email delivery, AdSense approval, or Google indexing is claimed. The project can be verified end-to-end after following setup with your Google account.
