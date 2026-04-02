# Domain Concepts — MemberCheck API v2

> MemberCheck is a compliance and Anti-Money Laundering (AML) screening platform that provides PEP (Politically Exposed Persons) & Sanctions screening, ID Verification, Know Your Business (KYB), risk assessment, and ongoing monitoring capabilities for individuals and corporate entities. The API (OpenAPI 3.0, base URL `https://demo.api.membercheck.com/api/v2`) exposes these services to integrating systems. This document captures the object model, domain terms, operations, and enumerations of the MemberCheck API.

---

## Authentication

| Scheme | Type | Description |
|--------|------|-------------|
| **Bearer** | HTTP Bearer Token | JWT/token-based authentication passed in the `Authorization` header |
| **ApiKey** | API Key | Key-based authentication for programmatic access |

All endpoints require one of the above authentication schemes. A multi-org header `X-Request-OrgId` is optionally provided to scope requests to a specific organisation.

---

## Terms

### Screening Domain

#### Member (Individual)
- **Type:** Entity
- **Definition:** A natural person subject to compliance screening. Members are identified by name, date of birth, ID number, nationality, and other personal attributes.
- **Key Attributes:** `firstName`, `middleName`, `lastName`, `scriptNameFullName`, `dob`, `idNumber`, `clientId`, `memberNumber` (deprecated → use `clientId`), `address`, `country`, `nationality`, `gender`, `emailAddress`
- **Relationships:** Scanned via Member Scans; may appear in Monitoring Lists; may have Risk Assessments, ID Verifications, and Supporting Documents.
- **Aliases:** Individual, Person, Client

#### Corporate (Company)
- **Type:** Entity
- **Definition:** A legal entity (company/organisation) subject to compliance screening. Identified by company name, registration number, and jurisdiction.
- **Key Attributes:** `companyName`, `clientId`, `entityNumber` (deprecated → use `clientId`), `registrationNumber`, `country`
- **Relationships:** Scanned via Corporate Scans; may appear in Corporate Monitoring Lists; may have KYB checks and Risk Assessments.
- **Aliases:** Company, Organisation (in screening context), Entity

#### Scan
- **Type:** Aggregate
- **Definition:** A compliance check operation that screens a member or corporate entity against selected watchlists. A scan produces zero or more matched entities.
- **Key Attributes:** `scanId` (int32), `scanType`, `scanService`, `matchType`, `date`, `matches`, `category`, `decisions`
- **Relationships:** Contains ScanResult with MatchedEntities; belongs to an Organisation; performed by a User.

#### Scan Result
- **Type:** Value Object
- **Definition:** The outcome of a scan, containing the number of matches and a list of matched watchlist entities.
- **Key Attributes:** `scanId`, `resultUrl`, `matchedNumber`, `matchedEntities[]`, `webSearchResults[]`, `advancedMediaResults[]`, `fatfJurisdictionRiskResults[]`
- **Relationships:** Belongs to a Scan; contains ScanEntity items.

#### Matched Entity (ScanEntity)
- **Type:** Entity
- **Definition:** A watchlist record that matched against a scanned member or corporate during screening. Each match has a result ID, match score, and category.
- **Key Attributes:** `resultId` (int32), `matchedEntity` (name), `matchRate`, `category`, `description`, `decision`, `decisionDetail`
- **Relationships:** Belongs to a ScanResult; can have Decisions recorded; links to an Entity profile.

#### Entity (Person Profile)
- **Type:** Aggregate
- **Definition:** The full watchlist profile of a matched individual, containing all available personal information from the data sources.
- **Key Attributes:** `uniqueId`, `category`, `gender`, `deceased`, `lastReviewed`
- **Nested Collections:** `descriptions[]`, `nameDetails[]`, `roles[]`, `importantDates[]`, `nationalities[]`, `locations[]`, `countries[]`, `officialLists[]`, `idNumbers[]`, `sources[]`, `images[]`, `linkedIndividuals[]`, `linkedCompanies[]`, `profilesOfInterest[]`
- **Relationships:** Referenced by ScanEntity via `resultId`.

#### EntityCorp (Company Profile)
- **Type:** Aggregate
- **Definition:** The full watchlist profile of a matched corporate entity.
- **Key Attributes:** `uniqueId`, `category`, `lastReviewed`
- **Nested Collections:** `descriptions[]`, `nameDetails[]`, `countries[]`, `officialLists[]`, `identifiers[]`, `sources[]`, `linkedIndividuals[]`, `linkedCompanies[]`
- **Relationships:** Referenced by CorpScanEntity via `resultId`.

#### Decision (Due Diligence)
- **Type:** Value Object
- **Definition:** A compliance officer's judgment on a matched entity — classifying whether the match is true/false and the associated risk level.
- **Key Attributes:** `matchDecision` (Match | NoMatch | NotSure), `risk` (Low | Med | High | Unallocated), `comment`, `username`, `date`
- **Relationships:** Applied to a Matched Entity (ScanEntity); tracked as DecisionHistory.

### Batch Scanning

#### Batch Scan
- **Type:** Aggregate
- **Definition:** A bulk screening operation where a file of member or corporate records is uploaded and scanned in one operation.
- **Key Attributes:** `batchScanId` (int32), `date`, `fileName`, `membersScanned`, `matchedMembers`, `numberOfMatches`, `status`, `statusDescription`
- **Status Values:** Uploaded, Completed, Completed with errors, In Progress, Error, Cancelled
- **Relationships:** Contains multiple individual scans; references an Organisation.

#### Batch Scan Status
- **Type:** Value Object
- **Definition:** Progress and status tracking for an in-progress batch scan.
- **Key Attributes:** `batchScanId`, `membersScanned`, `matchedMembers`, `numberOfMatches`, `progress`, `status`, `statusDescription`

### Monitoring

#### Monitoring List
- **Type:** Entity
- **Definition:** A register of members or corporates that are actively monitored on an ongoing basis. When watchlist data changes, monitoring scans automatically detect new matches, updated entities, or removed matches.
- **Key Attributes (Member Item):** `id`, `scanId`, `firstName`, `middleName`, `lastName`, `clientId`, `status` (On/Off), `lastMonitored`, `monitoringDate`
- **Relationships:** Members are added via scan enable; monitored at configured intervals.

#### Monitoring Scan
- **Type:** Event
- **Definition:** An automated periodic scan that checks all monitoring list members against updated watchlists.
- **Key Attributes:** `monitoringScanId`, `date`, `scanType`, `totalMembersMonitored`, `membersChecked`, `newMatches`, `updatedEntities`, `removedMatches`, `status`, `reviewStatus`, `membersReviewed`
- **Relationships:** Produces MonitoringScanResults with entities; linked to a Monitoring List.

### Risk Assessment

#### Risk Assessment
- **Type:** Aggregate
- **Definition:** A structured questionnaire-based risk evaluation of a member or corporate entity, scoring risk across multiple categories.
- **Key Attributes (Input):** `firstName`, `middleName`, `lastName`, `clientId`, `nationalityCode`, `domicileCountryCode`, `professionId`, `productId`, `deliveryChannelId`, `residentStatusId`, `isPEP`, `isSanctioned`, `hasAdverseMedia`, plus category-specific answer IDs
- **Key Attributes (Result):** `scanId`, `riskLevel`, `riskScore`, `riskAssessmentItems[]`
- **Relationships:** Associated with a member scan; has supporting documents.

#### Risk Assessment Detail
- **Type:** Value Object
- **Definition:** A single question in the risk assessment questionnaire with available options.
- **Key Attributes:** `id`, `category`, `question`, `options[]` (value, label), `controlType` (RadioButton | Dropdown | Checkbox), `isRequired`

### ID Verification

#### ID Verification
- **Type:** Aggregate
- **Definition:** An identity verification check for an individual, optionally combining document verification (DVS) and biometric face matching.
- **Key Attributes (Input):** `firstName`, `middleName`, `lastName`, `dob`, `mobileNumber`, `emailAddress`, `country`, `verificationProcessType`, `driverLicence`, `passport`, `medicare`, `consent`
- **Key Attributes (Result):** `scanId`, `idvStatus`, `idvFaceMatchStatus`, `signatureKey`, `idvUrl`, `driverLicenceResult`, `passportResult`, `medicareResult`, `facematchResult`, `facematchOCRData`
- **IDV Statuses:** NotVerified, Verified, Pass, PartialPass, Fail, Pending, Incomplete, NotRequested, Matched, NotMatched, PartialMatched, InvalidDocument, Error
- **FaceMatch Statuses:** Pass, Review, Fail, Pending, Incomplete, NotRequested

### Know Your Business (KYB)

#### KYB Scan
- **Type:** Aggregate
- **Definition:** A company search and verification operation that retrieves company profile data, UBO (Ultimate Beneficial Ownership) information, and official documents from business registries.
- **Key Attributes (Input):** `companyName`, `countryCode`, `stateCode`, `registrationNumber`
- **Key Attributes (Result):** `scanId`, `companies[]` (CompanyResult)
- **Relationships:** Contains products (document orders); linked to corporate scans.

#### Company Result (KYB)
- **Type:** Value Object
- **Definition:** A company profile found during a KYB search.
- **Key Attributes:** `companyCode`, `companyNumber`, `companyName`, `date`, `status`, `address`, `state`

#### KYB Product Order
- **Type:** Entity
- **Definition:** An ordered document product (company profile, UBO declaration, etc.) from a business registry.
- **Key Attributes:** `productId`, `status` (Requested | Pending | Completed | Failed | Cancelled), `price`

### Supporting Documents

#### Supporting Document
- **Type:** Entity
- **Definition:** An uploaded file associated with a scan or verification for audit trail purposes.
- **Key Attributes:** `id`/`supportingDocumentId`, `fileName`, `comment`, `documentType`, `createdDate`, `createdBy`, `isPinned`, `presignedUrl`
- **Supported File Types:** PDF, JPG, JPEG, PNG, GIF, TIF, TIFF, ZIP
- **Relationships:** Belongs to a specific scan (member, corporate, or IDV).

### AI Analysis

#### AI Analysis
- **Type:** Value Object
- **Definition:** An AI-powered Q&A capability that answers compliance questions about matched entities.
- **Key Attributes (Input):** `question` (required), `helperText` (optional context)
- **Key Attributes (Result):** `id`, `scanResultId`, `question`, `answer`, `isStriked`
- **Relationships:** Associated with a matched entity via `scanResultId`.

### Organisation & User Management

#### Organisation
- **Type:** Aggregate
- **Definition:** A tenant/account in MemberCheck representing a compliance team or business unit.
- **Key Attributes:** `id`, `name`, `status` (Inactive | Active | Deleted), `country`, `email`, `creationDate`
- **Settings:** `memberScanSettings`, `corporateScanSettings`, `monitoringSettings`, `idvSettings`, `kybSettings`, `subscriptionSettings`, `agreementSettings`, `listAccessSettings`
- **Relationships:** Has Users; owns Scans, Monitoring Lists, and configurations.

#### User
- **Type:** Entity
- **Definition:** A person with access to MemberCheck, with assigned roles, organisations, and access rights.
- **Key Attributes:** `id`, `username`, `firstName`, `lastName`, `email`, `role`, `status` (Inactive | Active | Deleted | Locked | Pending), `apiKey`, `mfaType`, `assignedOrganisations[]`, `accessRights[]`
- **Relationships:** Belongs to one or more Organisations; performs Scans.

### Reference Data

#### Watchlist
- **Type:** Concept
- **Definition:** A curated data source of sanctioned individuals, PEPs, and other persons/entities of interest. MemberCheck aggregates multiple data providers.
- **Data Sources:** DowJones, ThomsonReuters, MemberCheck, Acuris, LexisNexis, CustomList

#### FATF Jurisdiction Risk
- **Type:** Value Object
- **Definition:** Risk rating based on FATF (Financial Action Task Force) assessment of a country's compliance with AML/CFT standards.
- **Key Attributes:** `countryName`, `countryCode`, `isBlackList`, `isGreyList`, `comment`, `url`

#### Sanctioned Country
- **Type:** Value Object
- **Definition:** A country identified as sanctioned by international bodies.
- **Key Attributes:** `countryName`, `countryCode`, `isBlackList`, `isGreyList`, `comment`, `url`

#### Tax Haven Country
- **Type:** Value Object
- **Definition:** A country identified as a tax haven based on primary location and nationalities of a matched profile.

---

## Enumerations

### Scan Enumerations

| Enum | Values | Description |
|------|--------|-------------|
| **MatchType** | `Close`, `Exact`, `ExactMidName` | How closely a watchlist name must match before being considered |
| **ScanType** | `Single`, `Batch`, `Automatic`, `MonitoringRescan` | The type of scan operation |
| **ScanService (Member)** | `PepAndSanction`, `IDVerification`, `RiskAssessment` | Service type for individual scans |
| **ScanService (Corp)** | `PepAndSanction`, `KYB`, `RiskAssessment` | Service type for corporate scans |
| **WhitelistPolicy** | `Apply`, `Ignore` | Whether to apply the whitelist filter |
| **ScanResult** | `NoMatchesFound`, `MatchesFound` | Outcome classification of a scan |
| **ReportFormat** | `PDF`, `Word`, `Excel`, `CSV` | Downloadable report file formats |

### Category Enumerations

| Enum | Values | Description |
|------|--------|-------------|
| **Category (Member)** | `TER`, `PEP`, `SIP`, `RCA`, `POI` | Watchlist category for individuals |
| **Category (Corp)** | `TER`, `SIE`, `POI`, `SOE` | Watchlist category for corporates |
| **SubCategory (SIP/SIE)** | `SanctionsLists`, `LawEnforcement`, `RegulatoryEnforcement`, `OrganisedCrime`, `FinancialCrime`, `NarcoticsCrime`, `ModernSlavery`, `BriberyAndCorruption`, `CyberCrime`, `DisqualifiedDirectors`, `ReputationalRisk`, `Other`, `Insolvency`, `CustomWatchlist`, `WarCrimes`, `EndUseControl`, `EnvironmentalCrime`, `Fugitive`, `Gambling`, `HumanRightsViolation`, `InterstateCommerceViolation`, `LabourViolation`, `PharmaTrafficking`, `Piracy`, `UnauthorisedIncident`, `FormerSanctions` | Fine-grained sub-classification |

### Category Definitions

| Code | Full Name | Description |
|------|-----------|-------------|
| **TER** | Terrorist | Individuals/entities on terrorism-related watchlists |
| **PEP** | Politically Exposed Person | Individuals holding prominent public positions |
| **SIP** | Special Interest Person | Individuals flagged across regulatory/law enforcement subcategories |
| **RCA** | Relative or Close Associate | Persons closely associated with PEPs |
| **POI** | Profile of Interest | Individuals flagged for general compliance interest |
| **SIE** | Special Interest Entity | Corporate equivalent of SIP |
| **SOE** | State-Owned Enterprise | Companies owned or controlled by a state |

### Decision & Risk Enumerations

| Enum | Values | Description |
|------|--------|-------------|
| **MatchDecision** | `Match`, `NoMatch`, `NotSure` | Due diligence finding on a matched entity |
| **Risk** | `Unallocated`, `Low`, `Med`, `High` | Assessed risk level |
| **AMLRiskLevel** | `None`, `Low`, `Medium`, `High`, `Sanctions` | Risk level from risk assessment |
| **MonitoringStatus** | `NewMatches`, `UpdatedMatches`, `RemovedMatches`, `NoChanges` | Type of monitoring change detected |

### Verification Enumerations

| Enum | Values | Description |
|------|--------|-------------|
| **IDVStatus** | `NotVerified`, `Verified`, `Pass`, `PartialPass`, `Fail`, `Pending`, `Incomplete`, `NotRequested`, `Matched`, `NotMatched`, `PartialMatched`, `InvalidDocument`, `Error` | ID verification result status |
| **FaceMatchStatus** | `Pass`, `Review`, `Fail`, `Pending`, `Incomplete`, `NotRequested` | Biometric face matching result status |
| **KYBProductStatus** | `Requested`, `Pending`, `Completed`, `Failed`, `Cancelled` | Document product order status |

### System Enumerations

| Enum | Values | Description |
|------|--------|-------------|
| **UserStatus** | `Inactive`, `Active`, `Deleted`, `Locked`, `Pending` | User account status |
| **OrgStatus** | `Inactive`, `Active`, `Deleted` | Organisation status |
| **MonitoringInterval** | `Daily`, `Weekly`, `Fortnightly`, `Monthly`, `Quarterly`, `SemiAnnual` | Frequency of automated monitoring scans |
| **MfaType** | `Disabled`, `Email`, `VirtualMfaDevice` | Multi-factor authentication type |
| **DataSource** | `None`, `DowJones`, `ThomsonReuters`, `MemberCheck`, `Acuris`, `LexisNexis`, `CustomList` | Watchlist data provider |
| **NotificationService** | `None`, `Slack`, `Mattermost` | Webhook notification integration |

---

## Operations

### Member Scans — Single

#### Perform Member Single Scan
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single` |
| **Input** | `ScanInputParam` — firstName, middleName, lastName, scriptNameFullName, dob, dobTolerance, idNumber, address, country[], nationality[], gender, matchType, closeMatchRateThreshold, whitelistPolicy, residencePolicy, blankAddressPolicy, pepJurisdiction, excludeDeceasedPersons, includeResultEntities, updateMonitoringList, includeWebSearch, includeAdvancedMedia, includeJurisdictionRisk, watchlists[], includeRiskAssessment, ignoreBlankPolicy, clientId, dataBreachCheckParam, idvParam |
| **Output** | `201`: ScanResult (scanId, resultUrl, matchedNumber, matchedEntities[]) |
| **Side Effects** | Optionally adds member to Monitoring List; optionally triggers IDV or Risk Assessment |

#### Get Member Scan History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single` |
| **Input** | Query filters: userId, from, to, firstName, middleName, lastName, scriptNameFullName, clientId, idNumber, scanType, scanService, matchType, whitelistPolicy, includeWebSearch, includeAdvancedMedia, scanResult, category, subCategory, decision, risk, faceMatchVerification, pageIndex, pageSize |
| **Output** | `200`: ScanHistoryLog[] |

#### Get Member Scan Detail
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/{id}` |
| **Input** | Path: scanId; Query: fields, category, subCategory |
| **Output** | `200`: ScanHistoryDetail (scanParam, scanResult, decisions) |

#### Rescan Member
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/rescan` |
| **Input** | Path: scanId |
| **Output** | `201`: ScanHistoryDetail |

#### Get Matched Entity Profile
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}` |
| **Input** | Path: resultId |
| **Output** | `200`: SingleScanResultDetail (Entity profile with all watchlist data) |

#### Add Due Diligence Decision
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/results/{id}/decisions` |
| **Input** | Path: resultId; Body: DecisionParam (matchDecision, risk, comment) |
| **Output** | `201`: DecisionResult |

#### Add Bulk Decisions
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/results/decisions` |
| **Input** | Query: ids (comma-separated resultIds); Body: DecisionParam |
| **Output** | `201`: DecisionResult |

#### Get Decision History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/decisions` |
| **Output** | `200`: DecisionHistory[] |

#### Get Category Risk Levels
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/category-risks` |
| **Output** | `200`: RiskResult (category-wise risks and overall risk level) |

#### AI Analysis — Ask Question
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/results/{id}/questions` |
| **Input** | Body: AIAnalysisInputParam (question, helperText) |
| **Output** | `201`: AIAnalysisResultInfo |

#### AI Analysis — Get Answers
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/questions` |
| **Output** | `200`: AIAnalysisResultInfo[] |

#### AI Analysis — Strike/Unstrike
| Element | Detail |
|---------|--------|
| **Trigger** | `PUT /member-scans/single/results/{id}/questions/{questionId}` |
| **Output** | `204`: Success |

#### Get Linked Individual Profile
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/linked-individuals/{individualId}` |
| **Output** | `200`: Entity |

#### Get Linked Individual Risk Levels
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/linked-individuals/risk-levels` |
| **Output** | `200`: LinkedProfiles |

#### Get Linked Company Profile
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/linked-companies/{companyId}` |
| **Output** | `200`: EntityCorp |

#### Get Linked Company Risk Levels
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/results/{id}/linked-companies/risk-levels` |
| **Output** | `200`: CorpLinkedProfiles |

### Member Scans — Batch

#### Perform Member Batch Scan
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/batch` (multipart/form-data) |
| **Input** | BatchScanInputParam (param) + File (binary) |
| **Output** | `201`: BatchScanResult (batchScanId, status) |
| **Error** | `409`: Duplicate file name within 12 months |

#### Get Batch Scan History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/batch` |
| **Output** | `200`: BatchScanHistoryLog[] |

#### Get Batch Scan Detail
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/batch/{id}` |
| **Output** | `200`: BatchScanResults |

#### Get Batch Scan Status
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/batch/{id}/status` |
| **Output** | `200`: BatchScanStatus (progress, membersScanned, status) |

#### Enable Batch Monitoring
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/batch/{id}/monitor/enable` |
| **Output** | `204`: All batch members added to monitoring |

### Member Scans — Monitoring

#### Get Monitoring History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/monitoring` |
| **Output** | `200`: MonitoringScanHistoryLog[] |

#### Get Monitoring Scan Detail
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/monitoring/{id}` |
| **Output** | `200`: MonitoringScanResults |

#### Enable Member Monitoring
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/monitor/enable` |
| **Input** | Path: scanId; Query: allowDuplicate (boolean) |
| **Output** | `204`: Member added to monitoring list |
| **Error** | `409`: Duplicate clientId in monitoring list |

#### Disable Member Monitoring
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/monitor/disable` |
| **Output** | `204`: Member removed from active monitoring |

#### Set Monitoring Review Status
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/monitor/review` |
| **Input** | Query: status (boolean) |
| **Output** | `204`: Review status updated |

### Member Scans — Supporting Documents

#### Get Supporting Documents
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/{id}/documents` |
| **Output** | `200`: SupportingDocumentResult[] |

#### Upload Supporting Documents
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/documents` (multipart) |
| **Input** | Documents[] (files with comments and types), IsOverwrite (boolean) |
| **Output** | `201`: Documents uploaded |

#### Pin/Unpin Document
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/single/{id}/documents/{documentId}/pin` |
| **Output** | `204`: Pin status toggled |

#### Download Document
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/{id}/documents/{documentId}/download` |
| **Output** | `200`: File content |

#### Delete Document
| Element | Detail |
|---------|--------|
| **Trigger** | `DELETE /member-scans/single/{id}/documents/{documentId}` |
| **Output** | `204`: Document deleted |

#### Get Document History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /member-scans/single/{id}/documents/history` |
| **Output** | `200`: SupportingDocumentHistoryResult[] |

### Member Scans — Advanced Media

#### Toggle Bookmark
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /member-scans/advanced-media/bookmark` |
| **Input** | AdvancedMediaBookmarkParam (Id, ScanInputId, ArticleId, SiteId) |
| **Output** | `204`: Bookmark toggled |

### Member Scans — Reports

| Endpoint | Description | Formats |
|----------|-------------|---------|
| `GET /member-scans/single/report` | Scan history report | PDF, Excel, Word, CSV |
| `GET /member-scans/single/{id}/report` | Full scan profile report | PDF, Excel, Word |
| `GET /member-scans/single/{id}/no-results-report` | No-match scan report | PDF, Excel, Word |
| `GET /member-scans/single/results/{id}/report` | Entity profile report | PDF, Excel, Word |
| `GET /member-scans/single/results/{id}/compromised-report` | Data breach report | PDF, Excel, Word |
| `GET /member-scans/single/results/{id}/linked-individuals/{id}/report` | Linked individual report | PDF, Excel, Word |
| `GET /member-scans/single/results/{id}/linked-companies/{id}/report` | Linked company report | PDF, Excel, Word |
| `GET /member-scans/batch/report` | Batch history report | PDF, Excel, Word |
| `GET /member-scans/batch/{id}/report` | Batch scan detail report | PDF, Excel, Word |
| `GET /member-scans/batch/{id}/exception-report` | Batch exception report | CSV |
| `GET /member-scans/batch/{id}/full-report` | Batch full report | CSV |
| `GET /member-scans/monitoring/report` | Monitoring history report | PDF, Excel, Word |
| `GET /member-scans/monitoring/{id}/report` | Monitoring scan detail report | PDF, Excel, Word |

> Reports support multi-language via `X-Request-Language` header: EN, FR, ZH, AR, JA, ES.

### Corporate Scans — Single

#### Perform Corporate Single Scan
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /corp-scans/single` |
| **Input** | `CorpScanInputParam` — companyName, clientId, registrationNumber, country[], matchType, closeMatchRateThreshold, whitelistPolicy, includeWebSearch, includeAdvancedMedia, includeJurisdictionRisk, watchlists[], includeRiskAssessment, ignoreBlankPolicy |
| **Output** | `201`: CorpScanResult (scanId, matchedEntities[]) |

#### Get Corporate Scan History
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /corp-scans/single` |
| **Output** | `200`: CorpScanHistoryLog[] |

#### Get Corporate Scan Detail
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /corp-scans/single/{id}` |
| **Output** | `200`: CorpScanHistoryDetail |

#### Get Corporate Matched Entity Profile
| Element | Detail |
|---------|--------|
| **Trigger** | `GET /corp-scans/single/results/{id}` |
| **Output** | `200`: SingleScanCorpResultDetail (EntityCorp profile) |

#### Add Corporate Decision
| Element | Detail |
|---------|--------|
| **Trigger** | `POST /corp-scans/single/results/{id}/decisions` |
| **Input** | DecisionParam |
| **Output** | `201`: DecisionResult |

### Corporate Scans — Batch

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/corp-scans/batch` | `POST` | Upload and scan corporate batch file |
| `/corp-scans/batch` | `GET` | Get corporate batch scan history |
| `/corp-scans/batch/{id}` | `GET` | Get specific batch scan detail |

### Corporate Scans — Monitoring

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/corp-scans/monitoring` | `GET` | Get corporate monitoring history |
| `/corp-scans/monitoring/{id}` | `GET` | Get specific monitoring scan detail |

### Risk Assessment — Member

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/aml-risk/member-scans` | `GET` | Get risk assessment questionnaire (questions & options) |
| `/aml-risk/member-scans` | `POST` | Perform member risk assessment |
| `/aml-risk/member-scans/{scanId}` | `POST` | Recheck/update member risk assessment |
| `/aml-risk/member-scans/{scanId}` | `GET` | Get risk assessment detail |
| `/aml-risk/member-scans/{scanId}/report` | `GET` | Download risk assessment report |

### Risk Assessment — Corporate

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/aml-risk/corp-scans` | `GET` | Get corporate risk assessment questionnaire |
| `/aml-risk/corp-scans` | `POST` | Perform corporate risk assessment |
| `/aml-risk/corp-scans/{scanId}` | `POST` | Recheck/update corporate risk assessment |
| `/aml-risk/corp-scans/{scanId}` | `GET` | Get corporate risk assessment detail |
| `/aml-risk/corp-scans/{scanId}/report` | `GET` | Download corporate risk assessment report |

### ID Verification

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/id-verification/{scanId}` | `GET` | Get ID verification detail |
| `/id-verification/single` | `POST` | Perform new ID verification |
| `/id-verification/single/{scanId}/documents` | `GET` | Get IDV supporting documents |
| `/id-verification/single/{scanId}/documents` | `POST` | Upload IDV supporting documents |
| `/id-verification/single/{scanId}/documents/{documentId}/pin` | `POST` | Pin/unpin IDV document |
| `/id-verification/single/{scanId}/documents/{documentId}/download` | `GET` | Download IDV document |
| `/id-verification/single/{scanId}/documents/{documentId}` | `DELETE` | Delete IDV document |
| `/id-verification/single/{scanId}/documents/history` | `GET` | Get IDV document history |

### Know Your Business (KYB)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/kyb/countries` | `GET` | Get supported KYB countries and jurisdictions |
| `/kyb` | `POST` | Perform KYB company search |
| `/kyb/{scanId}` | `GET` | Get KYB scan detail |
| `/kyb/{scanId}/products` | `POST` | Order a document product |
| `/kyb/{scanId}/products` | `GET` | Get product order history |
| `/kyb/{scanId}/products/{productId}` | `GET` | Get specific document product |
| `/kyb/{scanId}/report` | `GET` | Download KYB scan report |
| `/kyb/{scanId}/documents` | `GET` | Get KYB supporting documents |
| `/kyb/jurisdictions` | `GET` | Get KYB jurisdictions list |

### User Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/users` | `GET` | List all users |
| `/users` | `POST` | Create new user |
| `/users/{id}` | `GET` | Get user by ID |
| `/users/{id}` | `PUT` | Update user |
| `/users/{id}` | `DELETE` | Delete user |
| `/users/myprofile` | `GET` | Get current user profile |
| `/users/myprofile/security` | `PUT` | Update password/MFA settings |
| `/users/myprofile/reset-mfa` | `POST` | Initiate MFA token reset |
| `/users/{id}/reset-api-key` | `POST` | Reset user API key |

### Organisation Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/organisations` | `GET` | List all organisations |
| `/organisations/{id}` | `GET` | Get organisation detail |

### Data Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/data-management/member-scans` | `GET` | Get member scan data for management |
| `/data-management/member-batch-scans` | `GET` | Get member batch data for management |
| `/data-management/corp-scans` | `GET` | Get corporate scan data for management |
| `/data-management/corp-batch-scans` | `GET` | Get corporate batch data for management |
| `/data-management/single-scans` | `DELETE` | Delete selected single scans |
| `/data-management/batch-scans` | `DELETE` | Delete selected batch scans |
| `/data-management/member-documents` | `GET` | Get member supporting documents |
| `/data-management/corp-documents` | `GET` | Get corporate supporting documents |

### Monitoring Lists

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/monitoring-lists/member` | `GET` | List members in monitoring list |
| `/monitoring-lists/member/report` | `GET` | Download monitoring list CSV report |
| `/monitoring-lists/member/{id}/enable` | `POST` | Activate monitoring for member |
| `/monitoring-lists/member/{id}/disable` | `POST` | Deactivate monitoring for member |
| `/monitoring-lists/member/{id}` | `DELETE` | Remove member from monitoring list |
| `/monitoring-lists/members` | `DELETE` | Bulk remove members from monitoring list |

### Lookup Values (Reference Data)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/lookup-values/system-settings` | `GET` | Get system settings |
| `/lookup-values/time-zones` | `GET` | Get supported time zones |
| `/lookup-values/document-types` | `GET` | Get supporting document types |

### Reports

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/reports/member-due-diligence` | `GET` | Member due diligence report (PDF, Excel, Word, CSV) |
| `/reports/corp-due-diligence` | `GET` | Corporate due diligence report (PDF, Excel, Word, CSV) |

---

## Core Object Model (Schema Relationships)

```
Organisation (1)
├── Users (*)
├── Member Scans
│   ├── Single Scan (*)
│   │   ├── ScanInputParam
│   │   ├── ScanResult
│   │   │   ├── Metadata
│   │   │   ├── ScanEntity / MatchedEntity (*)
│   │   │   │   ├── Entity (Person Profile)
│   │   │   │   │   ├── Description (*)
│   │   │   │   │   ├── NameDetail (*)
│   │   │   │   │   ├── Role (*)
│   │   │   │   │   ├── Date (*)
│   │   │   │   │   ├── Country (*)
│   │   │   │   │   ├── Location (*)
│   │   │   │   │   ├── OfficialList (*)
│   │   │   │   │   ├── IDNumber (*)
│   │   │   │   │   ├── Source (*)
│   │   │   │   │   ├── LinkedIndividual (*)
│   │   │   │   │   ├── LinkedCompany (*)
│   │   │   │   │   └── ProfileOfInterest (*)
│   │   │   │   ├── DecisionHistory (*)
│   │   │   │   ├── CategoryRisk (*)
│   │   │   │   └── AIAnalysisResultInfo (*)
│   │   │   ├── WebSearchResult (*)
│   │   │   ├── AdvancedMediaResult (*)
│   │   │   ├── FATFJurisdictionRiskInfo (*)
│   │   │   ├── SanctionedCountryResult (*)
│   │   │   ├── TaxHavenCountryResult (*)
│   │   │   └── DataBreachCheckResult (*)
│   │   ├── SupportingDocument (*)
│   │   └── IDVerification (0..1)
│   │       ├── IDVInputParam
│   │       ├── IDVResult (DVS + FaceMatch)
│   │       └── SupportingDocument (*)
│   ├── Batch Scan (*)
│   │   ├── BatchScanInputParam
│   │   ├── BatchScanResult
│   │   ├── BatchScanStatus
│   │   └── Member Scan (*) [individual scans within batch]
│   └── Monitoring
│       ├── MonitoringList
│       │   └── MonitoringListMemberItem (*)
│       └── MonitoringScan (*)
│           └── MonitoringScanResults
├── Corporate Scans
│   ├── Single Scan (*)
│   │   ├── CorpScanInputParam
│   │   ├── CorpScanResult
│   │   │   ├── CorpScanEntity (*)
│   │   │   │   └── EntityCorp (Company Profile)
│   │   │   │       ├── Description (*)
│   │   │   │       ├── NameDetail (*)
│   │   │   │       ├── Country (*)
│   │   │   │       ├── OfficialList (*)
│   │   │   │       ├── Identifier (*)
│   │   │   │       ├── Source (*)
│   │   │   │       ├── LinkedIndividual (*)
│   │   │   │       └── LinkedCompany (*)
│   │   │   ├── WebSearchResult (*)
│   │   │   └── AdvancedMediaResult (*)
│   │   └── SupportingDocument (*)
│   ├── Batch Scan (*)
│   └── Monitoring
│       └── CorpMonitoringScan (*)
├── Risk Assessment
│   ├── Member Risk Assessment (*)
│   │   ├── RiskAssessmentInputParam
│   │   ├── RiskAssessmentResult
│   │   └── RiskAssessmentItem (*)
│   └── Corporate Risk Assessment (*)
│       ├── CorpRiskAssessmentInputParam
│       └── CorpRiskAssessmentResult
└── KYB (Know Your Business)
    ├── KYBScan (*)
    │   ├── CompanyResult (*)
    │   └── KYBProductOrder (*)
    │       └── KYBProductHistoryResult
    └── KYBCountryResult (*)
        └── KYBStateResult (*)
```

---

## Scan Input Parameters — Detailed Field Tables

### ScanInputParam (Member Single Scan)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `firstName` | string (max 100) | No | Member's first name (Latin/Roman scripts) |
| `middleName` | string (max 100) | No | Member's middle name (Latin/Roman scripts) |
| `lastName` | string (max 100) | No | Member's last name (Latin/Roman scripts) |
| `scriptNameFullName` | string (max 200) | No | Full name in original script (Chinese, Japanese, Cyrillic, Arabic, etc.) |
| `dob` | string | No | Date of birth (DD/MM/YYYY or YYYY) |
| `dobTolerance` | int32 (0–9) | No | ± years tolerance for DOB matching |
| `idNumber` | string (max 100) | No | National ID, passport, registration number. Used in matching — only watchlist records with matching ID returned |
| `address` | string (max 255) | No | Member address (not used for matching, used for residence policy) |
| `country` | string[] (max 5) | No | Country of residence (ISO 3166-1 alpha-2) |
| `nationality` | string[] (max 5) | No | Nationality (ISO 3166-1 alpha-2) |
| `gender` | string | No | Member's gender |
| `matchType` | enum | No | Close, Exact, ExactMidName |
| `closeMatchRateThreshold` | int32 (1–100) | No | Threshold for close matching (default: 80) |
| `whitelistPolicy` | enum | No | Apply or Ignore whitelist |
| `residencePolicy` | enum | No | Ignore, ApplyPEP, ApplySIP, ApplyALL |
| `blankAddressPolicy` | enum | No | Apply or Ignore blank addresses |
| `pepJurisdiction` | enum | No | Apply or Ignore PEP jurisdiction |
| `excludeDeceasedPersons` | enum | No | No or Yes |
| `includeResultEntities` | enum | No | Yes or No (include full entity profiles in response) |
| `updateMonitoringList` | enum | No | ForceUpdate, No, Yes |
| `includeWebSearch` | enum | No | No or Yes (Google adverse media search) |
| `includeAdvancedMedia` | enum | No | No or Yes |
| `includeJurisdictionRisk` | enum | No | No or Yes (FATF jurisdiction risk) |
| `includeRiskAssessment` | enum | No | No or Yes |
| `watchlists` | string[] (max 6) | No | PEP, POI, RCA, SIP, Official Lists, Custom Watchlists |
| `ignoreBlankPolicy` | enum | No | DOB, Gender, IDNumber, Nationality |
| `clientId` | string (max 100) | No | Customer reference / account ID |
| `emailAddress` | string (max 128) | No | Email for data breach checks |
| `dataBreachCheckParam` | object | No | Email-based data breach check parameters |
| `idvParam` | object | No | ID verification parameters (mobile, country) |

### CorpScanInputParam (Corporate Single Scan)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `companyName` | string (max 255) | Yes | Company name to screen |
| `clientId` | string | No | Client reference ID |
| `registrationNumber` | string | No | Company registration number |
| `country` | string[] (max 5) | No | Country of registration (ISO 3166-1 alpha-2) |
| `matchType` | enum | No | Close, Exact |
| `closeMatchRateThreshold` | int32 (1–100) | No | Close match threshold (default: 80) |
| `whitelistPolicy` | enum | No | Apply or Ignore |
| `includeWebSearch` | enum | No | No or Yes |
| `includeAdvancedMedia` | enum | No | No or Yes |
| `includeJurisdictionRisk` | enum | No | No or Yes |
| `watchlists` | string[] | No | POI, SIE, Official Lists, SOE, Custom Watchlists |
| `includeRiskAssessment` | enum | No | No or Yes |
| `ignoreBlankPolicy` | enum | No | RegistrationNumber |

---

## Constraints

### Scan Constraints

| Constraint | Scope | Condition | Violation |
|------------|-------|-----------|-----------|
| Name required | Member Scan | At least `firstName`+`lastName` or `scriptNameFullName` must be provided | 400 Bad Request |
| Company name required | Corp Scan | `companyName` is mandatory | 400 Bad Request |
| Max 5 countries | ScanInputParam | `country[]` max 5 items | 400 Bad Request |
| Max 5 nationalities | ScanInputParam | `nationality[]` max 5 items | 400 Bad Request |
| Max 6 watchlists | ScanInputParam | `watchlists[]` max 6 items | 400 Bad Request |
| DOB tolerance range | ScanInputParam | `dobTolerance` must be 0–9 | 400 Bad Request |
| Close match threshold | ScanInputParam | `closeMatchRateThreshold` must be 1–100 | 400 Bad Request |
| Batch duplicate | Batch Scan | Same file name within 12 months | 409 Conflict |
| Monitoring duplicate | Monitor Enable | Same `clientId` already in monitoring list | 409 Conflict (unless `allowDuplicate=true` or `ForceUpdate`) |
| Authentication required | All endpoints | Valid Bearer token or API key | 401 Unauthorized |
| Organisation access | All endpoints | User must have access to specified org | 403 Forbidden |

### Report Constraints

| Constraint | Scope | Condition |
|------------|-------|-----------|
| CSV unlimited / others capped | Scan History Report | CSV returns all records; PDF/Excel/Word limited to 10,000 |
| Date format | All date filters | Must match `DD/MM/YYYY` pattern |

---

## Processes

### Individual PEP & Sanctions Screening Flow

**Goal:** Screen a member against watchlists and record compliance decisions.

```mermaid
flowchart TD
    A[Submit Member Scan] -->|POST /member-scans/single| B{Matches Found?}
    B -->|No| C[No Matches - Scan Complete]
    B -->|Yes| D[Review Matched Entities]
    D --> E[Get Entity Profile]
    E -->|GET /member-scans/single/results/{id}| F[Review Profile Details]
    F --> G{AI Analysis Needed?}
    G -->|Yes| H[Ask AI Question]
    H -->|POST .../questions| F
    G -->|No| I[Record Decision]
    I -->|POST .../decisions| J{Add to Monitoring?}
    J -->|Yes| K[Enable Monitoring]
    K -->|POST .../monitor/enable| L[Ongoing Monitoring]
    J -->|No| M[Scan Complete]
    L --> N[Automated Rescan at Interval]
    N -->|GET /member-scans/monitoring| O{Changes Detected?}
    O -->|Yes| P[Review Updates]
    P --> I
    O -->|No| N
```

### Corporate KYB + Screening Flow

**Goal:** Verify a company's identity and screen against watchlists.

```mermaid
flowchart TD
    A[Search Company] -->|POST /kyb| B[Review Company Results]
    B --> C[Order Document Products]
    C -->|POST /kyb/{scanId}/products| D[Wait for Product Completion]
    D -->|GET .../products/{productId}| E{Product Ready?}
    E -->|No| D
    E -->|Yes| F[Review Company Profile & UBO]
    F --> G[Perform Corporate Scan]
    G -->|POST /corp-scans/single| H{Matches Found?}
    H -->|No| I[Clear - No Matches]
    H -->|Yes| J[Review & Decide]
    J -->|POST .../decisions| K[Record Decision]
```

### Batch Processing Flow

**Goal:** Screen a large batch of members in a single operation.

```mermaid
flowchart TD
    A[Upload Batch File] -->|POST /member-scans/batch| B[Batch Processing Starts]
    B --> C[Poll Status]
    C -->|GET .../batch/{id}/status| D{Status?}
    D -->|In Progress| C
    D -->|Completed| E[Review Results]
    D -->|Error| F[Download Exception Report]
    E -->|GET .../batch/{id}| G[Review Matched Members]
    G --> H{Enable Monitoring?}
    H -->|Yes| I[Enable Batch Monitoring]
    I -->|POST .../batch/{id}/monitor/enable| J[All Members Monitored]
    H -->|No| K[Download Report]
    F -->|GET .../exception-report| L[Fix and Re-upload]
```

---

## API Summary by Tag

| Tag | Endpoints | Purpose |
|-----|-----------|---------|
| **member-scans** | ~30+ | Individual PEP/Sanctions screening — single, batch, monitoring, decisions, documents, reports, AI analysis |
| **corp-scans** | ~20+ | Corporate PEP/Sanctions screening — single, batch, monitoring, decisions, documents, reports |
| **member-risk-check** | 5 | Individual AML risk assessment questionnaire and scoring |
| **corp-risk-check** | 5 | Corporate AML risk assessment questionnaire and scoring |
| **id-verification** | 8 | Identity document verification and biometric face matching |
| **business-ubo-check** | 8 | Know Your Business — company search, document products, UBO |
| **users** | 8 | User CRUD, profile management, API key reset, MFA |
| **organisations** | 2 | Organisation listing and detail |
| **account** | 2 | Current user profile (alias for myprofile endpoints) |
| **data-management** | 8 | Administrative data management — view and delete scan data |
| **monitoring-lists** | 6 | Monitoring list management — enable, disable, delete members |
| **lookup-values** | 3 | Reference data — system settings, time zones, document types |
| **reports** | 2 | Due diligence reporting — member and corporate |
