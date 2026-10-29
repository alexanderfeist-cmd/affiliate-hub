# Affiliate Hub

Single HTML file. Data lives in three SharePoint lists, users sign in with their Microsoft account.
Open `index.html` locally to try demo mode (sample data, nothing saved).

## 1. Create the lists on your SharePoint site

Create every column **without spaces** first (the internal name is fixed on creation). You can rename the display name afterwards.

**Partners**: Title, Country (Choice: UK, ZA, DE, FR, ES, PT), Tier (Choice: Under Review, Regular Partner, Premium Partner, Strategic Partner),
Status (Choice: Onboarding, On track, At risk, Blocked, Paused), InternalLead (Text), PartnerOwner (Text),
PrimaryContact (Text), ImpactLink (Text), Notes (Multiple lines, plain text),
ImpactPartnerId (Text; turn on Enforce unique values), ActivationDate (Date), ImpactStatus (Text), PartnerType (Text), Website (Text),
DirectoryOnly (Yes/No, default No; Yes = listed in All partners only, not managed as a key partner)

**Updates**: Title (short preview, filled by the app), Partner (Lookup to Partners > Title),
StatusBefore (Text), StatusAfter (Text), Body (Multiple lines, plain text; full update text with links),
UpdateFiles (Multiple lines, plain text; file names and links, filled by the app. Not "Attachments": that name is built into every list)

**Tasks**: Title, Partner (Lookup to Partners > Title), DueDate (Date), AssignedTo (Text), Done (Yes/No, default No),
Market (Text; filled by the app for weekly checklist items), CheckKey (Text; in column settings > More options set **Enforce unique values: Yes**. This stops two people creating the same monthly check at once.)

**Links** (powers the Reporting and Market updates tabs; optional, the app runs without it): Title,
LinkUrl (Text), LinkType (Choice: Power BI, Excel, Loop, Other), Country (Choice: EMEA, UK, ZA, DE, FR, ES, PT),
Section (Choice: Reporting, Loop updates; "Loop updates" entries show in the Market updates tab), Summary (Multiple lines, plain text), UpdateDate (Date)

## 2. Host the file

GitHub: new public repo, upload `index.html`, Settings > Pages > deploy from main branch.
Note the URL, e.g. `https://yourname.github.io/partner-hub/`. No data or secrets are in the file.

## 3. App registration (Entra)

- App registrations > your app > Authentication > Add a platform > **Single-page application**
- Redirect URI: the exact GitHub Pages URL from step 2 (with trailing slash)
- API permissions: Microsoft Graph, Delegated: `User.Read`, `Sites.ReadWrite.All`
- For task emails also add Delegated: `Mail.Send`, `User.ReadBasic.All` (asked for once, the first time an email is sent)

## 4. Fill in CONFIG at the top of the script

`clientId`, `tenantId`, `siteHost` (e.g. `contoso.sharepoint.com`), `sitePath` (e.g. `/sites/EMEAAffiliates`).
Commit, wait a minute for Pages to update, open the URL, sign in.

## 5. Put it in SharePoint

Best: a button or Quick links web part on your SharePoint page pointing to the URL, and a Website tab in Teams.
Embed web part works only if the domain is allowed (Site settings > HTML Field Security), and Microsoft sign in inside an iframe can be blocked by some browsers.

## Notes

- All partners tab: every partner in the Partners list by market. Refresh it with **Import impact.com CSV**: export partners from impact.com, choose the file, check the column mapping, import. Rows are matched on Partner ID (or on name for existing partners without an ID). impact.com fields are overwritten on each import; tier, status, lead and notes are never touched. New partners arrive as directory only; tick Key partner in their details to manage them in the hub. Partners missing from a later export are not deleted.

- EMEA wide section (top of Market updates): programme actions from `CONFIG.emea.items`, created every Sunday for the coming week (or monthly if `cadence: 'monthly'`) and assigned to `CONFIG.emea.owner`. Set the owner to your name or email so the tasks and emails reach you. EMEA updates are posted with Country = EMEA in the Links list.

- Task emails: when a task is created, the owner gets an email from the mailbox of whoever created it (auto-created tasks are grouped into one email per owner). The owner is matched by display name in the Microsoft directory, or you can type an email address in Assigned to. People are not emailed about tasks they assign to themselves. Turn off with `CONFIG.email.enabled = false`.

- Market updates tab: shows each market's weekly tasks next to its posted updates. On Sunday the app creates the coming week's items for every market (common items plus any market specific ones in `CONFIG.weekly`), due on the second working day. Public holidays listed in `CONFIG.weekly.holidays` are skipped. If nobody opens the app on Sunday, the items are created on first open that week. Extra one-off tasks and updates can be added per market from the same tab.

- Attachments upload to the site's Documents library under `Affiliate Hub Attachments/<country>/<partner>`. Folders are created automatically. Max size per file is set by `maxFileMB` in CONFIG (default 100 MB). Anyone with access to the site can open them.
- Links in updates: paste a web address, or use Add link to insert `[text](https://...)`.

- Monthly checks: Premium Partner = 1, Strategic Partner = 2 per calendar month. They are created when anyone opens the app (or saves a partner), so there is no scheduler to run. Change counts or assignee in CONFIG (`checksPerMonth`, `checkAssignee`).

- Status history is logged by the app. Changes made directly in the Lists UI are not logged in the Health tab.
- Anyone using the app needs at least Edit access to the site.
