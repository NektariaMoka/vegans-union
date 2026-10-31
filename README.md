# vegans-union

A worldwide directory connecting vegans, vegetarians, and animal lovers with organizations, volunteering opportunities, and animal-friendly brands and products.

The project also supports optional research for personal analysis and academic publication. It is currently in planning; no application features are implemented.

## MVP

Launch an English-language directory accepting listings worldwide, beginning with a curated collection. Coverage will be shown honestly rather than presented as comprehensive.

- Discover animal rescues, sanctuaries, charities, and relevant farms.
- Find volunteering opportunities with tasks, destinations, dates, requirements, fees, accommodation, meals, and official application links.
- Explore curated brands and products, distinguishing vegan products from entirely vegan brands.
- Search and filter by location, category, and volunteering requirements.
- Suggest listings and brands, and report outdated information through moderated forms.
- Browse without registration and save favorites locally on the device.
- Install the PWA and receive an explicit offline fallback. Live opportunity information requires connectivity.
- Optionally participate in a research questionnaire.
- Provide authenticated administration for moderation, source checking, and controlled research exports.

Applications and donations link to organizations' official websites. Listings include sources, review status, and last-checked dates. Source review does not certify safety or animal-welfare practices. Farm listings describe their activities and animal-use practices.

### Outside the MVP

Chat, visitor accounts, public reviews, direct volunteer applications, payment processing, grant aggregation, automated recommendations, and large-scale scraping are deferred.

## Technology direction

| Layer | Planned choice |
| --- | --- |
| Frontend | Angular, TypeScript, Angular Router, semantic HTML, SCSS |
| Public rendering | Angular server-side rendering |
| PWA | Angular service worker with controlled caching and offline behavior |
| Backend | Java 25 LTS and Spring Boot |
| API | REST with an OpenAPI specification |
| Database | PostgreSQL |
| Persistence | Spring Data JPA; explicit SQL for reporting where appropriate |
| Migrations | Flyway |
| Admin authentication | Spring Security with secure sessions |
| Testing | JUnit, PostgreSQL integration tests through Testcontainers, and Playwright |
| Deployment | Docker Compose, HTTPS reverse proxy, automated off-server backups |
| Research analysis | Documented CSV exports; Python or R outside the production backend |

Use one backend application organized into directory, submissions, research, and administration modules. Business rules and data access belong in the backend. Exact dependency versions and compatibility must be verified against supported stable releases before scaffolding; these choices are a planning baseline, not an installed dependency inventory.

Worldwide data uses stable country identifiers, Unicode names, administrative areas for city disambiguation, and time zones where relevant. The initial interface is English.

## Data collection

### Public directory

- Organizations: name, description, type, website, public contact link, locations, and activities.
- Opportunities: organization, tasks, destinations or remote status, dates, requirements, languages, costs, accommodation, meals, and official application link.
- Brands and products: name, category, website/product link, availability countries, and evidence supporting vegan claims.
- Sources and claims: source URL, publisher, claim supported, review status, and date checked.
- Contributions: suggested content or corrections, supporting links, and moderation outcome.

Claims distinguish manufacturer statements, certification evidence, and unverified information. Unknown values remain distinct from negative answers.

### Optional research questionnaire

The initial study is intended for adults aged 18 or older. Proposed fields must be justified by the study questions before launch:

| Information | Proposed approach |
| --- | --- |
| Eligibility | Confirmation of age eligibility, without date of birth |
| Home location | Optional country and city; no exact address or GPS |
| Desired destinations | Countries, optional cities, and an anywhere option |
| Interests | Volunteer activity categories and an other option |
| Preferences | Remote/in-person, duration, travel willingness, accommodation needs |
| Barriers | Cost, time, travel, accessibility, information, confidence, other |
| Brand discovery | Discovery channels and factors influencing trust |
| Dietary identity | Optional, only if necessary for the research question |
| Feedback | Optional short text with guidance to avoid personal details |
| Study metadata | Questionnaire version, consent version, submission time, recruitment channel |

Do not collect names, email addresses, precise locations, full birth dates, or health information in the initial questionnaire. Future follow-up contact requires a separate workflow and permission.

Browsing and submitting directory suggestions do not enroll someone in research. No signup does not guarantee anonymity: locations, free text, identifiers, and operational logs can still identify people. Avoid fingerprinting and silently inferring home locations.

### Operational records

Maintain administrator audit events, error logs, and limited abuse-prevention information with defined retention. Keep operational records out of research exports and avoid copying questionnaire answers into logs.

## Database baseline

One PostgreSQL instance contains three schemas. Schema separation supports access control but does not itself anonymize data.

### Directory schema

| Tables | Purpose |
| --- | --- |
| `countries`, `cities` | Shared geographic reference data |
| `organizations`, `organization_locations` | Organizations and operating locations |
| `opportunities`, `opportunity_locations` | Opportunities linked to organizations and destinations |
| `opportunity_requirements`, `opportunity_costs` | Conditions, accommodation/meals, amounts, currencies, charging periods |
| `brands`, `products`, `product_countries` | Brands, their products, and country availability |
| `categories` | Controlled categories scoped by listing type |
| `organization_categories`, `opportunity_categories`, `brand_categories`, `product_categories` | Category relationships |
| `sources`, `claims`, `claim_sources` | Evidence, assessments, and provenance |
| `organization_claims`, `brand_claims`, `product_claims` | Claims attached through enforced foreign keys |

Use explicit foreign keys for core relationships instead of generic entity-type/entity-ID references. Use relational fields for structured data; reserve JSONB for limited metadata or preserved source payloads.

### Research schema

| Table | Purpose |
| --- | --- |
| `studies` | Purpose, protocol version, status, retention rules |
| `questionnaire_versions` | Immutable published questionnaire versions |
| `questions`, `question_options` | Stable codes, wording, response types, versioned options |
| `consent_versions` | Exact participant information and consent wording |
| `responses` | Study/version references, consent version, time, recruitment channel, withdrawal-token hash |
| `answers`, `answer_options` | Validated typed answers and multiple-choice selections |
| `response_destinations` | Structured desired countries and cities |
| `dataset_exports` | Dataset versions, filters, transformations, creator, creation date |

A response ID represents a submission, not a verified unique person. The initial cross-sectional questionnaire does not require a persistent participant identifier. Offer a random withdrawal code, store only its hash, and explain withdrawal limits before participation.

### Operations schema

| Table | Purpose |
| --- | --- |
| `admin_users` | Administrator identities and roles |
| `submissions` | Proposed listings/corrections, sources, moderation status |
| `moderation_actions` | Review decisions and reasons |
| `audit_events` | Administrative changes |

Submissions remain unpublished until reviewed. Approved submissions create or update directory records.

## Research governance

Before recruiting participants, define the study protocol and consult the relevant supervisor or institution about ethics review requirements. Academic publication suitability depends on the study design, not just the application.

Each study must document its questions, analysis plan, eligibility, recruitment, participant information, consent procedure, applicable lawful basis, retention, withdrawal, access permissions, and sharing rules. Research participation consent and the lawful basis for processing personal data require separate consideration.

Research exports must preserve questionnaire versions and field definitions, document missing-value handling and transformations, and apply disclosure controls. Review free text for identifying information and suppress rare location combinations in shared aggregates. Retention periods and withdrawal cutoffs must be decided before data collection.

Interpret results accurately:

- Stated interest is not completed volunteering.
- An application-link click is not a confirmed application.
- Voluntary participants form a self-selected sample, not a representative population.
- Without identity verification, one response per person cannot be guaranteed.

Reference: [European Commission guidance on GDPR principles](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/principles-gdpr_en).

## Next planning milestone

Define the first study's research questions and questionnaire, justify every collected field, settle retention and disclosure rules, then finalize database constraints and supported dependency versions before implementation.
