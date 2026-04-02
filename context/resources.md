# Resources

## MemberCheck — AML/Compliance Screening Platform

| Property | Value |
|----------|-------|
| **Provider** | MemberCheck (KYC / AML compliance SaaS) |
| **API Version** | 2.0 (OpenAPI 3.0) |
| **Base URL** | `https://demo.api.membercheck.com/api/v2` |
| **Swagger UI** | `https://demo.api.membercheck.com/swagger` |
| **OpenAPI Spec** | `https://demo.api.membercheck.com/swagger/v2/swagger.json` |
| **Authentication** | Bearer Token or API Key |
| **Multi-tenancy** | `X-Request-OrgId` header scopes requests to a specific organisation |

### Capabilities

| Capability | API Tags | Description |
|------------|----------|-------------|
| **Individual PEP & Sanctions Screening** | `member-scans` | Single, batch, and monitoring scans of individuals against watchlists (TER, PEP, SIP, RCA, POI) |
| **Corporate PEP & Sanctions Screening** | `corp-scans` | Single, batch, and monitoring scans of companies against watchlists (TER, SIE, POI, SOE) |
| **Individual Risk Assessment** | `member-risk-check` | Questionnaire-based AML risk scoring for individuals |
| **Corporate Risk Assessment** | `corp-risk-check` | Questionnaire-based AML risk scoring for corporates |
| **ID Verification** | `id-verification` | Document verification (DVS) and biometric face matching |
| **Know Your Business (KYB)** | `business-ubo-check` | Company search, document product orders, UBO verification |
| **Ongoing Monitoring** | `monitoring-lists` | Automated periodic watchlist rescanning with change detection |
| **AI Analysis** | `member-scans` | AI-powered Q&A on matched entity profiles |
| **Due Diligence Decisions** | `member-scans`, `corp-scans` | Record match/no-match/not-sure decisions with risk levels |
| **Supporting Documents** | `member-scans`, `corp-scans`, `id-verification` | Upload, pin, download, delete compliance documents |
| **Reporting** | `reports` | Due diligence reports in PDF, Excel, Word, CSV |
| **User Management** | `users` | User CRUD, role assignment, MFA, API key management |
| **Organisation Management** | `organisations` | Multi-org tenant configuration and settings |
| **Data Management** | `data-management` | Administrative view and deletion of scan data |
| **Reference Data** | `lookup-values` | System settings, time zones, document types |

### Data Retrievable via API

| Data | Endpoint | Description |
|------|----------|-------------|
| Member scan history | `GET /member-scans/single` | Full history of individual screening scans with filters |
| Member scan detail | `GET /member-scans/single/{id}` | Scan parameters, matched entities, and decisions for a specific scan |
| Matched entity profile | `GET /member-scans/single/results/{id}` | Full watchlist profile: names, roles, dates, countries, official lists, ID numbers, sources, images, relatives, associates |
| Decision history | `GET /member-scans/single/results/{id}/decisions` | Due diligence decision audit trail per matched entity |
| Category risk levels | `GET /member-scans/single/results/{id}/category-risks` | Category-wise and overall risk assessment |
| Linked individuals | `GET /member-scans/single/results/{id}/linked-individuals/risk-levels` | Linked person profiles with suggested risks |
| Linked companies | `GET /member-scans/single/results/{id}/linked-companies/risk-levels` | Linked corporate profiles with suggested risks |
| AI analysis Q&A | `GET /member-scans/single/results/{id}/questions` | AI-generated answers about matched entities |
| Batch scan history | `GET /member-scans/batch` | History of batch file uploads and results |
| Batch scan detail | `GET /member-scans/batch/{id}` | Matched members within a batch scan |
| Batch scan status | `GET /member-scans/batch/{id}/status` | Progress tracking for in-flight batch scans |
| Monitoring history | `GET /member-scans/monitoring` | History of automated monitoring scan runs |
| Monitoring scan detail | `GET /member-scans/monitoring/{id}` | New, updated, and removed matches per monitoring run |
| Monitoring list members | `GET /monitoring-lists/member` | All members currently in the monitoring list with status |
| Corporate scan history | `GET /corp-scans/single` | History of corporate screening scans |
| Corporate scan detail | `GET /corp-scans/single/{id}` | Corporate scan matches and parameters |
| Corporate entity profile | `GET /corp-scans/single/results/{id}` | Full corporate watchlist profile |
| Corporate monitoring | `GET /corp-scans/monitoring` | Corporate monitoring scan history |
| Risk assessment questions | `GET /aml-risk/member-scans` | Risk questionnaire with categories, questions, and options |
| Risk assessment detail | `GET /aml-risk/member-scans/{scanId}` | Results, scores, and risk levels for an individual |
| Corporate risk detail | `GET /aml-risk/corp-scans/{scanId}` | Results and scores for a corporate entity |
| ID verification detail | `GET /id-verification/{scanId}` | IDV status, DVS results, face match results, OCR data |
| IDV documents | `GET /id-verification/single/{scanId}/documents` | Supporting documents for ID verification scans |
| KYB countries | `GET /kyb/countries` | Supported countries and jurisdictions for KYB checks |
| KYB scan detail | `GET /kyb/{scanId}` | Company search results and matched companies |
| KYB product history | `GET /kyb/{scanId}/products` | Ordered document products and their statuses |
| Supporting documents | `GET /member-scans/single/{id}/documents` | Uploaded compliance documents per scan |
| Document history | `GET /member-scans/single/{id}/documents/history` | Audit trail of document uploads, downloads, deletes |
| Users | `GET /users` | All users with roles, organisations, and status |
| User detail | `GET /users/{id}` | User profile, access rights, assigned organisations |
| Organisations | `GET /organisations` | All organisations in the tenant |
| Organisation detail | `GET /organisations/{id}` | Organisation settings: scan, monitoring, IDV, KYB, list access |
| System settings | `GET /lookup-values/system-settings` | Platform configuration values |
| Time zones | `GET /lookup-values/time-zones` | Available time zones |
| Document types | `GET /lookup-values/document-types` | Supporting document type reference list |
| Member due diligence report | `GET /reports/member-due-diligence` | Downloadable member due diligence report |
| Corporate due diligence report | `GET /reports/corp-due-diligence` | Downloadable corporate due diligence report |

### Watchlist Categories

| Code | Name | Applies To |
|------|------|------------|
| TER | Terrorist | Individuals & Corporates |
| PEP | Politically Exposed Person | Individuals |
| SIP | Special Interest Person | Individuals |
| RCA | Relative or Close Associate | Individuals |
| POI | Profile of Interest | Individuals & Corporates |
| SIE | Special Interest Entity | Corporates |
| SOE | State-Owned Enterprise | Corporates |

### Data Sources

DowJones, ThomsonReuters, MemberCheck, Acuris, LexisNexis, CustomList

### Local Artefacts

| Artefact | Path | Description |
|----------|------|-------------|
| API Object Model | `context/capacity/membercheck.md` | Domain concepts, terms, operations, enumerations, schema relationships |
| OpenAPI Spec (YAML) | `spec/membercheck.yml` | Full OpenAPI 3.0 specification (21,000+ lines) |

---

## Clearview AI — Facial Recognition Investigative Platform

| Property | Value |
|----------|-------|
| **Provider** | Clearview AI, Inc. |
| **API Version** | 1.0.0 |
| **Base URL** | `https://app.clearview.ai/api/v1` |
| **Web Application** | `https://app.clearview.ai/app/` |
| **Website** | `https://www.clearview.ai` |
| **Authentication** | Bearer Token (API Key) — `Authorization: Bearer <your_api_key>` |
| **Pagination** | Cursor-based — `next_page_token` / `page_token` / `max_page_size` / `total_items` |
| **Audience** | Law enforcement, government agencies, military, national security (not available to private sector) |
| **Database** | 70 billion+ facial images from publicly available internet sources |
| **Accuracy** | 99%+ across all demographics (NIST FRVT tested) |
| **Certifications** | SOC 2 Type II, TX-RAMP |
| **Source Documentation** | `context/API Docs _ Clearview AI.pdf` (129 pages) |

### Capabilities

| Capability | API Tags | Description |
|------------|----------|-------------|
| **Facial Recognition Search** | `searches` | Search a probe image (file, URL, face_id, or search_history_id) against 70B+ images; supports fast and exhaustive modes |
| **Secure (Ephemeral) Search** | `secure_searches` | Search without creating a persistent search history item; optional `custom_log_id` for audit trail |
| **Search Source Control** | `searches` | Filter searches to main index, private galleries, collections, or federated search connections; include/exclude by ID or type |
| **Search Result Curation** | `search_results` | Mark results as read, curate (thumbs up/down/neutral), view similarity scores and source metadata |
| **Search History** | `search_history` | Browse, filter, and manage past search records with alerting and deconfliction |
| **Image Upload & Face Detection** | `search_uploads` | Upload probe images, detect faces with bounding boxes and FQA scores, select specific faces for search |
| **Bulk Image Upload** | `search_uploads` | Upload multiple files or URLs at once via background job |
| **Face Extraction** | `faces` | Extract face rectangles from any blob (upload, search result, gallery image) |
| **Investigation Management** | `investigations` | Create, list, update, close, and delete investigations with enrollment forms, categories, and ownership policies |
| **Investigation Feedback** | `investigations` | Record investigation outcomes: led to identification, location, associate identification, expanded knowledge |
| **Peer Review** | `investigations` | Peer review workflow for search results before investigative action |
| **Deconfliction** | `searches` | Detect when multiple users are researching the same subject |
| **Gallery (Watchlist) Management** | `galleries` | Create, list, update, and delete private image galleries for organisation-specific subjects |
| **Gallery Item Management** | `gallery_items` | Add, update, tag, and remove individuals from galleries; bulk import via background job |
| **Gallery Photo Management** | `gallery_photos` | Upload photos to gallery items with optional geolocation |
| **Gallery Tagging** | `gallery_tags` | Create and manage colour-coded tags for gallery item categorisation |
| **Gallery Coalitions** | `galleries` | Share galleries across organisations via coalition partnerships |
| **User Administration** | `users` | List, view, update, suspend users; manage roles (0,1,4,5), 2FA, notifications, feature flags |
| **Organisation Data Export** | `company_jobs` | Trigger full organisation data export |
| **SSO Integration** | `authentication` | SAML 2.0 and OAuth 2.0 single sign-on configuration |

### Data Retrievable via API

| Data | Endpoint | Description |
|------|----------|-------------|
| Authentication status | `GET /auth` | Current user session, user profile, company details, feature flags, licence info |
| Search results (list) | `GET /api/v1/search_results` | Paged list of search results filtered by organisation, user group, user, or search_history_id |
| Search result detail | `GET /api/v1/search_results/{search_result_id}` | Full result with image data, link data, profile data, match scores, gallery data, connection data |
| Search history (list) | `GET /api/v1/search_history` | Paged list of past searches with match counts, alert timestamps, investigation context |
| Search history detail | `GET /api/v1/search_history/{search_history_id}` | Full search history item including probe face, peer review, guest shares, deconfliction status |
| Search uploads (list) | `GET /api/v1/search_uploads` | Paged list of uploaded probe images with face detection results |
| Search upload detail | `GET /api/v1/search_uploads/{blob_id}` | Upload blob with EXIF data, dimensions, detected faces and bounding boxes |
| Face rectangles | `GET /api/v1/faces` | Detected face bounding boxes from any blob (upload, result, gallery) |
| Investigations (list) | `GET /api/v1/investigations` | Paged list of investigations with filters: creator, owner, status, success, enrollment, keyword, date range |
| Investigation detail | `GET /api/v1/investigations/{investigation_id}` | Full investigation: title, status, category, creator, search history items, enrollment form, owner, policies, feedback |
| Galleries (list) | `GET /api/v1/galleries` | Paged list of private galleries with item and photo counts |
| Gallery detail | `GET /api/v1/galleries/{gallery_id}` | Gallery metadata, schemas, coalition info |
| Gallery items (list) | `GET /api/v1/gallery_items` | Paged gallery items filtered by gallery_id or tag_id, with photos, tags, and schema values |
| Gallery item detail | `GET /api/v1/gallery_items/{gallery_item_id}` | Full gallery item with photos, tags, custom field values, schema |
| Gallery tags (list) | `GET /api/v1/gallery_tags` | Paged list of colour-coded tags used for gallery item categorisation |
| Users (list) | `GET /api/v1/users` | Paged list of users filtered by organisation, group, search term, active status; CSV export |
| User detail | `GET /api/v1/users/{user_id}` | Full user profile, roles, 2FA, feature flags, notification preferences; use "me" for current user |
| Secure search logs | `GET /api/v1/secure_searches` | Retrieve metadata for ephemeral searches logged with `custom_log_id` |

### Search Result Data Structure

Each search result includes the following nested data:

| Component | Fields | Description |
|-----------|--------|-------------|
| **ImageData** | width, height, original\_image\_url, image\_url, ts, md5 | Matched image metadata and URLs |
| **LinkData** | link, raw\_id, title, description, link\_type, alt\_text | Source web page where the image was found |
| **ProfileData** | name, bio, handle, link | Social/web profile information |
| **MatchData** | sim, group, group\_sim, dist, rect, fqa\_score, num\_faces, face\_id, face\_type | Similarity scores and face geometry |
| **GalleryData** | gallery\_id, gallery\_item\_id, gallery\_name, is\_coalition, tags, properties | Private gallery match context |
| **SearchConnectionData** | outbound\_search\_connection\_id, name, description | Federated search partner context |

### Security & Compliance

| Aspect | Detail |
|--------|--------|
| Transport | TLS 1.2+ for all API traffic |
| Encryption at rest | All stored data encrypted |
| Authentication | API Key (Bearer Token), 2FA (email, SMS, TOTP) |
| SSO | SAML 2.0, OAuth 2.0 |
| Audit | SOC 2 Type II certified |
| Certification | TX-RAMP (Texas Risk and Authorization Management Program) |
| Penetration testing | Regular third-party testing |
| Bug bounty | Active program |
| Jurisdiction restrictions | Not available in EU, UK, Australia, Canada; restricted in VT, NJ, IL, ME, MA, MT |
| Patents | US 11,250,266 · US 11,443,553 · US 11,694,477 · US 12,315,220 B2 |

### Local Artefacts

| Artefact | Path | Description |
|----------|------|-------------|
| API Documentation (PDF) | `context/API Docs _ Clearview AI.pdf` | Official API docs, 129 pages — endpoints, schemas, workflows |
| Domain Model | `context/capacity/clearview.md` | Domain concepts, terms, operations, processes, constraints |
| OpenAPI Spec (YAML) | `spec/clearview.yml` | Reconstructed OpenAPI 3.0 specification from PDF documentation |

---

## Exa AI — AI-Native Web Search & Content Extraction Platform

| Property | Value |
|----------|-------|
| **Provider** | Exa AI, Inc. (San Francisco) |
| **API Version** | 1.0.0 |
| **Base URL** | `https://api.exa.ai` |
| **Documentation** | `https://exa.ai/docs/reference/search-api-guide` |
| **Dashboard** | `https://dashboard.exa.ai` |
| **Authentication** | API Key via `x-api-key` header or `Authorization: Bearer <key>` |
| **SDKs** | Python (`exa-py`), JavaScript (`exa-js`) |
| **Index Size** | Billions of web pages; 1B+ people, 50M+ companies, 100M+ research papers |

### Capabilities

| Capability | Endpoint | Description |
|------------|----------|-------------|
| **Web Search** | `POST /search` | Natural language search with 6 type tiers: `auto`, `instant` (~200ms), `fast` (~450ms), `neural` (~1s), `deep` (5–60s), `deep-reasoning` |
| **Category Search** | `POST /search` | Focus on data verticals: `company` (50M+), `people` (1B+), `research paper` (100M+), `news`, `personal site`, `financial report` |
| **Deep Search with Structured Output** | `POST /search` | `type: "deep"` with `outputSchema` returns structured JSON with field-level citations and confidence scores |
| **Content Extraction** | `POST /contents` | Extract clean markdown text, token-efficient highlights, and LLM summaries from any URL |
| **Subpage Crawling** | `POST /contents`, `POST /search` | Automatically discover and extract content from linked pages within a site |
| **Similarity Search** | `POST /findSimilar` | Find web pages similar to a given URL with filtering and content options |
| **Question Answering** | `POST /answer` | LLM-generated answers to questions with source citations; supports streaming and structured output |
| **Asynchronous Research** | `POST /research/v1` | Deep research tasks with 3 model tiers (`exa-research-fast`, `exa-research`, `exa-research-pro`); returns reports with sources |
| **Content Freshness Control** | All content endpoints | `maxAgeHours` controls cache vs. livecrawl: `0` = always fresh, `-1` = cache only, positive = max stale age |
| **Domain/Date/Text Filtering** | `POST /search`, `POST /findSimilar` | `includeDomains`, `excludeDomains` (max 1200), date range, `includeText`, `excludeText` |
| **Content Moderation** | `POST /search`, `POST /findSimilar` | Filter unsafe content from results |

### Data Retrievable via API

| Data | Endpoint | Description |
|------|----------|-------------|
| Web search results | `POST /search` | Title, URL, published date, author, image, favicon for matched pages |
| Full page text | `POST /search` (contents.text), `POST /contents` (text) | Clean markdown text with optional maxCharacters, HTML tags, section filtering |
| Highlights (excerpts) | `POST /search` (contents.highlights), `POST /contents` (highlights) | Token-efficient key excerpts with cosine similarity scores (10x fewer tokens than full text) |
| LLM summaries | `POST /search` (contents.summary), `POST /contents` (summary) | LLM-generated abstracts; supports JSON Schema for structured extraction |
| Subpage content | `POST /search`, `POST /contents` | Nested content from linked pages via `subpages` + `subpageTarget` |
| Extracted links | `POST /search`, `POST /contents` | Hyperlinks and image URLs from pages via `extras.links` / `extras.imageLinks` |
| Deep search synthesis | `POST /search` (type: deep) | Structured or text `output.content` with `output.grounding` (field-level citations + confidence) |
| Similar pages | `POST /findSimilar` | Pages similar to a seed URL with full filtering and content options |
| Grounded answers | `POST /answer` | LLM answers with source citations; optional structured output via `outputSchema` |
| Research reports | `GET /research/v1/{researchId}` | Comprehensive research output with sources, events, and optional structured JSON |
| Research task list | `GET /research/v1` | Paginated list of research tasks with status tracking |
| Request costs | All endpoints | `costDollars` breakdown per request: search, contents, per-component pricing |

### Pricing (Per Request)

| Component | Price |
|-----------|-------|
| Neural search (1–10 results) | $0.007 |
| Neural search (additional result) | $0.001 |
| Deep search | $0.012 |
| Deep-reasoning search | $0.015 |
| Content text (per page) | $0.001 |
| Content highlight (per page) | $0.001 |
| Content summary (per page) | $0.001 |

### Local Artefacts

| Artefact | Path | Description |
|----------|------|-------------|
| Domain Model | `context/capacity/exa-ai.md` | Domain concepts, terms, operations, processes, constraints, applications |
| OpenAPI Spec (YAML) | `spec/exa-ai.yml` | OpenAPI 3.0 specification — 6 endpoints, 23 schemas, 5 tags |
