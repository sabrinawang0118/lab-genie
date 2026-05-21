# Exact Sciences AI/BI Workshop — Facilitator Talk Track
Audience: workshop facilitator (you)
Companion to: the attendee workbook in this folder
How to use: Part 1 is a quick reference you can glance at during delivery. Part 2 is verbatim script you can read aloud or rehearse from.
———
# Part 1: Tight Reference

## Permissions setup
Run these once before the workshop. They cover all participants via the built-in account users group.
-- Marketplace source data (Modules 0 & 1)
GRANT USE CATALOG ON CATALOG trilliant_health_trilliant_health_s_provider_directory TO `account users`;
GRANT USE SCHEMA ON SCHEMA trilliant_health_trilliant_health_s_provider_directory.directory TO `account users`;
GRANT SELECT ON TABLE trilliant_health_trilliant_health_s_provider_directory.directory.directory_providers TO `account users`;
GRANT SELECT ON TABLE trilliant_health_trilliant_health_s_provider_directory.directory.directory_organizations TO `account users`;

-- Create gold table
CREATE SCHEMA IF NOT EXISTS dbx_training.workshop_shared;

CREATE OR REPLACE TABLE
dbx_training.workshop_shared.directory_providers_gold AS
SELECT
  provider_npi,
  provider_primary_specialty_description,
  provider_affiliated_practice_1_state,
  primary_organization_name,
  active_provider,
  provider_specialty_classification,
  provider_affiliated_practice_1_county,
  provider_practices_total
FROM trilliant_health_trilliant_health_s_provider_directory.directory.directory_providers
WHERE provider_npi IS NOT NULL
  AND provider_affiliated_practice_1_state IS NOT NULL
  AND provider_primary_specialty_description IS NOT NULL
  AND primary_organization_name IS NOT NULL;

-- Grant permissions (gold table)
-- Grant permissions (gold table — continued)
GRANT USE CATALOG ON CATALOG dbx_training TO `account users`;
GRANT USE SCHEMA ON SCHEMA dbx_training.workshop_shared TO `account users`;
GRANT SELECT ON TABLE dbx_training.workshop_shared.directory_providers_gold TO `account users`;

-- Allow participants to create their personal schema
GRANT CREATE SCHEMA ON CATALOG dbx_training TO `account users`;

SQL warehouse: Compute → SQL Warehouses → dbx_workshop_warehouse → Permissions → add account users as Can use.
Genie entitlement: Confirm the Genie entitlement is enabled at the workspace level (Settings → Feature enablement) before the workshop day.

## Pre-workshop checklist
Do all of these the day before delivery. Each can silently eat 5–15 minutes if it surfaces during class.


☐ Databricks Apps reachable: Click the 9-dot grid icon (app switcher, top-right) → confirm "Databricks Apps" appears in the menu. Note: the old Compute → Apps tab now shows a redirect banner — that's expected.

☐ (Optional) Verify app creation wizard works: App switcher → Databricks Apps → + Create app → select any Genie Space → confirm the 3-step wizard completes. Delete the test app.


## Module-by-module reference

### Module 0 — Get oriented (~10–12 min)
Demo: Walk through Marketplace → Catalog Explorer → UC three-level namespace → click the Genie icon top right → ask one schema question.
Talking points:
UC is the three-level namespace: catalog.schema.object
Marketplace is the data product catalog — Trilliant published, attendees just consume
Genie is the conversational AI front door (this is what your business users see)
Watch for:
Marketplace install not yet propagated to a specific workspace
Attendees confused by which Genie icon (top-right global vs in-context per-page)
Transition: "Now that you can see and ask about the data, let's get hands-on with it in SQL Editor."

### Module 1 — SQL Editor + Genie Code (~15–20 min)
Demo: Run one of the manual SQL queries. Then click the Genie Code icon (top right of SQL Editor), prompt for the same query in natural language, show the AI-generated SQL.
Talking points:
Genie Code = AI assistant for SQL authors (you're driving, AI is helping)
Genie (Module 4) = AI experience for business users (they're driving, you're the data platform)
Same model, different surface
Watch for:
Attendees can't find the Genie Code button (it's the ✦ Assistant icon on the right side of the SQL Editor)
Generated SQL using the wrong schema/catalog — attendees should always verify
Transition: "You've queried this data four different ways. Now we're going to formalize what we know into a governed metric layer."

### Module 2 — Business Semantics + Metric View (~20–25 min)
Demo: Briefly explain the gold table (pre-built, clean, null-free). Walk through the metric view DDL line by line showing source points to the gold table. Then run the metric view creation live as a group — everyone runs it together while you have it on screen.
Talking points:
Gold table is a standard medallion architecture pattern — raw → silver → gold. It's pre-built so everyone starts from the same clean data. No nulls in key dimensions means no surprises in dashboards or Genie.
Metric view = governed YAML wrapped in CREATE VIEW, sourced from the gold table
As Eli covered this morning: semantic definitions belong here, not in Genie room instructions
One definition → consumed by dashboards, Genie, SQL clients, future custom apps
The metric view creation is a live group run-through, not just a demo:
Put the notebook cell on screen
Tell everyone to open the notebook, find the metric view cell, replace firstname_lastname with their name
Count to three, everyone runs it together
Check in: "Thumbs up if it succeeded. Red card if you got an error." Fix errors before moving on — this is a hard prerequisite for Modules 3 and 4
Have everyone verify their metric view appears in Catalog Explorer before continuing
Watch for:
YAML indentation errors (the most common failure mode)
version: 1.1 not supported on older workspaces
Attendees who didn't replace firstname_lastname — their metric view will be saved under the wrong schema or fail. If they can't fix it quickly, point them to dbx_training.workshop_shared.provider_metric_view as a shared fallback so they can keep up
Diagnose Error button is the friend — show attendees where it is
Section 2.4 caution exercise: see Part 2 for the full verbatim script. Total ~5–7 minutes.
Transition: "Now let's see this metric view consumed in two different ways — first as a dashboard, then as a Genie Space."

### Module 3 — AI/BI Dashboards (~30–40 min)
Demo: Create a dashboard. Set up two pages (Provider Overview, Provider Detail). Add both datasets — metric view via Add dataset (catalog browser) data; gold table via Add SQL dataset → table browser → SELECT * auto-populates. In 3.1b, build two bar charts from the gold table first (raw column names, manual aggregation). In 3.2+, rebuild using the metric view — no SQL, dropdown only. The contrast is the teaching moment.
Talking points:
One dataset (the metric view), many widgets — no SQL duplication
If the metric view's definition changes, every widget updates
Filters use metric view dimensions; pick from a dropdown, no SQL
Watch for:
Lakeview UI variations across workspace versions ("Counter" vs "Big Number" vs "Value")
Filter dialog differences
In 3.1b attendees intentionally use the gold table (raw SQL dataset) — that's expected. From 3.2 onward, redirect anyone writing raw SQL to the metric view dropdown instead
AI-generated dashboards may still create unnecessary derived views or misapply top-N limits — this is the headline teaching moment of the module (see 3.7 debrief below)
Transition: "Same metric view, now consumed conversationally by business users via Genie."

### Module 4 — Genie Spaces (~30–40 min)
Demo: Open prebuilt Genie Space (or create one). Add only the metric view as data. Ask 2–3 questions. Show the SQL Genie generated. Then show how to add room instructions and example SQL.
Talking points:
Empty-room baseline first — see what Genie does on metadata + metric view alone
Add instructions only for what the metric view can't express (synonyms, perf guardrails, disambiguation)
As Eli covered this morning: don't instruction-stuff. Fix the root cause where it lives.
Watch for:
State name encoding (NC vs North Carolina) — known failure
Specialty matching ('Cardiology' doesn't match 'Cardiology - Interventional') — known failure
Attendees adding metric definitions to room instructions — redirect to metric view
Sections 4.3, 4.4, 4.7 (Agent Mode vs. Chat Mode), 4.8 (Benchmarks) deeper scripts: see Part 2.

### Module 5 (BONUS) — Genie Chat App (~10 min, optional)


## Demo: App switcher (9-dot grid icon, top-right) → Databricks Apps → + Create app → select your Genie Space → compute size Medium → name it (lowercase, hyphens, unique) → deploy. Share the URL.


## Talking points:

## No code required — the chat UI is built in; you just attach your Genie Space

## App inherits all your room instructions, example SQL, and benchmarks from Module 4

## Access spectrum: SQL Editor (power users) → Dashboard (visual) → Genie Space (workspace users) → App (anyone with a link)

## One governed metric view powers all four access patterns


## Watch for:

## Attendees can't find Databricks Apps — use app switcher (9-dot grid, top-right), NOT Compute → Apps (that tab now shows a redirect)

## App name rules: lowercase, hyphens only, unique, cannot change after creation


## Transition: "One governed definition, multiple access patterns. Fix it once, update everywhere."


## If Time Is Short — What to Cut (in order)


## Cut first: 3.1b gold table charts — demo it yourself, skip their hands-on build. Saves ~10 min. They still see the contrast.

## Cut second: 3.7 AI dashboard generation — mention it, skip the hands-on. Saves ~8 min. Demo on your screen.

## Cut third: 2.4 patient panel exercise — skip entirely if behind. Saves ~7 min. Not a prerequisite.

## Cut fourth: 4.7 Agent Mode vs. Chat Mode — let them read the workbook section after. Saves ~5 min.

## Cut last: Module 0 hands-on — demo only. Saves ~5 min. Attendees will explore naturally during Module 1.

## Never cut: Module 2 metric view creation (hard prerequisite), Module 3.2+ metric view dashboard (core lesson), Module 4 baseline → instructions → benchmarks loop (the discipline).


## Q&A prep — anticipated questions


### Questions from a Tableau / Power BI audience
"How does AI/BI compare to Tableau?"
AI/BI is native to the Lakehouse — no extract, no connector required. The key structural difference: the metric layer lives in Unity Catalog, not in the viz tool. One governed definition serves dashboards, Genie, and SQL simultaneously. Tableau requires separate calculated fields per workbook.
"Do we have to replace our existing Tableau dashboards?"
No. Tableau connected to Databricks via the native connector continues to work. AI/BI is the right choice for new workloads — especially anything you want to pair with Genie. The two coexist without conflict.
"Tableau Hyper extracts are very fast. How does AI/BI performance compare?"
Hyper is fast because it's a local copy of the data. AI/BI queries live Delta Lake through Databricks SQL with Photon vectorized execution — no extract cycle. For typical dashboard loads, latency is comparable. The real advantage: data is always fresh without a refresh schedule.
"We have complex LOD expressions in Tableau. Can we replicate those?"
Most LODs — COUNT DISTINCT, filtered aggregations — map directly to metric view measures using the FILTER(WHERE ...) YAML syntax. Very complex multi-level LODs are better modeled in the gold table upstream than in the viz layer — a more durable design regardless of which BI product you're using.
"Can I build the same custom visualizations I have in Tableau?"
AI/BI covers the standard chart types plus choropleth maps. Tableau's visualization library is broader — extensions, custom mark types. For users who need highly custom viz, Tableau is still the right tool. AI/BI's edge is AI generation, metric view integration, and the Genie conversational layer.
"How is Genie different from Power BI Q&A or Copilot?"
Power BI Q&A is opaque — you see an answer but not the SQL. Genie shows the SQL it generated, so you can verify it's using your governed metric view definitions and correct it if it's wrong. It's also grounded in Unity Catalog governance, not a tool-local semantic model.
"Power BI DirectQuery hits live data. Is this the same?"
Yes — AI/BI always queries live data, no import cache. It's closer to DirectQuery but without the performance ceiling, because Databricks SQL and Photon are optimized for large-scale live queries.
"We use row-level security in Tableau / Power BI. How does that work here?"
RLS lives in Unity Catalog, not in the viz tool. One policy governs who sees what across AI/BI dashboards, Genie, SQL Editor, and every other Databricks consumer. You define it once instead of re-implementing it per product.
"Can we embed AI/BI dashboards in our own applications?"
Yes — AI/BI has an embedding API, similar to Tableau Embedded Analytics or Power BI Embedded.
"What about scheduled report delivery? We use Tableau Subscriptions."
AI/BI supports scheduled subscriptions with email delivery. You can also use Databricks Workflows to decouple data refresh from the dashboard — the dashboard stays fresh without a manual trigger or a separate extract job.
"What's the difference between Chat mode and Agent mode in Genie?"
Chat mode = one question, one SQL query, fast. Agent mode = multi-step reasoning — can decompose complex questions, self-correct, ask clarifying questions. Start with Chat for simple lookups; switch to Agent for comparisons, ratios, or vague questions.

"Where did Apps go? I can't find it under Compute."
Apps moved to its own home. Click the 9-dot grid icon (top-right corner) → "Databricks Apps." The Compute → Apps tab now just shows a redirect banner — that's expected behavior.


When in doubt, defer: "Great question — we'll touch on that in Module 4" or "Let's chat after the session."

## Wrap-up — three takeaways
Metric views are your source of truth. Dashboards, Genie, custom apps — all consume the same definitions.
Genie Code is for authors. Genie is for consumers. Different products, same underlying intelligence and governance.
When something goes wrong, fix the root cause in the right layer. The principle Eli described this morning: column comment, metric view definition, example SQL, or synonym. Find the layer and fix there. Don't pile up instructions.
Where to go next:
Databricks AI/BI docs: docs.databricks.com/aibi
Eli's blog: medium.com/@eliswanson/a-practical-guide-to-genie-space-optimization
Reach out to your Databricks SA team
———
# Part 2: Verbatim Script
Read these aloud or rehearse from them. Brackets like [do X] are stage directions.

## Opening (~3 min)
"Welcome, everyone. Over the next two and a half hours we're going to build a complete AI/BI experience together — from raw data, to a governed metric view, to a dashboard, to a Genie Space that business users can talk to.
The throughline of everything we'll do is one question: how do we make data analytics consistent across products, across teams, and now across AI tools? When a CFO asks 'how many active providers do we have' in a dashboard, in Slack via Genie, and in a board deck — they should all get the same number. Today is about building that consistency from the ground up.
Quick housekeeping. You'll be accessing this workshop through the Vocareum lab environment — the workspace should already be open in front of you. Your workbook is in the Drive folder I shared — open it now. We'll do five modules. I'll demo each one for a few minutes, then you'll get hands-on time. Stop me anytime with questions."

## Module 0 demo — Get oriented (~3 min of demo, then 8 min hands-on)
"Let me show you where we're starting. [open Catalog Explorer] You see a catalog called trilliant_health_trilliant_health_s_provider_directory. That's a data product from Databricks Marketplace — Trilliant Health published it, we installed it in this workspace, now it shows up just like any other catalog. [click directory_providers] One table, several million provider records.
Quick concept: Unity Catalog organizes everything in a three-level namespace — catalog dot schema dot object. Catalog is the top container, schema is a namespace inside, objects are tables, views, metric views, and so on.
Before we write any SQL, watch this. [click the Genie icon top right] This is Genie. It's a conversational AI experience over your governed data. I can just ask — [type: 'What tables are in trillianthealthtrillianthealthsproviderdirectory.directory?'] — and it answers. This is the experience your business users get. No SQL, no notebooks, no BI tool, just natural language.
Your turn. Open Catalog Explorer, browse the dataset, ask Genie a couple of questions about it. Eight minutes. Go."

## Module 1 demo — SQL Editor + Genie Code (~3 min demo, 15 min hands-on)
"OK — you've explored the data. Now let's write actual SQL. [open SQL Editor] Your workbook has three manual queries in your workbook to type out. But before you start typing — look at the top right. That icon. [click it] That's Genie Code. Different surface than Genie, similar idea. Watch — [type: 'count distinct providers by specialty, top 10'] It generates SQL, I review it, I can edit it, I run it.
The distinction matters. Genie Code is for query authors. You're driving, the AI is helping you write SQL faster. Genie — the one you used in Module 0 — is for business users who don't write SQL at all. Same underlying tech, different audiences.
Your turn. Run the three manual queries, then use Genie Code for the two prompted exercises at the end of the module. Fifteen minutes."

## Module 2 demo — Metric View (~5 min demo, 15 min hands-on)
"OK. You've queried this data four ways. Now we're going to formalize what we know into a governed semantic layer.
First — a quick concept. Before I built this workshop, I pre-created a gold table at dbx_training.workshop_shared.directory_providers_gold. That's a clean version of the provider directory: records with null values in key dimensions — specialty, state, organization — are already filtered out. This is a standard pattern called the medallion architecture: raw data, cleaned silver, business-ready gold. I handled this step so you don't have to. The point is: every downstream object in this workshop — your metric view, your dashboards, your Genie Space — reads from the same clean table. No nulls, no surprises.
Now. Watch. [open the notebook, scroll to the metric view cell] This is a metric view. It's a YAML spec wrapped in a CREATE VIEW. Two parts that matter:
Dimensions — [point at section] These are the things you slice by. Specialty, state, county, primary organization. Each maps to a column in the gold table.
Measures — [point at section] These are the things you count or compute. Provider Count. Active Provider Count. Average Practices per Provider. Each is a SQL expression.
Here's the magic. Once this exists, EVERY tool that talks to this data uses the same definition. Your dashboard in Module 3, your Genie Space in Module 4, your future Python app, your BI tool — they all see 'Active Provider Count' meaning the same thing. No drift. No re-derivation. This is what Eli was talking about this morning. Semantics belong here, not in your AI room instructions.
Now — I'm going to do this one with you, not just show you. Open the notebook. Find the metric view cell. Replace firstname_lastname with your own name — lowercase, underscore between first and last. I'll give you sixty seconds to do that. [pause]
Ready? Three, two, one — run it. [run on screen simultaneously]
[pause 15-20 seconds] OK — thumbs up if it succeeded. If you got an error, hold up and I'll come to you. [address errors — most common: YAML indentation, didn't replace firstname_lastname]
Now open Catalog Explorer. Navigate to dbx_training, your schema, and confirm provider_metric_view is there. If you can see it, you're good to go. This is your source of truth for everything that follows — don't skip this check."

### 2.4 Patient Panel caution exercise (~5–7 min)
[Pull the room back together after Module 2 hands-on]
Setup (~30 sec): "One more question before we move to dashboards. The dataset has a column called panel_percent_female — it's a percentage at the provider level. Like 'forty-seven percent of Dr. X's patients are female.' Should we add it as a measure to our metric view? Hands up for yes."
Vote (~30 sec): [Read the room. Whether yes/no/depends is the majority, set up the trap.]
Walk through the trap (~2 min): "Let me show you why this might be the most important moment in the workshop. Imagine I have two providers. Doctor A has 1,000 patients, 47% female. Doctor B has 10 patients, 50% female.
If I run AVG(panel_percent_female) across those two providers — what do I get? 48.5%. Right? Average of 47 and 50.
But what's the TRUE patient-weighted average? [pause] Doctor A's 1,000 patients dominate. The actual percentage of female patients across both panels is 47.03%. The naive average gives every provider equal weight regardless of how many patients they have.
That's wrong, and it's a wrong answer that LOOKS right. [if panel_total exists, run the SQL showing the divergence] [if not, just narrate]
Without a panel_size column to weight by, you literally cannot compute the right number. So this column is a trap disguised as a metric."
Discuss (~2 min): "Three questions:
One — would adding the wrong version of this metric to Genie be worse than not having it at all? [pause for answer] Yes. Wrong answers erode trust faster than missing answers. Once a dashboard lies to a VP, the whole metric view loses credibility.
Two — what would it take to expose this safely? [pause] Panel size field, plus a properly weighted measure expression: SUM of panel_pct times panel_size, divided by SUM of panel_size.
Three — where would that live? [pause] As a new measure in the metric view spec. Not as an instruction in your Genie space. The math goes in the semantic layer, where it can be governed and tested."
Land it (~30 sec): "Sometimes the right answer is to leave a column OUT of your governed semantic layer until you can define it correctly. That's the discipline. Treat your metric view like a contract."

## Module 3 demo — Dashboards on the metric view (~5 min demo, 25 min hands-on)
"Now we're going to build a dashboard. But here's what's different from every dashboard you've built before — we're NOT going to write any SQL. We're going to point at our metric view and assemble widgets from a dropdown.
Watch. [create dashboard] First, set up two pages — rename the default to 'Provider Overview,' add a second called 'Provider Detail.' [click Data tab] Now add both datasets: click + Add → Browse data to add the metric view; click Add SQL dataset and use the table browser to add the gold table — the SQL auto-populates. [switch to Gold Table page] In 3.1b you'll build two charts here using the gold table directly — you'll have to know exact column names and set aggregations manually per widget. [switch to Metric View page] Then in 3.2 you'll rebuild the same charts using the metric view — no SQL, just dropdowns.
[drag in a Counter widget] Pick a measure: Active Provider Count. Title: Active Providers. Save. [drag in a bar chart] X-axis: Specialty. Y-axis: Provider Count. Save. No SQL, no COUNT(DISTINCT), no risk of two charts disagreeing.
Why does this matter? Because every widget reads from the same source of truth. If I change the definition of 'Active Provider Count' next week — say I add a tenure filter — every widget updates automatically. No tracking down forty dashboards.
[IF TIME IS SHORT: Demo 3.1b yourself on screen — build one gold table chart live to show the per-widget pain, then say: "You saw the problem. Jump to Section 3.2 — that's the part that matters." This saves ~10 min.]

Your turn. Build the dashboard following the workbook. Twenty minutes. Then in the last five minutes, try the AI generation prompt at the end of Module 3 — but DON'T just admire what it produced. We're going to do a verification debrief together."

### 3.7 Verification debrief (~5 min)
[Pull the room back together after they've run the AI generation prompt]
"OK — Genie just built our dashboard in 30 seconds. Looks great, right? Now look closely.
[look at the Data tab] Did Genie create extra derived datasets — things like 'Top 20 Organizations' as their own dataset? Was that necessary, or could a widget config toggle have done the same thing? Sometimes AI takes a SQL shortcut that adds complexity you didn't ask for.
[look at a bar chart] Check the top-N limit. Is it applied at the widget level — so end users always see the top 20 — or only inside the underlying query? Those are different behaviors. If it's only in the query, a user who drills in sees everything unfiltered.
[click a filter] Do all the filters affect all the widgets, or just some? Genie sometimes scopes a filter to only the dataset it was generated from.
Three questions for the room:
One — which of these are real bugs versus acceptable defaults?
Two — where would you fix each one? The prompt, the widget config, the dataset, or the metric view?
Three — if you were handing this dashboard to a VP tomorrow, which fixes are non-negotiable?
This is the most important habit you'll take from today: every AI artifact needs a second pass. AI gives you 80% of the work for free. The remaining 20% — catching edge cases, fixing limits, sanity-checking filter scoping — is where you earn your salary. Same discipline as the patient panel question in Module 2.4. Preview of the benchmark loop in Module 4.7."

## Module 4 demo — Genie Space (~5 min demo, 25 min hands-on)
"Last module. We've built a dashboard for visual consumption. Now we're going to enable conversational consumption — Genie.
[open or create Genie Space] Same metric view as the dashboard. ONE data asset. No raw tables, no extra views — just the metric view. Why? Because that's where the governed definitions live. Per Eli — don't add tables for things the metric view can already answer.
[type a question: 'How many active providers are in California?'] Watch the SQL Genie generated. [point at it] It used MEASURE(Active Provider Count). It filtered on State = 'CA'. Because the metric view DEFINED those concepts, Genie just picks them up. The dashboard you built and this Genie Space are answering the same question the same way.
One critical workshop principle, then I'll let you go. EMPTY ROOM BASELINE FIRST. Ask 3 or 4 questions BEFORE you add any room instructions. See what works on metadata alone. Then add instructions. Then re-ask. That's how you measure whether your instructions actually helped.
Your turn. Module 4 walks you through it. Twenty-five minutes."

### 4.3 Empty-room baseline debrief (~3 min)
[Pull room back after baseline attempts]
"Show of hands — who got the right answer for 'active providers in California' on the first try? [pause] OK, who got something wrong on 'cardiology providers'? [pause]

Two common failures you probably hit — and these are KNOWN failure modes, not bugs: OK, who got something wrong on 'cardiology vs oncology'? [pause]
Details:
One — 'NC' versus 'North Carolina'. Genie might say 'I can't find providers in North Carolina' because the data has 2-letter codes. That's a synonym problem — fix it in room instructions, not the metric view.
Two — 'cardiology' as a specialty returns nothing. Why? [pause] Because the actual values are like 'Cardiology - Interventional', 'Cardiology - Electrophysiology'. Fuzzy matching is needed. Fix this with an example SQL using lower(...) IN (...) — or add a clarifying column comment.
For each failure, ask yourself: where should this fix go? Not always the metric view. Sometimes it's a comment, sometimes a synonym, sometimes an example query."

### 4.4 Prompt engineering discussion (~3 min)
"Did anyone get DIFFERENT answers to the four phrasings of the same question? [pause]
If you all got the same answer four ways — that's actually a GOOD sign. It means the metric view is robust to phrasing variation. Eli would say your semantic layer is doing its job.
Now try a HARDER phrasing variation. Try 'doctors' instead of 'providers.' Or 'physicians.' [pause for trying] What happened? Did Genie know they mean the same thing? [pause]
If yes — your room instructions are good. If no — that's a synonym you should add to room instructions. Not the metric view; that would be over-loading the metric view with terminology."

### 4.7 Agent Mode vs. Chat Mode (~2 min, before benchmarks)


### [Pause hands-on for this brief framing]


### "Before you run the benchmarks — one concept you'll want before you go further. Genie has two modes. Chat mode translates your question into a single SQL query. Fast, deterministic, great for well-scoped questions. Agent mode can plan, decompose, and iterate — multiple queries, self-correction, clarifying questions back to you.


### Simple rule of thumb: if your question maps to one SQL query, Chat mode is faster. If your question requires comparing two things, computing a ratio, or is deliberately vague — Agent mode shines.


### Your workbook Section 4.7 has the full breakdown with sample questions for each mode. As you test the benchmarks in 4.8, pay attention to which mode you're in."


### [Resume hands-on — direct them to Section 4.7 then 4.8 Benchmarks]


### 4.8 Benchmarks debrief (~5 min)
"Run the 3 benchmarks. Don't bother filling in a table — just SEE what passes and fails.
[after they've done it] OK. For each failure, the question is: what's the root cause, and where do I fix it?
Five root-cause categories (Eli's framework):
One — DISTINCT or granularity. Fix in the metric view measure expression.
Two — metadata (wrong column picked). Fix the column or measure comment.
Three — synonym/entity matching (user says 'doctors,' data says 'providers'). Fix in room instructions, or in the dimension comment.
Four — ambiguous metric (term has multiple valid definitions). Add a clarifying measure.
Five — performance anti-pattern (Genie wrote ILIKE '%...%'). Add an example SQL with the correct pattern.
The discipline: fix it ONCE in the right place, and re-run all 3 benchmarks. Don't paper over symptoms with more instructions."

### 4.9 BI Engineer Test — closing thought (~1 min)
"One closing thought. Imagine you hand your Genie Space — just the configuration, no documentation — to a BI engineer who has never seen this data. Give them one hour. Can they write the correct query on first try?
If yes, you've built a real semantic layer. If no, something's missing — and your job is to figure out WHAT and encode it. That's the loop. That's Eli's whole point. That's the difference between a Genie Space that works in your demo and a Genie Space that works for real users."

### Module 5 (BONUS) — Genie Chat App (~2 min demo, only if time permits)


## "If we have a few minutes left — or for those of you who finish Module 4 early — there's a bonus module. It takes your Genie Space and packages it as a standalone web app with its own URL. Three clicks, no code.


## [click the 9-dot grid icon top-right → Databricks Apps → + Create app]


## You pick your Genie Space from a dropdown, pick a compute size, name it, done. Now you have a URL you can hand to anyone with Databricks credentials — they get a chat interface backed by your governed metric view without ever touching the workspace UI.


## Think of it as the access spectrum: SQL Editor for power users, Dashboard for visual consumers, Genie Space for workspace users, Databricks App for anyone with a link. Same metric view powers all four.


## If you finish Module 4 early, give it a try. Module 5 in your workbook has the full steps."


## Two more things before you go:


## Each module has an Optional/Bonus section in the workbook. Module 3 covers sharing, PDF export, and scheduled email subscriptions. Module 4 covers sharing conversations, downloading results, and linking Genie to dashboards.


## Module 5 (bonus) — deploy your Genie Space as a standalone web app in under five minutes. Three clicks, a shareable URL, no code. Try it on your own after the session.


## Wrap-up (~3 min)
"OK — we're at time. Three things to take with you.
One. Your governed metric view is the source of truth. The dashboard you built and the Genie Space you built both consume the same definitions. Your future custom apps will too. Define it once, use it everywhere. Stop re-deriving 'active provider' in twenty different places.
Two. Genie Code is for authors. Genie is for consumers. Different products, same underlying intelligence, same governance. Pick the right surface for the right user.
Three. When something goes wrong with Genie — and things will go wrong — fix the root cause in the right layer. Don't pile up room instructions. The principle Eli described this morning: column comment, metric view definition, example SQL, or synonym. Find the right layer, fix there, move on.
Where to next: Databricks AI/BI docs at docs.databricks.com/aibi. Eli's blog has the deep dive. And your Databricks SA team is one Slack message away.
Thank you all. Questions?"
