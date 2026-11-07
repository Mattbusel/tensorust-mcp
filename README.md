# Tensorust MCP server

Remote MCP server (Streamable HTTP) from Tensorust LLC. Nothing to install: point any MCP client at the URL. The free tier needs no key, account or OAuth; an optional `Authorization: Bearer tdk_...` key returns full rows.

**Endpoint:** `https://shop.tensorust.com/api/mcp`
**Docs:** https://shop.tensorust.com/mcp
**Official MCP Registry:** `com.tensorust/ontology` ([server.json](server.json))
**Glama:** https://glama.ai/mcp/connectors/com.tensorust.shop/tensorust-ontology

It lists and samples a catalog of 145 entity-level datasets, and it answers who owns and controls US organizations and property, what private lenders mark their loans at, which federal research awards cite products from banned vendors, and which buyers fall under the ROAD Act investor ban. Everything comes from public records: SEC (10-Q, BDC data sets, Exhibit 21), USAspending/FPDS, NIH RePORTER, Europe PMC, DOL Form 5500, FCC (ULS, ECFS, Form 323), FAA registry, USAC E-Rate and county property rolls in 17 counties.

## Install

Claude Code:

```
claude mcp add --transport http tensorust https://shop.tensorust.com/api/mcp
```

Claude (claude.ai / Desktop): Settings, Connectors, Add custom connector, paste the URL, leave OAuth empty.

Cursor (`~/.cursor/mcp.json`), VS Code and other clients:

```json
{ "mcpServers": { "tensorust": { "url": "https://shop.tensorust.com/api/mcp" } } }
```

Quick check from a shell:

```
curl -s -X POST https://shop.tensorust.com/api/mcp -H 'content-type: application/json' -H 'accept: application/json, text/event-stream' -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Tools (38)

### Dataset catalog (145 entity-level datasets)

| tool | what it does |
|---|---|
| `list_datasets` | Browse 145 datasets built from public records (SEC, USAspending, FCC, CMS, EPA, DOL, USAC, county rolls and more): going-concern warnings, BDC non-accruals, E-Rate deadlines, Upper C-band fleets and others. Filter by words or category; returns one line each, row count, tiers, the dataset page and the free sample link. Free. |
| `get_dataset` | One dataset's card: who it is for, what each row holds, the headline finding, coverage, sources, tiers and how to buy it by invoice. Free. |
| `get_dataset_sample` | The free 25-row sample as CSV, the same file the dataset's tensorust.com page publishes. Full files are sold by invoice and never returned by a tool. Free. |

Plain HTTP, no MCP client needed: `GET https://shop.tensorust.com/api/datasets?q=<words>`, `GET https://shop.tensorust.com/api/datasets/<slug>`, `GET https://shop.tensorust.com/api/datasets/<slug>?sample=1` (CSV). Browse by hand at https://tensorust.com/datasets/.

### Ownership and control graph

| tool | what it does |
|---|---|
| `lookup_organization` | Resolve a company or organization name to its Tensorust Ontology card: legal entity or control group, data sources, linked objects by type (homes owned, control group members, subsidiaries, FCC licenses, federal awards, E-Rate requests) and the federal rules that apply to it, with status. Free: counts and previews. full=true: every linked object and property, $0.05 per call over x402 (USDC on Base) or with a Tensorust key. |
| `search_organizations` | Search organizations, control groups and other objects by name. |
| `get_object` | Expand any ontology id (for example `org:LOCKHEED MARTIN`, `cg:632693`, `rule:ROAD-1001`) into its card. Free tier returns counts and previews; full=true returns every linked object and property. |
| `who_owns_home` | Given a US street address and ZIP, return the owner of record and, if it is a business entity, its parent control group, how many homes the group holds and whether it is a large institutional investor under the ROAD Act (350+ homes). |
| `rule_coverage` | List the federal rules Tensorust models (ROAD Act investor ban, DoD SF 328 ownership disclosure, FCC foreign adversary attestation, Upper C-band elections, E-Rate deadlines) with counts and the first organizations each applies to. |
| `aircraft_owner` | Look up a US turbine aircraft by N-number (or search by registered owner name): registered owner, whether it is held by an owner trustee, model, and its FCC radio altimeter rebate category and proposed rebate. |

### Private credit (BDC loan marks)

| tool | what it does |
|---|---|
| `lender_marks` | Private credit marks for a US company: every BDC (business development company) fund that holds its first-lien term loan, the fair value each reported as a share of par in its 10-Q for 2026-06-30 (SEC filings), the manager (Ares, Blue Owl, Goldman, Golub, Apollo...) and how far the lenders disagree. A low mark by one manager tends to lead: lenders 10-20 points above the lowest peer cut by 5+ points the next quarter 20.4% of the time vs 2.8% when lenders agree. Free: lowest and highest manager mark, the gap and three funds. full=true: every fund's mark, par, prior-quarter mark and filing link, $0.05 per call over x402 (USDC on Base) or with a Tensorust key. |
| `credit_disagreements` | US companies whose first-lien loan two or more BDC managers mark 10+ points apart at 2026-06-30 (SEC filings), widest gap first: borrower, lowest and highest manager, marks and par. |
| `credit_q3_cuts` | Live through earnings season: BDC funds that cut a first-lien loan's mark by 5+ points from Jun 30 to Sep 30 2026, pulled from each Q3 10-Q on EDGAR the day it is filed, biggest cut first, with the holders that have not filed Q3 yet and still carry the same loan 5+ points higher (likely next markdowns). Free: the top 10. full=true: every cut with every pending holder, $0.25 per call over x402 (USDC on Base) or with a Tensorust key. |

### Research Exposure Ledger (banned-vendor lists x federal research awards)

| tool | what it does |
|---|---|
| `exposure_summary` | Research Exposure Ledger headline numbers: federal research awards whose 2025-26 papers name a vendor on a banned-vendor list (DoD 1260H Chinese military companies, Section 889 telecom/video, FCC Covered List, UFLPA forced-labor entity list, BIOSECURE biotech companies of concern) as the sequencing, contract-research, drone, camera or network provider they used. Per list: listed vendors and listing dates, papers, awards, FY2025 dollars (broad, narrow-attribution and most-likely-paid cuts), institutions, direct federal purchases; per rule (Section 889 grants ban, American Security Drone Act, BIOSECURE grace period, UFLPA import presumption; FCC Covered List as information only): award counts by status; the in-force / no-grace lead cut (in_force) and the split by award class; top agencies and institutions. Lists are re-pulled from the Federal Register, eCFR, FCC and DHS, and every award is re-scored when one changes. Free. |
| `exposure_institution` | For a university, hospital or other federal award recipient (name or UEI): how many of its federal awards have 2025-26 papers naming a listed vendor (BGI/MGI, Novogene, WuXi AppTec, DJI, Hikvision, Huawei and others) in methods, the FY2025 dollars, the split by list and by rule status (e.g. BIOSECURE no-grace vs grace-eligible, Section 889 in force), and its largest awards. Free: totals and the 5 largest awards. full=true: every award with the evidence sentence and paper link, $0.05 per call over x402 (USDC on Base) or with a Tensorust key. |
| `exposure_awards` | Search the Research Exposure Ledger's awards. |
| `exposure_screen_purchases` | Score up to 2,000 vendor payment or purchase-order lines (vendor, description, amount, date YYYY-MM-DD, award number, funding source) against the DoD 1260H, Section 889, FCC Covered, UFLPA and BIOSECURE lists as they stood on each line's date. |
| `exposure_workspace` | The exposure report for a private Exposure Ledger workspace (created at https://shop.tensorust.com/exposure/workspace): every stored vendor payment / PO line re-scored against today's lists, totals by vendor and rule, prohibited-use and review lines, and the join to the ledger (corroborated awards; ledger awards with no purchase line). Needs the workspace token (xlw_...). Free. |
| `exposure_dbgap` | NIH controlled-access (dbGaP) human studies whose sequencing runs in ENA were made on BGI/MGI instruments (DNBSEQ, BGISEQ, MGISEQ), each joined to its NIH grant (from the dbGaP study record, or an identical-title NIH RePORTER project), with the grantee, PI, project end date, RePORTER award amounts and the ledger's rule status (BIOSECURE no-grace, UFLPA import presumption since Oct 29 2026). Free. |

### Performance Ledger (small-business set-asides)

| tool | what it does |
|---|---|
| `perf_summary` | Tensorust Performance Ledger headline: small-business set-aside contract dollars FY2025, FY2026 and FY2027 to date (USAspending/FPDS) set against each prime's own DOL Form 5500 headcount (active participants) and its FSRS subawards. |
| `perf_firm` | One set-aside contractor (name or UEI): set-aside obligations by year, what was bought (labor, R&D, SaaS, travel, construction, products), its own Form 5500 workforce (active participants, EIN, EFAST2 ACK_ID, plan type, union/multiemployer flag, parent's headcount, largest same-name sponsor nationwide), labor dollars per participant against peers, flag tier and explanation codes, FSRS subawards and ontology rules. Free. full=true with a Tensorust key adds the case file: every award (office, PSC, offers, SBIR code, end date, subawards), every 5500 filing and the citations (13 CFR 125.6, FAR 52.219-14, VAAR 852.219-75, 13 CFR 121.103(h)). |
| `perf_queue` | The Performance Ledger review queue: set-aside firms with $1M+/yr of labor/R&D work billing $500K+ per active Form 5500 participant. |
| `perf_recent` | Set-aside labor/R&D actions in FY2027 to date (from Oct 1 2026, USAspending bulk pull) on awards held by firms in the Performance Ledger review queue, newest first: award, PIID, office, PSC, set-aside, FY2027 obligation, action date, tier and dollars per participant. Free: count and dollars. With a Tensorust key: the rows. |

### Contractor Credit (federal contractors financed by BDC lenders)

| tool | what it does |
|---|---|
| `contractor_credit` | Private credit x federal exposure for one company (name, 12-character UEI or 9-digit EIN): the operating company behind a BDC holdco borrower (resolved through exact name, holdco-stripped name, SAM parent UEI and Form 5500 EIN, with a confidence grade), its federal obligations since FY2025 and DoD FY2024-26 by agency, what every BDC lender marks its first-lien loan at (Jun 30 2026, plus Sep 30 2026 marks pulled from EDGAR as each 10-Q posts), the Q1->Q2 and Q2->Q3 moves, Form 5500 headcount 2024 vs 2025, FCC licenses, turbine aircraft and stress signals. Free: the card. full=true: every lender line with filing link, every recipient UEI, aircraft N-numbers, call signs and up to 17 quarters of mark history, $0.05 per call over x402 (USDC on Base) or with a Tensorust key. |
| `contractor_credit_watch` | BDC-financed federal contractors with a stress signal (a lender marks the first-lien loan below 90% of par, cut it 5+ points in a quarter including Q3 2026 marks as filed, or two managers sit 10+ points apart), largest federal exposure first. |
| `agency_credit_exposure` | Per federal agency: how many BDC-financed contractors it pays, their obligations since FY2025, and how much sits with contractors whose lenders are marking the loan down, with the names. |

### Clearinghouse (FCC Upper C-band and rip-and-replace claims)

| tool | what it does |
|---|---|
| `clearinghouse_precheck` | Validate one FCC Upper C-band claim before it is filed, against the records the Clearinghouse pays from: a lump sum election (kind=lumpsum: claimant_name, call_sign, site_id, antennas as 'ID=qty/ID=qty', antenna_type single/multi/smb/lmb/gateway/tfe, mvpd_site) checked against the Oct 15 2026 incumbent list (DA 26-1126), the 2020 and 2025 lists, licensee FRNs, Form 323 control groups, dish co-location across call signs and OFAC / foreign-adversary flags; or a radio altimeter rebate (kind=altimeter: claimant_name, n_number, radio_altimeters, retrofit_type upgrade/replacement, trustee_certification, optional serial) checked against the FAA Aircraft Registry (owner of record, trustee, registration status and expiry, airframe serial, ownership changes since FCC 26-46, co-owners). Returns pass / warn / fail per rule with the reason and the record compared, the verdict (accepted / review / rejected) and the amount at the preliminary DA 26-1049 / proposed DA 26-1076 figures. Nothing is recorded. Free. |
| `clearinghouse_validate` | Free validation API for the FCC Upper C-band Clearinghouse: send one claim or a batch of up to 100 (claims: [...]), lump sum elections (kind=lumpsum: claimant_name, call_sign, site_id, antennas 'ID=qty/ID=qty', antenna_type, mvpd_site) and radio altimeter rebates (kind=altimeter: claimant_name, n_number, radio_altimeters, retrofit_type, trustee_certification, optional serial) mixed. Runs the same 24 rules as the Tensorust Clearinghouse Console against the Oct 15 2026 incumbent list (DA 26-1126), the 2020 and 2025 lists, Form 323 control groups, OFAC, the preliminary Cost Catalog (DA 26-1049), the FAA Aircraft Registry and the proposed altimeter amounts (DA 26-1076). Duplicate rules compare the claims in the call with each other. Returns per claim the verdict (accepted / review / rejected), the amount, and every rule hit with its reason and the record compared, plus totals and hits per rule. all=true adds the passing rules. Nothing is stored. Limit: 30 calls a minute and 2,000 claims an hour per IP. |
| `clearinghouse_record` | For an Upper C-band earth station call sign: registrant, licensee FRN, former registrants on the 2020 / Nov 2025 lists, control group (FRN plus Form 323 50%+ entity ownership) with sister licensees, other call signs filed at the same dish location, and OFAC / foreign-adversary flags. For an N-number: owner of record (latest FAA MASTER file), airframe (model code + serial), registration status and expiry, certificate date, fractional / co-owners, prior airframe on the N-number, trustee / Part 121 / government flags. Free. |
| `clearinghouse_program` | Program-wide findings on the real records the Upper C-band Clearinghouse will pay against: call signs whose registrant changed since the 2020 / 2025 lists, control groups, dish locations filed under two or more call signs (antennas and dollars), registrants with foreign-adversary flags, FAA registrations that expire before the Oct 31 2034 deadline, ownership changes since the order, co-owned aircraft, N-numbers previously on another airframe. With build date and sources. Free. |
| `program_precheck` | Validate one claim with the Tensorust Clearinghouse program modules. |
| `rip_replace_deadlines` | FCC rip-and-replace (SCRP) program record: 126 granted applications and 85 recipients with approved cost estimates (DA 22-774), applications grouped by RRD deadline after the May 1 and Sep 3, 2026 extension orders with the final certification, claim, extended claim and final spending report dates for each group, and the recipients named under a reimbursement hold. Optional application (SCRP0001xxx) returns that one application. Free. |

### ROAD Act Purchase Monitor

| tool | what it does |
|---|---|
| `road_screen_purchase` | Screen a single-family home purchase (a title order) under the ROAD Act ban on large institutional investors (350+ homes, alone or in concert; effective Jan 7 2027; Treasury NPRM TREAS-DO-2026-0529). |
| `road_investor_group` | One control group by group name, any member LLC name or group id: homes held in the 17 covered counties, legal entities, largest single entity, ROAD Act status (COVERED, BUILDER_IBUYER, REVIEW, EXCLUDED, BELOW_THRESHOLD), whether a name search finds it, link types, deeds by month since 2025, Texas-registered vehicles with no homes yet, and its 25 most recent recorded purchases. Free. |
| `road_purchase_stream` | The purchase stream and agency case queue: recorded deeds (last sale per parcel, county rolls) into 350+ home groups, newest first. |
| `road_monitor_summary` | Headline numbers for the ROAD Act ban: covered investor groups and homes in 17 counties, groups no single entity of which reaches 350, groups invisible to name search, deeds into covered groups since enactment and per month, builder/iBuyer deeds, Texas vehicles with no homes yet, county coverage and deed freshness, rule dates, method and limits. Free. |

## Products behind the tools

- [Exposure Ledger](https://shop.tensorust.com/exposure)
- [Contractor Credit](https://shop.tensorust.com/contractor-credit)
- [Performance Ledger](https://shop.tensorust.com/performance-ledger)
- [ROAD Act Purchase Monitor](https://shop.tensorust.com/road-monitor)
- [Private Credit Marks](https://shop.tensorust.com/credit)
- [Clearinghouse Console](https://shop.tensorust.com/clearinghouse)
- [Ontology explorer](https://shop.tensorust.com/ontology), [Radar](https://shop.tensorust.com/radar), [Company Watch](https://shop.tensorust.com/company-watch)

Open datasets: https://huggingface.co/tensorust and https://www.kaggle.com/tensorust

## License

This repository (docs and config) is MIT. The hosted server is operated by Tensorust LLC; data terms are on each product page. Contact: mattbusel@tensorust.com
