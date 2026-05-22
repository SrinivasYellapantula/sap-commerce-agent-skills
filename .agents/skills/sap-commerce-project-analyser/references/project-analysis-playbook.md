# SAP Commerce / SAP CX Project Analysis Playbook

## Purpose

This document explains how an SAP CX Enterprise Architect would quickly read and analyze any SAP Commerce project when given access to the backend and Spartacus codebase.

The goal is not just to understand the code. The goal is to quickly understand what business the platform supports, how the solution is shaped, where the customizations are, where the risks are, and how safely the project can be changed.

---

## 1. Start with the project map, not the code

Before jumping into individual classes, first build a mental map of the solution.

| Area | What to understand |
|---|---|
| Business model | B2B, B2C, marketplace, PunchOut, distributor portal, dealer portal, D2C, multi-brand, multi-country |
| Channels | Spartacus storefront, OCC APIs, Backoffice, SmartEdit, Assisted Service, mobile app, WhatsApp, PunchOut, integrations |
| Commerce scope | Product browsing, pricing, cart, checkout, order management, returns, customer account, reports, approvals |
| SAP CX products | SAP Commerce only, or also CDC, CPQ, Emarsys, Sales Cloud, Service Cloud, SAP ERP, S/4HANA, CPI/SCPI |
| Deployment model | CCv2, on-premise, private cloud, hybrid |
| Ownership | Backend, frontend, integrations, DevOps, QA, content, and master-data ownership |

A common mistake is reading the project like a developer when the real complexity is architectural, integration, data, or operational.

---

## 2. Understand the extension structure first

In SAP Commerce, the extension structure tells the story of the project.

Start with:

```text
localextensions.xml
manifest.json
extensioninfo.xml
build.gradle
external-dependencies.xml
project.properties
local.properties
*-spring.xml
*-items.xml
*-beans.xml
*-facades-spring.xml
*-web-spring.xml
*-backoffice-config.xml
*-impex
```

Classify extensions into buckets:

| Extension type | What to check |
|---|---|
| Core extensions | Data model, services, DAOs, business logic |
| Facades | DTO population, converters, frontend-facing business APIs |
| OCC/webservices | REST APIs, controllers, request/response DTOs |
| Storefront/Spartacus integration | CMS, OCC customizations, frontend contracts |
| Integration extensions | S/4HANA, CPI, Akeneo, payment, tax, shipping, CRM |
| Backoffice extensions | Custom widgets, editor areas, workflows, admin tools |
| Initial data/sample data | Catalogs, content, sites, users, configuration |
| Cronjob/process extensions | Jobs, business processes, scheduled integrations |
| Test extensions | Unit tests, integration tests, test data, mock integrations |

Immediately identify:

```text
Which extensions are OOTB?
Which extensions are custom?
Which custom extensions are project-specific?
Which are reusable accelerators/framework extensions?
Which ones override SAP behavior?
```

A mature architect reads the project through extension boundaries.

Bad signs:

```text
Too much logic in web controllers
Huge utility classes
Core logic spread randomly across storefront, OCC, facades, and services
OOTB classes copied and modified
Business rules hidden in populators
Excessive use of static helpers
Direct FlexibleSearch everywhere
Hardcoded baseSite, baseStore, catalog, or customer logic
```

---

## 3. Analyze data modeling deeply

Data modeling is one of the best starting points.

Inspect all custom:

```text
*-items.xml
```

### 3.1 Custom item types

Examples:

```xml
<itemtype code="CustomProduct" extends="Product">
<itemtype code="Dealer" extends="B2BUnit">
<itemtype code="BookingRequest">
<itemtype code="ExternalServiceContract">
<itemtype code="CustomerReport">
```

Ask:

| Question | Why it matters |
|---|---|
| What business concepts are modeled as itemtypes? | Reveals domain complexity |
| Are OOTB types extended or reused cleanly? | Impacts upgradeability |
| Are attributes localized? | Impacts catalog/content behavior |
| Are attributes indexed? | Impacts performance |
| Are relations modeled correctly? | Impacts data integrity |
| Are enums used properly? | Impacts flexibility |
| Are dynamic attributes overused? | Impacts performance/debugging |
| Are partOf relations used correctly? | Impacts cascading deletion |
| Are collection attributes abused? | Impacts querying and maintenance |
| Are custom attributes added to Cart, Order, Product, or Customer? | Reveals checkout/order/data complexity |

### 3.2 Key SAP Commerce data areas

Review customizations on:

```text
ProductModel
VariantProductModel
CategoryModel
CatalogVersionModel
BaseSiteModel
BaseStoreModel
CMSSiteModel
CustomerModel
B2BCustomerModel
B2BUnitModel
AddressModel
CartModel
AbstractOrderModel
OrderModel
ConsignmentModel
PaymentInfoModel
PriceRowModel
StockLevelModel
PromotionResultModel
BusinessProcessModel
CronJobModel
MediaModel
```

These types usually reveal where the project has changed standard Commerce behavior.

### 3.3 Data model smell checklist

Flag:

```text
String fields used where enum/reference should exist
Large JSON stored in String/Text fields without governance
Too many attributes added directly to ProductModel
Sensitive data stored unencrypted
Relation cardinality not matching business reality
Missing indexes on frequently queried fields
Duplicate data copied across Cart, Order, Consignment, and Customer
Business state stored as free text
No audit/status history for critical flows
```

---

## 4. Understand catalogs, base sites, base stores, and content structure

SAP Commerce architecture is heavily driven by:

```text
BaseSite
BaseStore
CMSSite
ProductCatalog
ContentCatalog
CatalogVersion
Storefront channel
Language
Currency
Warehouse
PointOfService
SolrIndexedType
```

Inspect impex and configuration for:

```text
baseSite
baseStore
cmsSite
catalog
catalogVersion
solrIndexedType
languages
currencies
countries
delivery modes
payment modes
warehouses
stores
site-channel mapping
```

Important questions:

| Question | Why it matters |
|---|---|
| Is this single-site or multi-site? | Impacts all frontend/backend logic |
| Are product catalogs shared? | Impacts product governance |
| Are content catalogs shared? | Impacts CMS rollout |
| Are base stores country-specific? | Impacts pricing, stock, tax, integrations |
| Are languages/currencies dynamic? | Impacts Spartacus and OCC |
| Is there brand-based visibility? | Impacts CMS/product/category rules |
| Are restrictions used properly? | Impacts personalization and content bugs |

Inspect impex files such as:

```text
essentialdata.impex
projectdata.impex
sampledata.impex
cms-content.impex
solr.impex
base-store.impex
base-site.impex
catalog.impex
user-groups.impex
```

---

## 5. Analyze initialization, update, and impex strategy

Check:

```text
resources/**/impex
resources/**/essentialdata
resources/**/projectdata
resources/**/sampledata
SystemSetup classes
@SystemSetup annotations
ImpExImportJob
ImportCockpit / Backoffice import usage
```

Understand:

| Area | What to inspect |
|---|---|
| Essential data | Mandatory system configuration |
| Project data | Real project setup |
| Sample data | Demo/local-only data |
| Environment-specific impex | Dev/QA/UAT/Prod differences |
| Update safety | Whether updates are repeatable |
| Idempotency | Whether impex can be safely re-run |
| Ownership | Who maintains CMS/product/customer data |

Bad signs:

```text
Production-critical config only in manual Backoffice changes
Impex not idempotent
Environment-specific values hardcoded
No clear separation between essential/project/sample data
Manual post-deployment steps
Duplicate CMS components or slots due to non-unique impex
```

---

## 6. Read the Spring configuration and bean overrides

SAP Commerce architecture often hides in Spring XML.

Search for:

```text
alias
parent=
abstract=
class=
<property name=
<list>
<map>
<util:list>
converter
populator
strategy
facade
service
dao
validator
interceptor
eventListener
```

Especially inspect:

```text
*-spring.xml
*-facades-spring.xml
*-web-spring.xml
*-occ-spring.xml
*-backoffice-spring.xml
```

Identify:

| Bean type | What to check |
|---|---|
| Service overrides | Business logic changes |
| Strategy overrides | Checkout/cart/pricing/search behavior changes |
| Converter/populator changes | API response impact |
| Interceptors | Data validation, prepare, remove side effects |
| Validators | Business rules |
| Event listeners | Async or hidden behavior |
| Cronjob beans | Scheduled processing |
| Business process actions | Order/process workflow |
| RestTemplate beans | Integration behavior |
| OCC controller beans | API customizations |

Architectural question:

```text
Is the project extending SAP Commerce through clean strategies and services,
or hacking behavior through copied classes and overrides?
```

---

## 7. Analyze services, DAOs, facades, and controllers layer by layer

Inspect backend in this order:

```text
Controller -> Facade -> Service -> DAO -> Model
```

Do not trust naming blindly. Check whether responsibilities are clean.

### 7.1 OCC controllers

Look for:

```text
*Controller.java
@RequestMapping
@GetMapping
@PostMapping
@Secured
@PreAuthorize
@ApiOperation
@RequestBody
@RequestParam
@PathVariable
DataMapper
WsDTO
```

Check:

| Area | What to look for |
|---|---|
| API contract | URL, method, request, response |
| Security | Anonymous, customer, trusted client, B2B user |
| Validation | Input validation, DTO validation |
| Error handling | Custom exceptions, error codes |
| Performance | Heavy logic directly in controller |
| API versioning | v2/v3/custom APIs |
| Base site awareness | Whether API behaves differently by site/store |

Bad signs:

```text
Business logic inside controller
Direct model exposure
FlexibleSearch from controller
Large request DTOs without validation
No authorization checks for B2B data
Returning backend exception messages to frontend
```

### 7.2 Facades

Facades usually show frontend business journeys.

Look for:

```text
*Facade
*FacadeImpl
Converter
Populator
Data
WsDTO
```

Check:

```text
What frontend needs are being served?
What backend domain models are converted?
Are populators becoming business-rule dumping grounds?
Is there duplication between OCC DTO and facade DTO?
```

### 7.3 Services

Services should hold business logic.

Look for:

```text
*Service
*ServiceImpl
```

Check:

```text
Does each service have a clear domain?
Are transactions handled correctly?
Are model saves explicit and controlled?
Are integrations abstracted behind services?
Are service methods reusable?
```

### 7.4 DAOs

Look for:

```text
FlexibleSearchQuery
GenericQuery
SearchResult
FlexibleSearchService
```

Check:

```text
Are queries indexed?
Are parameters safely bound?
Are queries paginated?
Are joins expensive?
Are results restricted by baseSite/baseStore/catalogVersion/user?
Are large result sets loaded into memory?
```

---

## 8. Analyze interceptors and validators

Interceptors are extremely important in SAP Commerce because they create hidden behavior.

Inspect:

```text
PrepareInterceptor
ValidateInterceptor
RemoveInterceptor
LoadInterceptor
InitDefaultsInterceptor
```

Questions:

| Question | Why it matters |
|---|---|
| Which model types have interceptors? | Hidden lifecycle rules |
| Do interceptors call services? | Can cause recursion/performance issues |
| Do they perform external calls? | Dangerous |
| Do they modify many related models? | Side effects |
| Do they rely on session/baseSite/user? | Fragile |
| Are they disabled during imports? | Impex performance/data behavior |

Bad signs:

```text
External API call inside interceptor
FlexibleSearch inside interceptor for frequently saved models
Saving models inside prepare interceptor carelessly
Business process triggered from interceptor unexpectedly
Interceptor logic depending on current user/site during cronjob/import
```

---

## 9. Understand cart and checkout customization

Cart and checkout are usually the most customized and risk-heavy areas.

Inspect:

```text
CommerceCartService
CartFacade
CheckoutFacade
CalculationService
CartCalculationStrategy
AddToCartStrategy
CommerceCartModification
CartValidationStrategy
PaymentService
DeliveryService
PlaceOrderStrategy
CommerceCheckoutService
OrderService
```

Check:

| Area | What to check |
|---|---|
| Add to cart | Product eligibility, min/max quantity, B2B restrictions |
| Cart validation | Stock, price, customer, address, payment, threshold |
| Pricing | SAP pricing, Commerce price rows, custom price APIs |
| Tax | Internal or external tax engine |
| Promotions | OOTB or custom |
| Payment | Online, offline, credit, invoice, wallet, OBC, custom |
| Delivery | Shipping modes, warehouses, pickup |
| Order placement | Sync or async ERP order creation |
| Cart merge | Anonymous-to-logged-in behavior |
| Recalculation | When and how cart totals update |

Bad signs:

```text
Custom price stored manually on cart entries without recalculation discipline
Order placement depends on fragile external sync calls
Stock checked only on PDP, not checkout
B2B authorization missing on cart access
Cart restored across wrong B2B units
Calculation disabled to work around bugs
```

---

## 10. Analyze order management and business processes

Inspect:

```text
processdefinition.xml
BusinessProcessModel
OrderProcessModel
ConsignmentProcessModel
Action classes
Decision actions
Wait nodes
Events
BusinessProcessService
```

Key files:

```text
resources/processes/*.xml
*-process-spring.xml
```

Check:

| Area | What to inspect |
|---|---|
| Order process | From order placed to completion |
| Consignment process | Fulfillment behavior |
| Retry behavior | Failed integration retry |
| Event triggering | Who triggers what event |
| Timeout/wait nodes | Long-running processes |
| Error handling | Failed process recovery |
| Manual intervention | Backoffice actions |

Bad signs:

```text
Business process stuck without alerting
No retry design for ERP failures
Order status and process state inconsistent
Process actions doing too much
No idempotency in external order submission
```

---

## 11. Analyze integration architecture

For enterprise SAP Commerce projects, integrations are often the real architecture.

Create an integration inventory.

| Integration | Direction | Sync/Async | Trigger | Protocol | Owner |
|---|---|---|---|---|---|
| Product master | Inbound | Async | Cron/event | CPI/REST/SFTP | ERP/PIM |
| Customer master | Outbound | Sync/Async | Registration/update | REST/CPI | Commerce |
| Pricing | Sync | Real-time | PDP/cart | REST/SOAP/RFC | ERP |
| Stock | Sync/Async | PDP/cart/cron | REST/CPI | ERP/WMS |
| Order | Outbound | Async/sync | Place order | CPI/REST | ERP |
| Invoice | Inbound/fetch | Sync | My account | REST | ERP |
| Payment | Outbound | Sync | Checkout | PSP API | PSP |
| Tax | Sync | Cart/checkout | REST | Tax engine |

Inspect:

```text
RestTemplate
WebClient
HttpClient
OutboundServiceFacade
ConsumedDestinationModel
ConsumedOAuthCredentialModel
DestinationTarget
IntegrationObject
Integration APIs
OAuth config
Retry templates
Timeout config
Circuit breaker patterns
Cronjobs
Events
```

Important questions:

```text
Which system is master for product/customer/order/pricing/stock?
What is real-time vs replicated?
What happens when the external system is down?
Are calls idempotent?
Are retries safe?
Are errors persisted?
Are payloads auditable?
Are secrets stored safely?
Are timeouts configured?
Are integrations environment-specific?
```

Bad signs:

```text
No timeout configuration
No retry/backoff policy
No idempotency key
No payload logging for critical integrations
External calls during model save
Business users cannot reprocess failures
Environment URLs hardcoded
Credentials in properties or impex without encryption
```

---

## 12. Analyze Solr/search architecture

Inspect:

```text
SolrIndexedType
SolrIndexedProperty
SolrValueProvider
FieldValueProvider
ValueResolver
SolrFacetSearchConfig
SolrIndexConfig
SolrSort
SearchQueryPopulator
ProductSearchFacade
```

Files:

```text
solr.impex
*-solr-spring.xml
```

Check:

| Area | What to inspect |
|---|---|
| Indexed types | Product, variant, content, custom |
| Indexed properties | Searchable/facet/sort/filter fields |
| Value providers | Custom logic and performance |
| Catalog versions | Online/Staged behavior |
| Indexing mode | Full, update, two-phase |
| Facets | Business relevance |
| Search restrictions | Visibility logic |
| Variant indexing | Base/variant product behavior |

Bad signs:

```text
Heavy DB calls inside Solr value providers
Indexing too many unnecessary fields
Variant visibility logic inconsistent with PDP
Search result differs from product detail availability
Frequent full indexing due to bad delta design
Solr fields used as workaround for missing data model
```

---

## 13. Analyze CMS and SmartEdit

Inspect:

```text
CMSComponent
ContentSlot
ContentPage
PageTemplate
CMSRestriction
CMSNavigationNode
CMSLinkComponent
SimpleCMSComponent
ProductCarouselComponent
CMSParagraphComponent
Custom CMS components
```

Also inspect:

```text
*-items.xml for CMS components
cms-content.impex
*-cms.impex
SmartEdit configuration
CMS restrictions
Component controllers
Spartacus CMS mapping
```

Check:

| Area | What to inspect |
|---|---|
| Page structure | Templates, slots, pages |
| Component model | Custom CMS components |
| Restrictions | User, time, category, brand, site |
| Navigation | Header/footer/category navigation |
| SmartEdit | Preview, editing, sync |
| Spartacus mapping | Backend CMS component to Angular component |
| Content catalog sync | Staged-to-Online process |

Bad signs:

```text
Hardcoded frontend components instead of CMS-driven
Too many duplicate CMS components
CMS restrictions used for complex business authorization
Frontend assumes component data shape not guaranteed by backend
SmartEdit broken due to CSP, preview, or CORS issues
```

---

## 14. Analyze Backoffice customizations

Inspect:

```text
backoffice-config.xml
*-backoffice-config.xml
widgets.xml
editorArea
listView
advanced-search
collection browser
custom renderers
custom editors
cockpitng actions
```

Ask:

```text
What can business/admin users manage?
What operations require developer/database intervention?
Are failure reprocess actions available?
Are integrations visible?
Can support users troubleshoot orders/customers/products?
```

Bad signs:

```text
Critical support operations require HAC queries
No Backoffice visibility for failed integrations
Custom actions without authorization control
Backoffice widgets directly executing heavy queries
```

---

## 15. Analyze cronjobs and scheduled processing

Inspect:

```text
ServicelayerJob
CronJobModel
JobPerformable
FlexibleSearchCronJob
CompositeCronJob
TriggerModel
```

Files/classes:

```text
*Job.java
*Performable.java
cronjob.impex
*-jobs-spring.xml
```

Check:

| Area | What to inspect |
|---|---|
| Job purpose | Import, export, cleanup, sync, reporting |
| Frequency | How often it runs |
| Runtime | How long it takes |
| Cluster behavior | Runs on all nodes or one node |
| Error handling | Retry/failure behavior |
| Logging | Enough to troubleshoot |
| Data volume | Pagination/batching |
| Cleanup | Old logs/media/processes |

Bad signs:

```text
Cronjobs loading all data into memory
No pagination
No cluster safety
No log cleanup
No alerting on failures
No last-success tracking
Manual rerun requires developer
```

---

## 16. Analyze events and asynchronous behavior

Search for:

```text
publishEvent
ApplicationEvent
EventListener
AbstractEventListener
AfterSaveListener
BusinessProcessEvent
```

Check:

```text
What events exist?
Who publishes them?
Who consumes them?
Are they synchronous or asynchronous?
Are failures visible?
Are duplicate events safe?
```

Bad signs:

```text
Critical logic hidden in event listeners
No idempotency
Event failure not monitored
Listener assumes session/baseSite/customer exists
```

---

## 17. Analyze security architecture

### 17.1 Authentication

Inspect:

```text
OAuth clients
authorizationserver
resourceserver
Spring Security config
Spartacus auth config
CDC/CIAM/MSAL/SSO integration
ASM
Remember-me
Session timeout
Token lifetime
PKCE
```

Questions:

```text
What login flows exist?
Customer login, B2B login, admin login, PunchOut login, API client login?
Are public clients using PKCE?
Are password/implicit flows removed?
Are tokens scoped correctly?
```

### 17.2 Authorization

Inspect:

```text
UserGroup
PrincipalGroup
B2BUnit
B2BPermission
B2BRole
SearchRestriction
@Secured
@PreAuthorize
CMSRestriction
CatalogVersion permissions
Backoffice roles
```

Questions:

```text
Can users access only their own orders/carts/reports?
Can B2B users access only their own unit data?
Are OCC APIs protected correctly?
Are admin APIs exposed?
Are search restrictions correct?
```

### 17.3 Data protection

Check:

```text
PII storage
Encryption
Masked logging
Payment data handling
GDPR deletion/anonymization
Sensitive DTO fields
Audit logs
```

Bad signs:

```text
Customer/order data accessible by code only
Missing B2B unit checks
Sensitive values in logs
Client secrets in properties
Frontend-controlled authorization decisions
```

---

## 18. Analyze Spartacus architecture

Start with:

```text
package.json
angular.json
app.module.ts
spartacus-configuration.module.ts
app-routing.module.ts
feature modules
custom components
CMS mappings
services
guards
interceptors
NgRx customizations
SSR setup
```

Identify:

| Area | What to check |
|---|---|
| Spartacus version | Compatibility with Commerce version |
| Feature libraries | cart, checkout, order, ASM, organization, product, user |
| Custom CMS components | Mapping from backend CMS |
| Custom OCC adapters | API customizations |
| Auth config | OAuth, PKCE, login redirects |
| Routing | Custom routes, guards |
| State management | Custom stores/effects/facades |
| Styling | Global CSS overrides, responsive behavior |
| SSR | Server rendering, timeouts, API calls |
| Performance | Lazy loading, bundle size, duplicate calls |

### 18.1 Spartacus code areas to inspect

```text
src/app/spartacus/
src/app/features/
src/app/cms-components/
src/app/core/
src/app/shared/
src/styles.scss
src/assets/
server.ts
main.server.ts
proxy.conf.json
environment.ts
```

### 18.2 Spartacus integration points

Inspect:

```text
provideConfig
cmsComponents
backend.occ.baseUrl
context.baseSite
context.language
context.currency
authentication
routing
features
i18n
```

Search for:

```text
cmsComponents:
OccEndpoints
provideConfig
AuthConfig
OAuthLibConfig
RoutingConfig
translationChunksConfig
```

Bad signs:

```text
Hardcoded baseSite/language/currency
Direct HTTP calls instead of OCC adapters/connectors
Business logic inside components
Large global CSS hacks
No lazy loading
Duplicate OCC calls on page load
SSR timeout due to browser-only code
Authentication flow customized without understanding token lifecycle
```

---

## 19. Analyze OCC contract between backend and Spartacus

Create an API contract map.

| Page/Feature | Spartacus component/service | OCC API | Backend controller/facade |
|---|---|---|---|
| PDP | ProductDetailsComponent | `/products/{code}` | ProductController |
| Cart | CartComponent | `/users/{userId}/carts` | CartController |
| Checkout | Checkout components | `/checkout` APIs | CheckoutController |
| Orders | Order history | `/orders` | OrdersController |
| Reports | Custom report component | `/reports` | CustomReportController |

Check:

```text
Are frontend models aligned with backend DTOs?
Are fields coming from backend or computed in frontend?
Are calculations duplicated?
Are enum/status values mapped safely?
Are errors handled consistently?
Are APIs cacheable where needed?
```

Bad signs:

```text
Same business calculation in Java and Angular
Frontend reconstructing totals/prices/taxes
Backend returns loosely structured maps
Frontend depends on undocumented optional fields
Breaking API changes without versioning
```

---

## 20. Analyze pricing, stock, and availability

Identify:

```text
Where does price come from?
Where does stock come from?
When are price and stock refreshed?
Are they site/store/customer-specific?
Are they real-time?
Are they stored or calculated?
```

Areas to inspect:

```text
PriceFactory
Europe1PriceFactory
PriceRow
DiscountRow
TaxRow
StockService
CommerceStockService
AvailabilityService
WarehouseModel
ATP integrations
ERP pricing services
CartCalculationStrategy
```

Questions:

```text
Is pricing customer-specific?
Is pricing B2B unit-specific?
Is pricing country/currency-specific?
Is stock plant/warehouse-specific?
Can price change between PDP and checkout?
What happens if the price service is down?
```

Bad signs:

```text
Frontend calculates final price
Stock not revalidated at checkout
Price stored in custom fields without recalculation
ERP price call made too frequently without caching
No fallback/error behavior for pricing downtime
```

---

## 21. Analyze product and catalog architecture

Inspect:

```text
ProductModel
VariantProductModel
ClassificationClass
Feature
CategoryModel
CatalogVersion
Supercategories
Product references
Media gallery
Approval status
Stock levels
Price rows
Solr visibility
```

Questions:

```text
Is product data mastered in Commerce, ERP, PIM, or manually?
Are variants modeled correctly?
Are classifications used or custom attributes?
Are categories business categories or navigation categories?
Are products shared across countries/sites?
How are discontinued/unapproved/unavailable products handled?
```

Bad signs:

```text
Too many product attributes directly on ProductModel
Category structure used for business rules everywhere
Product visibility inconsistent between Solr/PDP/cart
Variant logic duplicated in frontend
No clear master system for product data
```

---

## 22. Analyze customer, B2B, organization, and account model

For B2B projects, inspect:

```text
B2BUnit
B2BCustomer
B2BUserGroup
B2BPermission
B2BBudget
B2BCostCenter
B2BOrderApprovalProcess
OrgUnit
Address
Contact person
Sold-to / Ship-to / Bill-to / Payer
```

Questions:

```text
How is the customer hierarchy modeled?
Is SAP customer master replicated?
Are contacts created in Commerce and sent to ERP?
Are addresses mastered in ERP or Commerce?
Are partner functions modeled clearly?
Are B2B permissions used or custom authorization?
```

Bad signs:

```text
B2BUnit hierarchy does not match ERP reality
Partner functions stored as strings everywhere
Authorization checks implemented inconsistently
User can switch units without proper validation
Address ownership unclear
```

---

## 23. Analyze checkout-to-ERP flow

For SAP Commerce enterprise projects, always map this flow:

```text
Cart -> Validation -> Price/Stock -> Payment/Approval -> Order -> ERP Order -> Confirmation -> Invoice/Delivery/Status
```

Identify:

| Step | Key question |
|---|---|
| Cart creation | Anonymous, logged-in, B2B, PunchOut? |
| Price | Commerce price or ERP price? |
| Stock | Commerce stock or real-time ATP? |
| Order placement | Sync or async ERP call? |
| Order status | Commerce-owned or ERP-owned? |
| Documents | Invoice/proforma/dispatch from ERP? |
| Reprocessing | Can failed orders be retried? |

Bad signs:

```text
Order created in Commerce but not ERP with no recovery
ERP call failure exposes bad UX
No order correlation ID
No integration audit trail
Order status manually patched
```

---

## 24. Analyze email, notification, and communication flows

Inspect:

```text
EmailPageTemplate
EmailPage
EmailContext
RendererTemplate
Process actions
NotificationService
Event listeners
```

Questions:

```text
Which emails are sent?
When are they sent?
Are templates CMS-managed?
Are emails localized?
Are failures logged?
Are attachments generated?
```

Bad signs:

```text
Email sending done directly inside business logic
No retry
No template ownership
PDF/email generation tightly coupled to order placement
```

---

## 25. Analyze media, documents, and PDF generation

Inspect:

```text
MediaModel
MediaFolder
MediaService
MediaStorageStrategy
PDF generation utilities
Apache POI / iText / Jasper usage
Document storage
```

Questions:

```text
Are PDFs generated in Commerce or fetched from ERP?
Are generated files stored as Media?
Are old files cleaned up?
Are fonts/resources packaged correctly?
Are documents secured per user/B2B unit?
```

Bad signs:

```text
Temporary files not cleaned up
PDFs stored forever without retention
Fonts loaded from local paths
Sensitive documents accessible without authorization
Large files generated synchronously during page load
```

---

## 26. Analyze performance and scalability

### Backend

Inspect:

```text
FlexibleSearch queries
Cronjobs
Solr indexing
Interceptors
Populators
Integration calls
Cache usage
Session usage
ModelService saves
```

### Frontend

Inspect:

```text
Initial bundle size
Lazy loading
SSR timeout
Duplicate API calls
Large CMS payloads
Image optimization
Carousel/render-heavy components
```

### Platform

Inspect:

```text
Node count
Cluster config
Cache regions
DB indexes
Solr config
Media storage
CCv2 deployment config
Dynatrace/New Relic/AppDynamics
Logs
```

Bad signs:

```text
FlexibleSearch in loops
Populators performing DB calls per item
No pagination
Synchronous external calls on PLP
Too many OCC calls during SSR
Large CMS payloads
No DB indexes for custom query fields
```

---

## 27. Analyze caching strategy

Check:

```text
SAP Commerce cache regions
EHCache/Hazelcast config
FlexibleSearch query cache
Solr cache
HTTP cache headers
CDN
Spartacus transfer state
SSR caching
Integration response caching
```

Questions:

```text
What is cached?
Where is it cached?
How is it invalidated?
Is cache site/user/customer-specific?
Can stale price/stock/customer data be shown?
```

Bad signs:

```text
Caching customer-specific data globally
No cache invalidation strategy
Over-caching price/stock
No CDN strategy for media/static assets
```

---

## 28. Analyze error handling and observability

Inspect:

```text
Exception classes
ControllerAdvice
RestHandlerExceptionResolver
Logging patterns
Correlation IDs
Integration audit tables
Cronjob logs
Business process logs
Dynatrace traces
Kibana dashboards
Alerting
```

Questions:

```text
Can support trace one order end-to-end?
Can we find why an API failed?
Are external request/response payloads logged safely?
Are business errors different from technical errors?
Are alerts configured for critical failures?
```

Bad signs:

```text
Only stack traces, no business context
No correlation ID
Errors swallowed and logged as info
No alerting on failed order integrations
Sensitive payloads logged fully
```

---

## 29. Analyze deployment and environment setup

Inspect:

```text
manifest.json
build.gradle
external-dependencies.xml
local.properties
project.properties
environment properties
CCv2 manifest
cloud hot folders
media config
Solr config
health checks
pipeline config
```

Questions:

```text
How many environments exist?
How are properties managed?
How are secrets managed?
How are releases deployed?
Are DB updates automated?
Are impex updates automated?
Are rollback steps documented?
```

Bad signs:

```text
Manual production config
Secrets committed
Environment-specific code branches
No rollback plan
Deployment requires tribal knowledge
```

---

## 30. Analyze testing maturity

Inspect:

```text
JUnit tests
Mockito tests
Integration tests
ServicelayerTransactionalTest
OCC API tests
Spartacus unit tests
Cypress/Playwright tests
Postman collections
Performance tests
Contract tests
```

Questions:

```text
Are core services covered?
Are checkout/order/integration flows tested?
Are custom OCC APIs tested?
Are frontend and backend contracts tested?
Are data imports tested?
```

Bad signs:

```text
Only controller happy-path tests
No tests for pricing/order/integration failure
No test data strategy
Tests rely on environment state
```

---

## 31. Analyze upgradeability

Check:

```text
OOTB class overrides
Deprecated APIs
Custom copies of SAP classes
Spring bean aliases replacing SAP beans
Modified accelerator/spartacus code
Custom patches
Use of removed OAuth flows
Use of legacy extensions
```

Questions:

```text
How painful will the next SAP Commerce upgrade be?
Have OOTB classes been copied?
Are extension points used properly?
Are deprecated APIs present?
Is Spartacus compatible with Commerce version?
```

Bad signs:

```text
Copied OOTB classes with minor changes
Deep overrides of checkout/calculation/search
Legacy OAuth/password grant dependencies
Deprecated APIs everywhere
No upgrade notes
```

---

## 32. Analyze documentation and knowledge transfer quality

Look for:

```text
README
Architecture diagrams
Integration specs
API specs
Data model diagrams
Deployment guide
Runbook
Troubleshooting guide
KT recordings/transcripts
Postman collections
Sequence diagrams
```

Create missing documentation while analyzing.

Minimum documents to expect or create:

```text
Solution overview
Extension map
Data model summary
Integration inventory
Checkout/order flow
Deployment/runbook
Known issues and risks
Environment/property matrix
API contract list
Backoffice operations guide
```

---

## 33. Preferred first 2 days analysis sequence

### Day 1: Architecture discovery

```text
1. Read localextensions.xml
2. Identify all custom extensions
3. Read items.xml files
4. Read base site/base store/catalog impex
5. Read key Spring XML overrides
6. Identify custom OCC controllers
7. Identify custom facades/services/DAOs
8. Identify integrations and cronjobs
9. Review Spartacus config and CMS mappings
10. Build first architecture map
```

### Day 2: Business flow discovery

```text
1. Trace login/auth flow
2. Trace product listing and PDP flow
3. Trace cart add/update flow
4. Trace checkout/place order flow
5. Trace pricing/stock calculation
6. Trace customer/account/order history
7. Trace integration failure handling
8. Trace CMS-driven pages/components
9. Review Backoffice support operations
10. Document risks, unknowns, and quick wins
```

---

## 34. Search terms to use in the codebase

### Backend search terms

```text
extends Product
extends VariantProduct
extends Customer
extends B2BUnit
extends AbstractOrder
extends Cart
extends Order
PrepareInterceptor
ValidateInterceptor
RemoveInterceptor
FlexibleSearchQuery
GenericQuery
publishEvent
BusinessProcessService
ServicelayerJob
JobPerformable
RestTemplate
WebClient
ConsumedDestination
OAuth
@Secured
@PreAuthorize
@RequestMapping
Converter
Populator
SolrValueProvider
ValueResolver
CartCalculationStrategy
AddToCartStrategy
PlaceOrder
Checkout
OrderProcess
ConsignmentProcess
```

### Spartacus search terms

```text
provideConfig
cmsComponents
OccEndpoints
AuthConfig
OAuthLibConfig
RoutingConfig
HttpInterceptor
CmsConfig
ConfigModule.withConfig
backend.occ
baseSite
language
currency
cart
checkout
order
product
user
organization
```

---

## 35. Outputs to create after analysis

An architect should not just understand the project silently. They should produce clear artifacts.

### 35.1 Extension map

```text
customcore
customfacades
customocc
custombackoffice
custominitialdata
customintegration
customstorefront
```

For each extension:

```text
Purpose
Key itemtypes
Key services
Key APIs
Dependencies
Risks
```

### 35.2 Data model map

```text
Business concept -> SAP Commerce model -> Source system -> Used by
```

Example:

```text
Dealer -> B2BUnitModel -> S/4HANA -> Login, pricing, order placement
Contract -> CustomContractModel -> ERP/API -> Cart validation, reports
```

### 35.3 Integration map

```text
System -> Direction -> API -> Trigger -> Error handling -> Owner
```

### 35.4 Critical flow diagrams

At minimum:

```text
Login
Product search
PDP
Add to cart
Checkout
Place order
Order status/document fetch
Customer registration/update
Product import
Price/stock fetch
```

### 35.5 Risk register

```text
Risk
Impact
Evidence
Recommendation
Priority
```

### 35.6 Change impact matrix

```text
If we change X, impacted areas are Y.
```

Example:

```text
Changing product model impacts:
- Solr indexing
- PDP OCC response
- Spartacus product components
- Impex imports
- Backoffice editor area
- Integration mapping
```

---

## 36. SAP Commerce project comprehension checklist

Reusable checklist:

```text
1. Business model understood
2. Extension structure understood
3. Custom itemtypes reviewed
4. Base sites/base stores/catalogs reviewed
5. Impex strategy reviewed
6. Spring bean overrides reviewed
7. OCC APIs reviewed
8. Facades/services/DAOs reviewed
9. Interceptors/validators reviewed
10. Cart/checkout/order flow traced
11. Pricing/stock/tax strategy understood
12. Product/catalog/Solr strategy understood
13. CMS/SmartEdit setup understood
14. Spartacus config and CMS mappings reviewed
15. Auth/security/authorization reviewed
16. B2B/customer/org model reviewed
17. Integrations inventoried
18. Cronjobs and async flows reviewed
19. Business processes reviewed
20. Backoffice support tooling reviewed
21. Media/document/PDF handling reviewed
22. Performance risks reviewed
23. Caching strategy reviewed
24. Observability/logging reviewed
25. Deployment/config/secrets reviewed
26. Testing maturity reviewed
27. Upgradeability reviewed
28. Documentation gaps identified
29. Critical risks documented
30. Change impact matrix created
```

---

## 37. Mindset of a senior SAP CX architect

A developer asks:

```text
Where is this method called?
```

An architect asks:

```text
Why does this business capability exist?
Which system owns this data?
Which channel consumes it?
What happens when it fails?
How is it secured?
How does it scale?
How will it behave during upgrade?
Can support teams operate it?
Can we safely change it?
```

The fastest path to SAP Commerce project comprehension is:

```text
Data model -> Extension structure -> Site/catalog/store setup -> Spring overrides -> OCC/facade/service flow -> Integrations -> Checkout/order process -> Spartacus mapping -> Security/performance/operations
```

This gives the full picture quickly without getting lost in random classes.
