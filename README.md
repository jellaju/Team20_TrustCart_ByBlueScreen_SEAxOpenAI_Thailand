TrustCart
Verify the listing. Trace the evidence. Escalate uncertainty.

TrustCart is an API-first, human-governed verification workflow for health-product listings. It is designed for marketplaces and seller operations that need to check product identity, advertising claims, and supporting evidence before a listing is published. A lightweight web dashboard demonstrates the same verification service from seller, shopper, and reviewer perspectives.

Built by Team BlueScreen for the Sea × OpenAI Regional Codex Hackathon Thailand — 21 November 2026.
[!IMPORTANT]
Hackathon prototype — synthetic data only. This repository uses a fictional product registry, simulated listings, and synthetic evidence. It is not connected to the Thai FDA, Shopee, or any live marketplace. APPROVED means only that the applicable demo checks passed; it is not Thai FDA certification, proof of product authenticity, legal approval, or medical advice. We are not affiliated with those organizations.

Try it: Hosted demo · Interactive API docs · [Demo script](DEMO.md) · [Technical implementation](IMPLEMENTATION.md)

The hosted demo is password-protected. Use the separately shared demo credentials; never commit credentials to the repository.
The problem: a valid registration number is not a verified listing
A health-product listing combines several different claims of trust: this registration number exists; it belongs to this particular product and licence holder; the advertised category and statements are appropriate; and the seller can substantiate what is being presented. Verifying one field alone does not establish the others.

This matters in Thailand. The Thai FDA reported identifying more than 20,000 potentially unlawful health-product advertisements per year, along with more than 3,000 health-product complaints annually, over half associated with illegal online advertising. A 2024 FDA Journal study documented 979 reports of unlawful online health-product advertising during a targeted two-month survey in 2023. These figures describe enforcement and study findings, not the number of TrustCart customers or the prevalence of violations across all listings.

The Thai FDA and major platforms are already investing in AI/API-based screening. TrustCart is not claiming that platform verification does not exist. Our product hypothesis is more specific: a reusable, evidence-linked workflow may help teams turn fragmented registration, listing-text, and document checks into a traceable pre-publication decision with a clear route to human review.
One-line problem statement
Health-product listings can display seemingly credible registration details while presenting mismatched product identities, questionable claims, or missing evidence. Review teams need a consistent way to identify and resolve those conflicts before publication, without treating uncertainty as approval.

Product: one verification engine, two delivery surfaces
Seller / marketplace listing
        |
        v
POST /v1/verifications  ──>  Registry adapter (synthetic in demo)
        |                      + rule-based identity, category,
        |                        advertising and evidence checks
        v
  APPROVED / PENDING / DENIED / UNKNOWN
        |
        +--> Evidence upload --> Recheck --> Human review when required
        |
        +--> Evidence-linked report + hash-chained audit trace
        |
        +--> API response / reviewer dashboard / shopper-facing demo widget
The API is the product; the dashboard is the demonstration and review interface. They share the same backend decision model rather than implementing separate verification systems.

Design principle: AI extracts and suggests. Deterministic rules decide. Humans govern. Optional language-model extraction must remain grounded in the listing text and cannot set or override a verification status. With no approved external AI configuration, the demo uses local rule-based screening.
Intended users and commercial buyers
Role	Job to be done	TrustCart interface
Marketplace trust-and-safety or listing-operations team	Screen listings consistently and route unresolved cases	Verification API + reviewer queue + audit report
Supplement brand, seller-operations team, or marketing agency	Find listing problems and provide missing evidence before publication	Seller dashboard + evidence/recheck flow
Shopper	Understand which checks ran and what remains unverified	Demo trust status + explanation
Regulatory reviewer or other authorized human reviewer	Inspect evidence, resolve escalations, and retain decision history	Reviewer dashboard + tamper-evident trace


Who might pay? The commercial hypotheses are (1) marketplaces or seller platforms paying for API usage / enterprise integration, and (2) brands or agencies paying for a hosted review workflow. Neither willingness to pay nor customer ROI has been validated. We are not asserting that a large marketplace would outsource functionality it can build internally.
Verification workflow and safety states
Status	Meaning in the synthetic prototype
APPROVED	Required applicable demo rules pass, confidence is at least 0.75, and no unresolved conflict requires review. Not an official certification.
PENDING	Seller evidence, a correction, or a human decision is still required.
DENIED	A conclusive synthetic rule failed or a reviewer recorded a denial. Does not establish that a physical product is counterfeit.
UNKNOWN	A source is unavailable, a reference is missing or unsupported, or verification cannot be completed. Never silently approved.


The current rule pack uses five checks:
Rule	Scope
R-01	Registration reference, product name, and licence-holder identity against the demo registry
R-02	Product category and category/claim consistency
R-03	Selected Thai/English advertising-claim patterns and required text in supported cases
R-04	Advertising permit requirements or conflicts, where present in the synthetic record
R-05	Identity declarations associated with seller-provided label/certificate evidence


For evidence uploads, the service stores SHA-256 file digests and declared fields, rather than treating a file upload as proof of document authenticity. Submitted file contents are not retained by that endpoint. Authenticating real certificates, COAs, and physical products remains future work.

Each workflow step contributes to a SHA-256 hash-chained audit trace. Changes using a stale chain_hash or a failed audit check are rejected. The risk score is an illustrative reviewer-priority heuristic, not a probability of fraud or a substitute for the rule verdict.
What is implemented in this repository
The current code includes:
- FastAPI /v1 endpoints for submitting and retrieving verifications, uploading evidence, rechecking, reviewer decisions, reports, badges, comparisons, audit checks, and aggregate agency metrics.
- A synthetic registry adapter and configurable advertising-claim rules.
- SQLite persistence for verification records, evidence metadata, decisions, and audit history in the local service.
- A password-protected demo app with Seller, Shopper, Reviewer, and Agency views.
- Optional OpenAI structured-output extraction, guarded by explicit approval for external transfer; local deterministic screening is the default demo path.
- Synthetic testing fixtures: 18 fictional registry references, 40 tagged listing scenarios, and 16 evidence-case definitions. Some evidence scenarios describe roadmap behavior rather than implemented checks.
- Automated tests in tests/. The previous README recorded 45 passing tests; re-run the suite on the submission commit before quoting a current count.
Code entry points: [`backend/app/main.py`](backend/app/main.py), [`backend/app/verification.py`](backend/app/verification.py), [`backend/app/registry.py`](backend/app/registry.py), [`backend/app/api/`](backend/app/api/), [`app.py`](app.py), and [`demo.html`](demo.html).
Important limitations
- No live Thai FDA registry connection or marketplace integration. Registration outcomes are determined by fictional fixtures.
- No real product authentication. Matching a registry record or hashing a document is not physical-product verification.
- No universal legal determination. Rule coverage is limited and must be checked against the current applicable Thai advertising requirements before real use.
- No measured accuracy, enforcement reduction, or conversion lift on live listings. Synthetic tests validate implementation behavior, not real-world regulatory performance.
- No production-grade multi-tenant identity or authorization. The hosted demo uses shared HTTP Basic access; marketplace OAuth and distinct reviewer identities are future work.
- Hosted SQLite storage is ephemeral. Persistence across separate serverless instances is not guaranteed.
API quick start
Python 3.9-compatible commands are provided in the repo's development instructions. From the repository root:

Windows PowerShell
python -m venv .venv
.venv/Scripts/python -m pip install -r requirements.txt
.venv/Scripts/python -m uvicorn backend.app.main:app --port 8000
macOS / Linux
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m uvicorn backend.app.main:app --port 8000
Open http://localhost:8000/docs. Example submission:
curl -X POST http://localhost:8000/v1/verifications \
  -H 'Content-Type: application/json' \
  -d '{
    "registration_reference": "DEMO-001",
    "product_name": "Demo Supplement",
    "license_holder": "Demo Brand",
    "category": "supplement"
  }'
The response includes a verification ID, status, rule results, chain_hash, and an audit trace. Use GET /v1/verifications/{id} for the stored result. The unprotected local API entry point above is intended for development, not internet exposure.

To run the password-protected dashboard locally, set a private TRUSTCART_DEMO_PASSWORD environment variable and start app:app instead:
$env:TRUSTCART_DEMO_PASSWORD = "<your-local-demo-password>"
.venv/Scripts/python -m uvicorn app:app --port 8000
The demo login uses username demo. Do not commit the password or expose this development setup as a production service.
API endpoints
Method	Endpoint	Purpose
POST	/v1/verifications	Submit listing; receive a saved decision
GET	/v1/verifications/{id}	Retrieve current decision and trace
POST	/v1/verifications/{id}/evidence	Upload synthetic label/certificate and declared identity fields
POST	/v1/verifications/{id}/recheck	Re-evaluate with the latest evidence
GET	/v1/review-queue	Prioritized human-review cases
POST	/v1/verifications/{id}/review	Record human reviewer action
GET	/v1/reports/{id}	Detailed finding, evidence, and audit report
GET	/v1/audit/{id}	Validate audit-chain integrity
GET	/v1/widgets/{id}	Demo shopper-facing trust-status payload
GET	/v1/agency/summary	Aggregated synthetic metrics without seller identities
POST	/v1/listings/suggest-copy	Deterministic factual-copy template for further review
POST	/v1/products/compare	Compare saved synthetic reports
GET	/v1/demo/scenarios	40 tagged sample listings (hosted demo app)


Use the generated /docs page for the current request and response schemas. Review/evidence operations require the latest chain_hash. 409 errors indicate stale hashes, invalid audit state, or unmet review requirements.
Two-minute judge demo
1. Submit a suspicious listing. In the Seller view, load scenario L1: the displayed product identity conflicts with the fictional registry and the title contains a concerning claim. The API routes the case to PENDING instead of approving it.
2. Request and review evidence. Upload the synthetic label and certificate, then recheck. The identity conflict remains unresolved and requires a human reviewer.
3. Complete a decision. In Reviewer, inspect the risk-ordered queue, provide a reason, and deny the unresolved case. Open the report and audit trace.
4. Demonstrate reliability. Simulate a registry outage: cases become UNKNOWN, not silently approved. If time permits, show the explicit demo tamper action and the failed audit check.
5. Show APIaaS. Open /docs and demonstrate the same workflow through HTTP endpoints. The dashboard and API are two surfaces over the same core decision engine.
Demo success criteria: an end-to-end recorded decision; every finding traceable to the submitted data or synthetic fixture; no approval during an unavailable registry; no fabricated registration facts; and an audit check that detects deliberate tampering. These are functional demonstrations, not evidence of production accuracy.
Business model and validation plan
B2B APIaaS (proposed): A marketplace or seller platform could pay per verification or under an enterprise contract. B2B workflow SaaS (proposed): Brands/agencies with frequent reviews could pay by user seat or review volume. Shoppers benefit from transparency but are not the assumed subscription payer.

A credible commercial case needs a pilot with an authorized data source and real operations teams:
Hypothesis to test	Measure
Current verification is fragmented or costly	Baseline time per review; number of systems consulted; evidence follow-ups
TrustCart reduces repetitive work	Change in time-to-decision and reviewer workload versus current process
TrustCart does not sacrifice safety for speed	Critical-issue recall, false approvals, false positives, escalation rate
Evidence history is operationally useful	Share of decisions with complete source/rule/reviewer trace
Organizations will pay	Signed pilot, actual budget owner, paid conversion, retention


Market sizing discipline: the total value of Thailand's supplement sales is not TrustCart's addressable software market. A bottom-up estimate should use eligible paying organizations × validated annual contract value, with an explicit market-access assumption. No TAM, revenue, ROI, or customer conversion figure is claimed as validated here.
Why not just use the Thai FDA search, generic AI, or marketplace filters?
- Official lookup: authoritative product information is vital, but product identity, listing claims, missing evidence, and a reviewer decision still need to be connected in a workflow.
- Generic AI: useful for interpreting text, but TrustCart keeps the decision behind explicit rules, exposes uncertainty, and records the actions taken.
- Marketplace tooling: large platforms already invest in screening and can build internally. TrustCart's hypothesized value is a reusable, auditable service that can fit smaller platforms, seller operations, or cross-channel workflows. Integration and ROI must be proven.
Roadmap
1. Evidence and quality: expert-reviewed Thai rule pack; current regulatory sources and rule versioning; representative evaluation sets; calibrated risk ranking.
2. Trusted data: authorized Thai FDA registry/permit adapter with field provenance, freshness, source availability, and error handling.
3. Pilot integration: one real marketplace or seller-operations workflow with consent, proper API authentication, human-review permissions, durable storage, and privacy/security controls.
4. Expansion: additional health-product categories and jurisdictions only after local rules, data access, and product-market fit are validated.
Responsible AI and provenance
TrustCart is built around explicit uncertainty, not a promise of autonomous certification. The default demo is deterministic; optional LLM claim extraction is source-grounded, gated by external-transfer approval, and never allowed to change status directly. Reviewers can inspect rule outcomes, hashes, and decision history.

The team states that the hackathon repository was started on 21 November 2026; the current README records first commit 1d8b004 at 10:44 Thailand time. It also identifies an earlier personal prototype (zhunbeiboshi-del/TrustCart-BlueScreen) and pre-event planning documents. The team should verify and disclose the provenance of any code reused from that prototype rather than treating all preparation as newly built competition work. Codex-assisted work is documented via AGENTS.md, tests, and [codex]-marked commits; verify the final commit/test counts on the submission revision.
References
- Thai FDA — misleading health-product advertising and complaints (27 January 2026)
- Thai FDA Journal — survey of unlawful online health-product advertisements (2024)
- Thai FDA — platform screening and enforcement update (2026)
- Thai FDA Food Division — advertising criteria (2026)
- [Product brief](PRODUCT.md) · [Implementation details](IMPLEMENTATION.md) · [Demo guide](DEMO.md) · [Developer/agent instructions](AGENTS.md)
