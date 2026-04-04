
# Summary

The OpenSubject is an open source intelligent platform, which aggregates and analyze the notices and bullets by governments, organizations, foundations etc. The platform should be capable to digest the messages/notices, deep research on an sourcing subject, score/register/link subject, and allow other system to consume the information.

---

## Business Scenarios

### BS-1: Ingest Government Notices and Bulletins

**Problem:** Law enforcement agencies, compliance teams, and security organisations receive threat notices, sanctions updates, and bulletins from multiple government sources (OFAC, UN, EU, Interpol, FBI, local agencies) in inconsistent formats — PDFs, emails, XML feeds, web pages. Manually triaging and extracting actionable subject data is slow and error-prone.

**Actors:** System Administrator, Notice Ingestion Service, External Government Sources

**Flow:**

1. OpenSubject connects to configured government and organisation feeds (RSS, API, email inbox, file drop).
2. The ingestion service receives a new notice (e.g., an OFAC SDN update, an Interpol Red Notice, a local police bulletin).
3. The platform uses **Exa AI** (`POST /contents`) to extract clean structured text from the notice URL or document, handling PDFs and JS-rendered pages automatically.
4. An NLP pipeline parses the extracted content to identify subject entities (individuals, organisations), associated attributes (names, aliases, dates of birth, nationalities, ID numbers), and the notice category (sanctions, terrorism, PEP, law enforcement alert).
5. Extracted subjects are registered in the OpenSubject registry with a unique Subject ID, source reference, and ingestion timestamp.
6. The system publishes an event (`notice.ingested`) for downstream processing.

**Outcome:** Notices from disparate sources are normalised into a unified subject registry with traceable provenance.

**Integrated Resources:**

| Resource | Usage |
|----------|-------|
| Exa AI — Content Extraction | `POST /contents` with `text` + `highlights` to extract notice content from URLs and documents |
| Exa AI — Subpage Crawling | `subpages` + `subpageTarget` to follow linked annexes and supplementary pages |

---

### BS-2: Deep Research on a Sourcing Subject

**Problem:** When a new subject is ingested or flagged, analysts need comprehensive background information — web presence, social media profiles, news articles, published connections, and corporate affiliations — gathered from open sources. Manual research is time-consuming and inconsistent.

**Actors:** Analyst, OpenSubject Research Service, Exa AI

**Flow:**

1. A subject is flagged for deep research (triggered by ingestion, analyst request, or a risk threshold).
2. OpenSubject composes a natural-language research query from the subject's known attributes (name, aliases, nationality, date of birth).
3. The platform calls **Exa AI** (`POST /search`, `type: "deep"`) with an `outputSchema` defining the desired profile structure (known associates, corporate roles, news mentions, social profiles).
4. For people-specific searches, the platform uses `category: "people"` to search 1B+ people records; for corporate subjects, `category: "company"` to search 50M+ company pages.
5. The platform calls **Exa AI** (`POST /search`, `category: "news"`) with date filters to gather recent media coverage.
6. Results are aggregated, deduplicated, and stored as research findings linked to the Subject ID.
7. Source URLs, publication dates, and confidence scores are preserved for audit trail.
8. If the research requires deeper investigation, the platform creates an **Exa AI** Research Task (`POST /research/v1`) for comprehensive multi-source synthesis.

**Outcome:** A comprehensive, cited dossier is built for each subject with minimal analyst effort.

**Integrated Resources:**

| Resource | Usage |
|----------|-------|
| Exa AI — Deep Search | `POST /search` with `type: "deep"`, `outputSchema` for structured subject profiles |
| Exa AI — Category Search | `category: "people"` or `"company"` for targeted entity discovery |
| Exa AI — News Search | `category: "news"` with `startPublishedDate` / `endPublishedDate` for recent coverage |
| Exa AI — Research Task | `POST /research/v1` for comprehensive async research with `exa-research-pro` |
| Exa AI — Find Similar | `POST /findSimilar` to discover related pages from a known subject URL |

---

### BS-3: Screen Subject Against Sanctions and Watchlists

**Problem:** Every subject in the registry must be checked against global sanctions lists, PEP databases, and adverse media watchlists. Regulatory compliance requires documented screening with auditable decisions.

**Actors:** Compliance Officer, OpenSubject Screening Service, MemberCheck API

**Flow:**

1. When a subject is registered or updated, the screening service triggers an AML/sanctions check.
2. For **individual subjects**: the platform calls **MemberCheck** (`POST /member-scans/single`) with the subject's name, date of birth, nationality, and ID numbers to scan against watchlists (TER, PEP, SIP, RCA, POI).
3. For **corporate subjects**: the platform calls **MemberCheck** (`POST /corp-scans/single`) to scan against corporate watchlists (TER, SIE, POI, SOE).
4. The platform retrieves matched entity profiles (`GET /member-scans/single/results/{id}`) including names, roles, dates, countries, official lists, relatives, and associates.
5. The platform retrieves category risk levels (`GET /member-scans/single/results/{id}/category-risks`) and linked entities to assess the full risk picture.
6. If matches are found, the subject's risk score is updated and an analyst review is triggered.
7. The platform records the due diligence decision (`POST /member-scans/single/results/{id}/decisions`) with match/no-match/not-sure and risk level.
8. Supporting documents and reports are generated (`GET /reports/member-due-diligence`) for compliance records.

**Outcome:** Every subject is screened against global watchlists with auditable decisions and risk scores.

**Integrated Resources:**

| Resource | Usage |
|----------|-------|
| MemberCheck — Individual Screening | `POST /member-scans/single` for PEP, sanctions, and watchlist checks |
| MemberCheck — Corporate Screening | `POST /corp-scans/single` for corporate entity screening |
| MemberCheck — Risk Assessment | `GET /member-scans/single/results/{id}/category-risks` for risk scoring |
| MemberCheck — AI Analysis | `GET /member-scans/single/results/{id}/questions` for AI-powered Q&A on matched profiles |
| MemberCheck — Due Diligence Reports | `GET /reports/member-due-diligence` for downloadable compliance reports |
| MemberCheck — Decisions | `POST /member-scans/single/results/{id}/decisions` for recording match decisions |

---

### BS-4: Facial Recognition and Identity Verification

**Problem:** Text-based screening alone cannot confirm a subject's identity when dealing with aliases, common names, or deliberately falsified identity documents. Agencies need to match a subject's photo against known-image databases.

**Actors:** Investigator, OpenSubject Identity Service, Clearview AI

**Flow:**

1. An analyst uploads a probe image (photo, still from video, or image URL) for a subject under investigation.
2. OpenSubject uploads the image to **Clearview AI** (`POST /api/v1/search_uploads`) for face detection.
3. Detected faces with bounding boxes and FQA (Face Quality Assessment) scores are returned. The analyst selects the target face.
4. The platform submits a facial search (`POST /api/v1/searches`) with the selected `face_id` against Clearview's 70B+ image database.
5. Search results return matched images with source web links, profile data (name, handle, bio), and similarity scores.
6. For sensitive operations, the platform uses **Secure Search** (`POST /api/v1/secure_searches`) to perform an ephemeral search without creating a persistent history record.
7. Matched results are linked back to the Subject ID in the OpenSubject registry, contributing to the subject's identity confidence score.
8. If the subject warrants ongoing monitoring, the analyst adds the image to a **Gallery** (`POST /api/v1/gallery_items`) with tags for automated alert generation when new matches appear.

**Outcome:** Subjects are verified or identified via facial recognition, resolving ambiguities that text-based screening cannot address.

**Integrated Resources:**

| Resource | Usage |
|----------|-------|
| Clearview AI — Image Upload | `POST /api/v1/search_uploads` for probe image upload and face detection |
| Clearview AI — Face Search | `POST /api/v1/searches` against 70B+ image database |
| Clearview AI — Secure Search | `POST /api/v1/secure_searches` for ephemeral, non-persistent searches |
| Clearview AI — Gallery Management | `POST /api/v1/gallery_items` for ongoing monitoring watchlists |
| Clearview AI — Investigations | `POST /api/v1/investigations` for case management and outcome tracking |

---

### BS-5: Continuous Monitoring and Alerting

**Problem:** Threat landscapes and sanctions lists change daily. A subject cleared today may appear on a watchlist tomorrow. Agencies need automated, ongoing monitoring that detects changes without manual re-screening.

**Actors:** Monitoring Service, Compliance Officer, Analyst

**Flow:**

1. When a subject is registered, the platform adds the individual to the **MemberCheck Monitoring List** (`POST /monitoring-lists/member`) for periodic automated rescanning.
2. MemberCheck runs scheduled scans against updated watchlists (DowJones, ThomsonReuters, LexisNexis, etc.) and detects new matches, updated matches, and removed matches.
3. OpenSubject retrieves monitoring results (`GET /member-scans/monitoring/{id}`) and compares against previous scan state.
4. For image-based monitoring, the platform maintains subjects in **Clearview AI Galleries** (`POST /api/v1/gallery_items`) with alerts enabled (`alerts: true` on search history items). When new matches appear in Clearview's crawled data, the platform is notified.
5. For open-source intelligence (OSINT) monitoring, the platform runs periodic **Exa AI** searches (`POST /search`, `type: "auto"`, `startPublishedDate` set to last-check date) to detect new news, articles, or web presence changes.
6. All detected changes generate internal alerts with severity levels.
7. High-severity alerts (new sanctions match, new criminal association) are escalated to assigned analysts for immediate review.
8. Alert resolution decisions are recorded with timestamps and justifications.

**Outcome:** Subjects are continuously monitored across watchlists, image databases, and open sources with automated change detection.

**Integrated Resources:**

| Resource | Usage |
|----------|-------|
| MemberCheck — Monitoring Lists | `POST /monitoring-lists/member` for ongoing watchlist rescanning |
| MemberCheck — Monitoring Results | `GET /member-scans/monitoring/{id}` for change detection |
| Clearview AI — Gallery Alerts | Gallery items with alerts for new facial match notifications |
| Exa AI — Periodic Search | `POST /search` with date filters for OSINT change detection |

---

### BS-6: Subject Scoring and Risk Registration

**Problem:** Each subject in the system has data from multiple sources — watchlist hits, research findings, facial matches, media coverage — but there is no unified risk score. Analysts need a composite score that reflects the overall threat level.

**Actors:** Scoring Engine, Analyst, Downstream Consumer Systems

**Flow:**

1. The scoring engine collects all evidence vectors for a subject:
   - **Watchlist score** — number and category of MemberCheck matches (TER hits weighted highest, PEP/SIP/RCA progressively lower).
   - **Identity confidence** — Clearview AI match similarity scores and number of confirmed visual matches.
   - **OSINT exposure** — Exa AI research findings: volume of adverse media, recency of coverage, source credibility (e.g., government sites weighted higher than blogs).
   - **Network risk** — linked individuals and companies from MemberCheck (`linked-individuals`, `linked-companies`) and Exa research (associates, corporate affiliations).
2. The engine applies a weighted scoring algorithm to produce a composite Subject Risk Score (0–100).
3. Subjects are classified into risk tiers: **Critical** (80–100), **High** (60–79), **Medium** (30–59), **Low** (0–29).
4. The score, tier, and contributing factors are persisted on the Subject record.
5. Score changes trigger re-evaluation of monitoring frequency and alert severity thresholds.
6. The Subject Risk Score is exposed via the OpenSubject API for downstream systems to consume.

**Outcome:** Every subject carries a composite, explainable risk score derived from multiple intelligence sources.

---

### BS-7: Cross-System Subject Linking

**Problem:** The same real-world person or organisation may appear across multiple sources under different names, aliases, or identifiers. Without linking, analysts investigate fragmented records and miss connections.

**Actors:** Entity Resolution Service, Analyst

**Flow:**

1. When a new subject is registered, the entity resolution service searches for potential duplicates using:
   - **Name/alias matching** — fuzzy text matching against existing subjects.
   - **Biometric matching** — Clearview AI facial search against the internal gallery of known subjects.
   - **Identifier matching** — exact match on passport numbers, national IDs, tax IDs from MemberCheck screening results.
   - **Network overlap** — shared associates or corporate affiliations discovered via Exa AI research.
2. Candidate matches above a configurable threshold are surfaced for analyst review.
3. The analyst confirms or rejects the link. Confirmed links merge subject records under a master Subject ID while preserving all source records.
4. Linked subjects inherit the highest risk score and combined evidence from all constituent records.
5. Links are bidirectional and auditable — any downstream consumer can trace the link chain back to source evidence.

**Outcome:** Fragmented intelligence is unified into consolidated subject profiles. Related individuals and organisations are explicitly linked.

---

### BS-8: External System Integration (API Consumption)

**Problem:** OpenSubject does not operate in isolation. Law enforcement databases, compliance platforms, case management systems, and intelligence sharing networks need to consume subject data programmatically.

**Actors:** External Consumer System, OpenSubject API Gateway

**Flow:**

1. External systems authenticate via API key or OAuth 2.0 token.
2. Systems query the OpenSubject API for subjects by:
   - Name, alias, or identifier search.
   - Risk tier filter (e.g., only Critical and High).
   - Category filter (e.g., only TER/sanctions subjects).
   - Date range (subjects added or updated since last sync).
   - Geographic filter (nationality, jurisdiction).
3. The API returns subject records including: identity attributes, risk score, screening results, research summaries, linked subjects, source provenance, and last-updated timestamps.
4. For real-time integration, external systems subscribe to webhooks for events: `subject.created`, `subject.updated`, `subject.risk_changed`, `alert.triggered`.
5. Bulk export is available for data warehouse and analytics pipelines.
6. All API access is logged with caller identity, requested data, and timestamps for compliance audit.

**Outcome:** Subject intelligence is consumable by any authorised system via a clean REST API and event-driven webhooks.

---

## Scenario Coverage Matrix

| Scenario | MemberCheck | Clearview AI | Exa AI | Core Platform |
|----------|:-----------:|:------------:|:------:|:-------------:|
| BS-1: Ingest Notices | | | X | X |
| BS-2: Deep Research | | | X | X |
| BS-3: Watchlist Screening | X | | | X |
| BS-4: Facial Recognition | | X | | X |
| BS-5: Continuous Monitoring | X | X | X | X |
| BS-6: Subject Scoring | X | X | X | X |
| BS-7: Subject Linking | X | X | X | X |
| BS-8: External API | | | | X |

