# Azure Advisor Service-Retirement Remediation Plan

**Subscription:** `53b36492-771d-40cb-8527-dc4c191366a7` (AzureCXP-AzureAdvisor-Dev)
**Scope:** Active Azure Advisor recommendations in the `ServiceUpgradeAndRetirement` sub-category
**Total:** 113 recommendations — **all High impact**
**Tracking work item:** [ADO #39676376](https://dev.azure.com/msazure/One/_workitems/edit/39676376)

---

## Summary by retirement date

| # | Recommendation | Retirement date | Status | Count |
|---|----------------|-----------------|--------|------:|
| 1 | Event Grid requires TLS 1.2 or later | 2025-02-28 | Overdue | 2 |
| 2 | Speech-to-text REST preview API v3.1-preview.1 retirement | 2026-03-13 | Overdue | 17 |
| 3 | Azure Machine Learning preview features retirement | 2026-03-31 | Overdue | 30 |
| 4 | General-purpose v1 storage retirement | 2026-10-13 | Upcoming | 12 |
| 5 | Azure Functions .NET retirement | 2026-11-10 | Upcoming | 52 |

> As of 2026-09-15, groups 1–3 (49 recommendations) are already past their retirement date and should be prioritized.

---

## 1. Event Grid TLS 1.0/1.1 retirement — 2 recs (OVERDUE)

- **Type ID:** `ef89da0a-b5d9-4f20-adfb-defbb7f6c1a4`
- **Resource type:** `microsoft.eventgrid/topics`
- **Affected:** `advisor-eventgrid-dev-eastus` in `rg-adv-backfill-pub-dev-eus` (x2)
- **Action required:** Enforce TLS 1.2+ on the topic and ensure all publishers/subscribers use TLS 1.2+.

### Remediation steps

1. **Find affected topics** (Azure Resource Graph):
   ```kusto
   resources
   | where type contains 'microsoft.eventgrid/topics'
     and tostring(properties['minimumTlsVersionAllowed']) in ('1.1','1.0')
   | project subscriptionId, id
   ```
2. **Check client compatibility first** — enforcing TLS 1.2 breaks clients still on 1.0/1.1:
   - Publishers use TLS 1.2+ (modern Azure SDKs / current runtimes default to 1.2+; legacy .NET Framework may need `ServicePointManager.SecurityProtocol = Tls12` or the `SchUseStrongCrypto` registry key).
   - Subscriber/webhook endpoints negotiate TLS 1.2+.
   - Review diagnostic logs for recent 1.0/1.1 handshakes before switching.
3. **Enforce TLS 1.2** (choose one):
   - **Portal:** Event Grid topic → Networking/Configuration → Minimum TLS version → **1.2** → Save.
   - **CLI:**
     ```powershell
     az eventgrid topic update `
       --name advisor-eventgrid-dev-eastus `
       --resource-group rg-adv-backfill-pub-dev-eus `
       --minimum-tls-version 1.2
     ```
   - **REST/ARM property:** `properties.minimumTlsVersionAllowed = "1.2"`
   - **Bicep:**
     ```bicep
     resource topic 'Microsoft.EventGrid/topics@2025-02-15' = {
       name: 'advisor-eventgrid-dev-eastus'
       location: resourceGroup().location
       properties: {
         minimumTlsVersionAllowed: '1.2'
       }
     }
     ```
4. **Validate:** confirm `minimumTlsVersionAllowed == 1.2`, publish a test event, re-run the ARG query (topic should disappear); the Advisor recommendation clears on next refresh (~24h).
5. **Prevent recurrence:** bake the setting into IaC; add Azure Policy to audit/deny topics below TLS 1.2.
- **Docs:** https://go.microsoft.com/fwlink/?linkid=2282034

---

## 2. Speech-to-text REST preview API v3.1-preview.1 retirement — 17 recs (OVERDUE)

- **Action required:** Migrate to the GA Speech-to-text REST API version.
- **Affected resources** (`Retire_Speechservice_SpeechTotextREST`):
  - `rg-advisorcatalog-dev-eus` (x3)
  - `rg-lavkumar-refresh` (x4)
  - `rg-advisorcatalog-dev` (x1)
  - `rg-onboardingassistant-dev-eus` (x2)
  - `rg-azureadvisor-dev` (x2)
  - `rg-nisha-refresh` (x2)
  - `rg-homeng-ai` (x2)
  - `rg-arpgupta-4612` (x1)

---

## 3. Azure Machine Learning preview features retirement — 30 recs (OVERDUE)

- **Action required:** Migrate away from retiring AML preview features.
- **Affected resources** (`Retire_machine_learning_preview_features`):
  - `rg-advisorcatalog-dev-eus` (x4)
  - `rg-nisha-refresh` (x4)
  - `rg-lavkumar-refresh` (x4)
  - `rg-advisorcatalog-dev` (x6)
  - `rg-onboardingassistant-dev-eus` (x4)
  - `rg-homeng-ai` (x4)
  - `rg-azureadvisor-dev` (x4)

---

## 4. General-purpose v1 storage retirement — 12 recs

- **Type ID:** `1d70919c-1a4a-4f79-8300-bb576c291e9d`
- **Retiring feature:** Legacy Blob and General-Purpose v1 storage accounts
- **Action required:** Upgrade to GPv2 / BlockBlobStorage / FileStorage based on workload.
- **Affected resources:**
  - `rgadvisorcoredevuswaceb` [rg-advisorcore-dev-usw3] (x4)
  - `rgadvisorcoredevusw9d65` [rg-advisorcore-dev-usw3] (x4)
  - `rgadvisorcatalogdev9d80` [rg-advisorcatalog-dev] (x4)
- **Docs:** https://learn.microsoft.com/azure/storage/common/storage-account-upgrade

---

## 5. Azure Functions .NET retirement — 52 recs

- **Action required:** Migrate Function Apps to **.NET 10**.
  - 1 app from .NET 9: `func-adv-agent-mcp-dev-krc` [rg-adv-agent-fn-dev-krc]
  - 51 apps from .NET 8 (below).
- **Affected .NET 8 Function Apps:**

```
advisor-sgfanout-pub-dev-eus            [rg-adv-sgfanout-pub-dev-eus]
advisor-recinstancepatchpub-pub-dev-wus [rg-adv-recinstancepatchpub-pub-dev-wus]
fn-adv-lifecyclemanagement-pub-dev-wus  [rg-adv-lifecyclemanagement-pub-dev-wus]
fn-adv-deduporch-pub-dev-wus            [rg-adv-deduporch-pub-dev-wus]
advisor-sgfanout-pub-dev-wus            [rg-adv-sgfanout-pub-dev-wus]
fn-adv-ronboard-pub-dev-eus             [rg-adv-ronboard-pub-dev-eus]
fn-recom-changefeed-pub-dev-eus         [rg-recom-changefeed-pub-dev-eus]
fn-adv-risk-arg-dev-cac                 [rg-adv-risk-arg-dev-cac]
fn-adv-backfill-pub-dev-wus             [rg-adv-backfill-pub-dev-wus]
fn-recom-changefeed-pub-dev-neu         [rg-recom-changefeed-pub-dev-neu]
fn-sg-argrefresh-pub-dev-wus            [rg-sg-argrefresh-pub-dev-wus-test]
fn-adv-cleanup-pub-dev-wus              [rg-adv-cleanup-pub-dev-wus]
advisor-patchpub-pub-dev-wus            [rg-adv-patchpub-pub-dev-wus]
fn-recom-changefeed-pub-dev-wus         [rg-recom-changefeed-pub-dev-wus]
fn-adv-ingestionorc-pub-dev-wus         [rg-adv-ingestionorc-pub-dev-wus]
func-adv-agent-worker-dev-krc           [rg-adv-agent-fn-dev-krc]
advisor-sgbackfill-pub-dev-wus          [rg-adv-sgbackfill-pub-dev-wus]
fn-adv-argref-pub-dev-eus               [rg-adv-argref-pub-dev-eus]
advisor-recplan-pub-dev-neu             [rg-adv-recplan-pub-dev-neu]
fn-adv-rlm-pub-dev-neu                  [rg-adv-rlm-pub-dev-neu]
advisorcatalog-partnerqueryagent-dev-eus [rg-advisorcatalog-dev-eus]
fn-adv-sasing-pub-dev-wus               [rg-adv-sasing-pub-dev-wus]
fn-sg-argref-pub-dev-wus                [rg-adv-sgargref-pub-dev-wus]
demofunctionapp112                      [rg-advisorcore-dev-usw3]
func-adv-agent-api-dev-krc              [rg-adv-agent-fn-dev-krc]
fn-adv-recproc-pub-dev-eus              [rg-adv-recproc-pub-dev-eus]
advisor-recmeta-pub-dev-wus             [rg-adv-recmeta-pub-dev-wus]
fn-adv-ronboard-pub-dev-neu             [rg-adv-ronboard-pub-dev-neu]
advisor-deduporch-pub-dev-eus           [rg-adv-dedup-pub-dev-eus]
advisor-prihandler-pub-dev-wus          [rg-adv-prihandler-pub-dev-wus]
advisorcatalog-partnerquery-dev-eus-test [rg-advisorcatalog-dev-eus]
fn-risk-armapi-dev-cac                  [rg-adv-risk-armapi-dev-cac]
fn-adv-rlm-pub-dev-wus                  [rg-adv-rlm-pub-dev-wus]
advisor-recplan-pub-dev-wus             [rg-adv-recplan-pub-dev-wus]
appfunctiondemo20250314153504           [advisorcore_geneva_auto_create_rg]
advisor-deduporch-pub-dev-neu           [rg-adv-dedup-pub-dev-neu]
advisorcatalog-fn-recc-content-service-dev [rg-advisorcatalog-dev]
fn-adv-lifecyclemanagement-pub-dev-neu  [rg-adv-lifecyclemanagement-pub-dev-neu]
fn-adv-deduporch-pub-dev-neu            [rg-adv-deduporch-pub-dev-neu]
advisorcatalog-fn-reccomendationservice-dev [rg-advisorcatalog-dev]
advisor-deduporch-pub-dev-wus           [rg-adv-dedup-pub-dev-wus]
fn-adv-recproc-pub-dev-neu              [rg-adv-recproc-pub-dev-neu]
fn-adv-risk-dev-cac                     [rg-adv-risk-fn-dev-cac]
fn-risk-armapi-v2-dev-cac               [rg-adv-risk-armapi-v2-dev-cac]
fn-adv-recproc-pub-dev-wus              [rg-adv-recproc-pub-dev-wus]
fnsgargw1                               [rg-alokdesai-sgargdr-test]
fn-adv-rlm-pub-dev-eus                  [rg-adv-rlm-pub-dev-eus]
func-adv-patch-dev-wus                  [rg-adv-patch-dev-wus]
advisorcatalog-fn-configservice-dev     [rg-advisorcatalog-dev]
fn-sg-changefeed-pub-dev-wus            [rg-sg-changefeed-pub-dev-wus]
fn-adv-sasing-pub-dev-neu               [rg-adv-sasing-pub-dev-neu]
```

---

*Generated from Azure Advisor recommendation data for subscription `53b36492-771d-40cb-8527-dc4c191366a7`.*
