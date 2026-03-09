# BMW APIOps – Pending Tasks Execution Steps (as of 2026-03-05)

This document lists only the tasks that are **not completed yet** as of 2026-03-05, based on your tracker and the statement that workshops/troubleshooting items are already done.

## Scope and assumptions

- Platform: **GitHub Enterprise Server (GHES)**
- APIM environments in scope: **apim dev** and **apim uat**
- Repository: `apiops-test-main`
- Existing workflows in repo:
  - `.github/workflows/run-extractor-ghes.yaml`
  - `.github/workflows/run-publisher.yaml`
  - `.github/workflows/run-publisher-with-env.yaml`
- APIOps references:
  - https://github.com/Azure/apiops/tree/main/docs
  - https://github.com/Azure/apiops/blob/main/docs/apiops/3-apimTools/apimtools-azdo-2-3-new.md

---

## 1) Environment Preparation (Pending)

### 1.1 Prepare Azure Service Principal (Dev/UAT)

1. In Azure AD, create one service principal for APIOps automation (or one per environment if BMW policy requires strict isolation).
2. Capture `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`.
3. Assign RBAC on APIM resource group:
   - Minimum: `Contributor` on the APIM resource group used by `apim dev` and `apim uat`.
4. Verify access by running a dry-run extractor workflow against dev.
5. Store credentials as **GHES repository/environment secrets** (never in code or YAML files).

**Output:** Service principal credentials available in GHES secrets and validated with a successful auth run.

### 1.2 Prepare VS Code

1. Install VS Code extensions: YAML, Git tooling, Azure Account, APIM-related helpers if BMW allows.
2. Configure Git identity in terminal:
   - `git config --global user.name "<name>"`
   - `git config --global user.email "<email>"`
3. Verify repo line endings and YAML formatting consistency.
4. Open the repository and validate workflows can be edited/linted locally.

**Output:** Local workstation ready for APIOps authoring and Git operations.

### 1.3 Prepare GitHub Enterprise repository settings

1. Confirm Actions are enabled for the repository.
2. Configure branch protection on `main` (required PR, review, and status checks).
3. Create GHES environments `dev` and `uat` (environment-level secrets + approvals if needed).
4. Add required secrets per environment:
   - `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`
   - `AZURE_RESOURCE_GROUP_NAME`, `API_MANAGEMENT_SERVICE_NAME`
   - PAT token for PR automation if needed by extractor PR step.
5. Validate access control: only approved maintainers can trigger publish to `uat`.

**Output:** Repository governance and GHES environment model ready.

### 1.4 Prepare permissions (GitHub + Service Principal)

1. Ensure APIOps operators have GHES repo write access and PR permissions.
2. Ensure environment maintainers are assigned for `dev` and `uat`.
3. Confirm service principal has correct APIM scope (not over-privileged subscription-wide if avoidable).
4. Run one permission validation matrix:
   - Trigger extractor on dev
   - Trigger publish on dev
   - Trigger publish on uat

**Output:** End-to-end permission path validated.

---

## 2) Basic Operations – VS Code (Pending)

### 2.1 Install/verify Git client in VS Code

1. Verify Git executable is discoverable in VS Code terminal: `git --version`.
2. Choose auth mode (SSH recommended for GHES if BMW policy permits; otherwise HTTPS with PAT).
3. Test connectivity to GHES remote.
4. Confirm pull/push works on a test branch.

**Output:** Reliable Git client workflow from VS Code.

### 2.2 Clone APIOps baseline repository

1. Clone the BMW APIOps repo from GHES.
2. Open folder in VS Code.
3. Validate baseline structure exists (workflows, configuration files, artifacts folders).
4. Run a read-only sanity check (no deployment): ensure YAML files parse without syntax errors.

**Output:** Local APIOps working copy ready.

---

## 3) Basic Operations – GitHub (Pending)

### 3.1 Clone / Commit / Pull / Push workflow

1. Create feature branch from `main`.
2. Make a small non-production documentation change.
3. Commit with descriptive message.
4. Pull/rebase from `main`.
5. Push branch to GHES.

**Output:** Team confirms standard Git loop.

### 3.2 Pull Request workflow

1. Open PR from feature branch to `main`.
2. Add reviewers and required labels.
3. Verify workflow checks pass.
4. Resolve comments and update branch.
5. Merge according to BMW policy (squash/rebase/merge commit).

**Output:** PR-based change control validated.

---

## 4) Git Flow Operations (Pending)

### 4.1 Define and enforce branch strategy

1. Adopt branch model for BMW APIOps:
   - `main` = source of truth for dev-ready artifacts
   - `release/*` = candidate for uat promotion
2. Configure branch protections accordingly.
3. Document who can create/approve `release/*` promotions.

**Output:** Agreed branch governance.

### 4.2 Publish from main to apim dev

1. Merge approved PR to `main`.
2. Trigger `.github/workflows/run-publisher.yaml` with default commit-based mode.
3. Verify publisher job uses `dev` environment in reusable workflow.
4. Validate changes in `apim dev` portal.

**Output:** `main -> apim dev` promotion path proven.

### 4.3 Publish from release branch to apim uat

1. Create `release/<version>` from a validated `main` commit.
2. Open/approve PR to merge release artifacts per BMW process.
3. Trigger publisher for `uat` (environment must map to `apim uat`; use env-specific config YAML).
4. Record deployment tag/commit for rollback tracking.
5. Simulate rollback by re-publishing previous known-good release commit.

**Output:** Controlled `release -> apim uat` and rollback process.

---

## 5) APIOps – Extractor Operations (Pending)

### 5.1 Extract all artifacts from apim dev

1. Trigger `.github/workflows/run-extractor-ghes.yaml`.
2. Select `CONFIGURATION_YAML_PATH = Extract All APIs`.
3. Select API spec format (e.g., `OpenAPIV3Yaml`).
4. Confirm output artifacts are generated under `apimartifacts` (or BMW-specific target folder if agreed).
5. Verify PR is auto-created by extractor job.

**Output:** Full extraction PR from `apim dev`.

### 5.2 Extract partial artifacts with configuration.extractor.yaml

1. Update `configuration.extractor.yaml` to include only intended filters.
2. Trigger extractor workflow with `CONFIGURATION_YAML_PATH = configuration.extractor.yaml`.
3. Validate PR includes only scoped artifacts.

**Output:** Selective extraction with controlled scope.

### 5.3 Filter sub-tasks (run one by one, same process)

For each filter below, apply only that filter in `configuration.extractor.yaml`, run extractor, and verify output:

1. `apiNames` (API-level control)
2. `backendNames` (backend isolation)
3. `namedValueNames` (secret/named value scope)
4. `productNames` (product governance)
5. `subscriptionNames` (subscription scope)
6. `policyFragmentNames` (shared policy fragments)

**Output:** Evidence that each filter works independently.

### 5.4 Create PR to main branch

1. Review extractor-generated PR for unwanted drift.
2. Request architecture/security review where needed.
3. Merge only after validation in dev test checklist.

**Output:** Approved GitOps extraction flow into `main`.

---

## 6) APIOps – Publisher Operations (Pending)

### 6.1 Create configuration.uat.yaml

1. Copy structure from environment-specific config pattern.
2. Set APIM service-specific overrides for `apim uat`.
3. Replace sensitive values with tokens/secrets, not plain text.
4. Validate YAML syntax.

**Output:** `configuration.uat.yaml` ready for uat publishing.

### 6.2 Publish main branch to apim uat

1. Confirm `main` is stable and approved for promotion.
2. Trigger publisher workflow for `uat` with correct configuration YAML.
3. Verify deployment status in Actions and APIM portal.
4. Run smoke test of key APIs in uat.

**Output:** Standard promotion to uat verified.

### 6.3 Publish release branch to apim uat (rollback/version simulation)

1. Pick a tagged release candidate commit.
2. Publish that commit to `apim uat`.
3. Publish an older known-good commit to simulate rollback.
4. Record results in deployment log.

**Output:** Rollback/versioning scenario validated.

### 6.4 Modify locally in VS Code and publish to apim dev

1. Change one API artifact (e.g., policy/routing) in branch.
2. Open PR and merge to `main`.
3. Trigger publish to dev.
4. Validate behavior in `apim dev`.

**Output:** Developer inner loop validated end-to-end.

### 6.5 Create dev-only publish script/workflow path

1. Add a dedicated workflow input or wrapper for `dev` only.
2. Restrict to `dev` environment secrets.
3. Add clear naming for low-risk iteration usage.
4. Validate with one test deployment.

**Output:** Fast iteration path for dev.

### 6.6 Create uat-only publish script/workflow path

1. Add dedicated `uat` deployment path with environment approval gate.
2. Require release branch/tag input.
3. Add runbook notes for approvers.
4. Validate controlled deployment.

**Output:** Governed deployment path for uat.

---

## 7) APIM Code Validation (Pending)

### 7.1 Add REST API + policy test (dev)

1. Add new sample REST API artifact in repo.
2. Add inbound/outbound policy updates.
3. Publish to dev.
4. Validate API import, operations, and runtime behavior.

**Output:** Create-flow validated.

### 7.2 Revise REST API + policy test (dev)

1. Update operation and policy logic for existing API.
2. Publish incremental change.
3. Validate revision/version behavior and backward compatibility.

**Output:** Update-flow validated.

### 7.3 Delete REST API + policy test (dev)

1. Remove target API artifacts from repo.
2. Publish to dev.
3. Confirm API/policy deletion behavior matches APIOps expectations.

**Output:** Deletion and cleanup behavior validated.

### 7.4 Maintain policy fragments test (dev)

1. Create/update shared policy fragment.
2. Reference fragment from multiple APIs.
3. Publish and validate all dependent APIs.

**Output:** Shared governance fragment lifecycle validated.

### 7.5 GraphQL API test (dev, TBD)

1. Identify a BMW GraphQL candidate (synthetic or pass-through).
2. Model artifact structure and policy.
3. Extract/publish cycle in dev.
4. Validate schema operations and policy behavior.

**Output:** Feasibility conclusion and implementation notes.

---

## 8) Experience & Knowledge Capture (Pending item only)

### 8.1 Other support topics (to be extended)

1. Keep a running backlog of APIOps issues/improvements.
2. Classify each item: process/tooling/security/performance.
3. Assign owner and target date.
4. Review weekly and convert mature items into runbook updates.

**Output:** Continuous improvement queue.

---

## 9) Documentation Deliverables (Pending)

### 9.1 GitHub / Git Flow operations manual

1. Capture branch strategy and PR policy.
2. Document promotion path: `main -> apim dev`, `release/* -> apim uat`.
3. Include rollback procedure.

### 9.2 Extractor operations manual

1. Document full extraction flow.
2. Document each partial extraction filter and examples.
3. Add troubleshooting and validation checklist.

### 9.3 Publisher operations manual

1. Document environment-specific publish process.
2. Include required GHES environment secrets.
3. Include approval gates and rollback method.

### 9.4 Code/Scripts manual

1. Document workflow files, inputs, and expected outputs.
2. Add change management rules for workflow/script edits.
3. Provide run examples for dev and uat.

### 9.5 Support/Troubleshooting records (remaining documentation updates)

1. Consolidate solved issues and known limitations.
2. Add “symptom -> cause -> fix -> prevention” format.
3. Keep records versioned in repo.

**Output:** Complete APIOps operational documentation set.

---

## Recommended execution order (BMW)

1. Environment preparation
2. Git/GitHub basics
3. Git flow enforcement
4. Extractor validations
5. Publisher validations (dev, then uat)
6. APIM lifecycle tests (add/update/delete/fragments/GraphQL)
7. Documentation closure

This order reduces deployment risk and ensures governance is in place before uat promotion.
