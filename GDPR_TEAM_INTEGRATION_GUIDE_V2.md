# GDPR Backend — Executive Summary Integration Guide (v2)

> **For:** GDPR Rails Backend Team  
> **From:** HIPAA-LLM-APIs Team  
> **Date:** March 28, 2026  
> **Version:** 2.0 — Adds `hipaa_score` in V2, suggestions endpoint, `is_form`, and organization name  
> **Status:** Python service is implemented and ready. **Requires 4 changes on the Rails side.**

---

## TL;DR — What the GDPR Team Needs to Do

| # | Change | Where | Effort | Priority |
|---|---|---|---|---|
| 1 | Add `hipaa_score` + `risk_percentages` to V2 `_row` partial | `_row.json.jbuilder` | 5 min | 🔴 **REQUIRED** |
| 2 | Add `organization_name` to V2 `_row` partial | `_row.json.jbuilder` | 2 min | 🟢 Nice-to-have |
| 3 | Create new executive summary context endpoint with suggestions | New controller action + route | 1-2 hours | 🟡 Phase 2 |
| 4 | Call LLM API from new endpoint + cache invalidation on re-scan | Controller + Sidekiq job | 30 min | 🔴 **REQUIRED** |

**Change 1 is blocking** — without `hipaa_score` in the V2 response, the executive summary cannot show the official compliance score.

---

## Background

We have a Python-based LLM service that generates GDPR executive summaries from scan data. It accepts the raw `rule_wise` API pages, strips ~97.7% of the data (code/matched_data), and feeds a compact payload to GPT-OSS 20B for narrative generation.

**The Python service is fully implemented.** This guide covers what the Rails backend needs to change/add.

---

## Change 1: Add `hipaa_score` to V2 `_row` Partial (REQUIRED)

### Why
The V2 `_row` partial (used in `rule_wise`, `file_wise`, `dashboard`, `report`) does **not** include `hipaa_score` or `risk_percentages`. These are only in the V1 `_row`. The `hipaa_score` is the **official compliance score** computed from YAML pattern weights — it's displayed on the frontend dashboard.

**Our Python service uses this score directly. It does NOT compute its own score.** This ensures the executive summary never contradicts the dashboard.

### What to Change

**File:** `app/views/api/v2/user_uploads/_row.json.jbuilder`

**Current:**
```ruby
json.id                       user_upload.id
json.user_id                  user_upload.user_id
json.name                     user_upload.ios? && user_upload.project_name.present? ? user_upload.project_name.titleize : user_upload.file.filename.to_s.gsub('.apk', '').gsub('.zip', '').humanize
json.upload_type              user_upload.upload_type
json.platform                 user_upload.platform
json.environment              user_upload.environment
json.created_at               user_upload.created_at
json.project_name             user_upload.project_name
json.project_identifier       user_upload.project_identifier
json.is_form                  user_upload.is_form

calculator = HipaaRiskScoreCalculator.new(user_upload.hipaa_risk_scores(@rules2))
calculator.calculate
json.severity_counts user_upload.dashboard_report_as_hash(@rules2)[:severity_counts]
json.hipaa_risk_scores        calculator.generate_score_summary
```

**Add these lines at the end (after `json.hipaa_risk_scores`):**
```ruby
# ── Executive Summary fields (v2.1) ──────────────────────────
# hipaa_score: Official compliance score (used by LLM executive summary).
# Must match the score shown on the frontend dashboard.
json.hipaa_score              user_upload.hipaa_score
json.high_risk_percentage     user_upload.high_risk_percentage.round(2)
json.medium_risk_percentage   user_upload.medium_risk_percentage.round(2)
json.low_risk_percentage      user_upload.low_risk_percentage.round(2)
json.no_risk_percentage       user_upload.no_risk_percentage.round(2)
```

**Full file after change:**
```ruby
json.id                       user_upload.id
json.user_id                  user_upload.user_id
json.name                     user_upload.ios? && user_upload.project_name.present? ? user_upload.project_name.titleize : user_upload.file.filename.to_s.gsub('.apk', '').gsub('.zip', '').humanize
json.upload_type              user_upload.upload_type
json.platform                 user_upload.platform
json.environment              user_upload.environment
json.created_at               user_upload.created_at
json.project_name             user_upload.project_name
json.project_identifier       user_upload.project_identifier
json.is_form                  user_upload.is_form

calculator = HipaaRiskScoreCalculator.new(user_upload.hipaa_risk_scores(@rules2))
calculator.calculate
json.severity_counts user_upload.dashboard_report_as_hash(@rules2)[:severity_counts]
json.hipaa_risk_scores        calculator.generate_score_summary

# ── Executive Summary fields (v2.1) ──────────────────────────
json.hipaa_score              user_upload.hipaa_score
json.high_risk_percentage     user_upload.high_risk_percentage.round(2)
json.medium_risk_percentage   user_upload.medium_risk_percentage.round(2)
json.low_risk_percentage      user_upload.low_risk_percentage.round(2)
json.no_risk_percentage       user_upload.no_risk_percentage.round(2)
```

### Impact
- **Frontend:** No breaking change — new fields are additive. The frontend already uses `severity_counts` and `hipaa_risk_scores` from V2. The new fields are ignored by existing consumers.
- **Caching:** The `rule_wise_cache_key` includes user_id + page, so you may need to bust the cache after deploying this change (e.g., `Rails.cache.clear` or wait for the 1-month TTL).

---

## Change 2: Add `organization_name` to V2 `_row` Partial (Nice-to-have)

### Why
The executive summary header benefits from referencing the organization by name (e.g., "GDPR Compliance Executive Summary for **Acme Corp** — Pharmeasy APK").

### What to Change

**File:** `app/views/api/v2/user_uploads/_row.json.jbuilder`

**Add this line** (after the hipaa_score lines from Change 1):
```ruby
json.organization_name        user_upload.user&.organization&.name
```

### Impact
- Additive field, no breaking changes.
- Returns `null` if the user has no organization (individual account) — the Python service handles `null` gracefully.

---

## Change 3: Create Executive Summary Context Endpoint with Suggestions (Phase 2)

### Why
The suggestions API (`GET /api/v1/suggestions`) provides per-rule regulatory expectations and per-subrule remediation code snippets. Including these in the executive summary makes the "Remediation Priorities" section dramatically more actionable — the LLM can tell the DPO/CTO *what the regulation requires* and *what code changes to make*, not just *what was found*.

### New Route

```ruby
# config/routes.rb
namespace :api do
  namespace :v2 do
    resources :user_uploads, only: [] do
      member do
        get :executive_summary
      end
    end
  end
end
# => GET /api/v2/user_uploads/:id/executive_summary
```

### New Controller Action

```ruby
# app/controllers/api/v2/user_uploads_controller.rb

def executive_summary
  @user_upload = current_user.user_uploads.find(params[:id])
  @rules = UserUpload::HIPAA_RULES.deep_dup.paginate(page: nil, per_page: 1)
  @rules2 = UserUpload::HIPAA_RULES.deep_dup

  # 1. Collect ALL rule_wise pages (reuse existing logic)
  all_rules = UserUpload::HIPAA_RULES.deep_dup
  raw_pages = []

  all_rules.each_with_index do |rule, index|
    page_num = index + 1
    paginated_rules = UserUpload::HIPAA_RULES.deep_dup.paginate(page: page_num, per_page: 1)

    # Build the page data structure (matching rule_wise format)
    page_data = {
      user_upload: build_row_data(@user_upload, paginated_rules),
      pagination: {
        current_page: paginated_rules.current_page,
        next_page: paginated_rules.next_page,
        prev_page: paginated_rules.previous_page,
        total_pages: paginated_rules.total_pages,
        total_entries: paginated_rules.total_entries
      }
    }

    raw_pages << page_data
  end

  # 2. Collect suggestions for rules that have findings
  suggestions_by_rule = collect_suggestions_for_upload(@user_upload)

  # 3. Forward to LLM API
  response = forward_to_llm_api(raw_pages, @user_upload.id, suggestions_by_rule)

  render json: response, status: response["status"] == "success" ? :ok : :internal_server_error
end

private

def build_row_data(upload, paginated_rules)
  rules2 = UserUpload::HIPAA_RULES.deep_dup
  calculator = HipaaRiskScoreCalculator.new(upload.hipaa_risk_scores(rules2))
  calculator.calculate

  {
    id: upload.id,
    user_id: upload.user_id,
    name: upload.ios? && upload.project_name.present? ? upload.project_name.titleize : upload.file.filename.to_s.gsub('.apk', '').gsub('.zip', '').humanize,
    upload_type: upload.upload_type,
    platform: upload.platform,
    environment: upload.environment,
    created_at: upload.created_at,
    project_name: upload.project_name,
    project_identifier: upload.project_identifier,
    is_form: upload.is_form,
    severity_counts: upload.dashboard_report_as_hash(rules2)[:severity_counts],
    hipaa_risk_scores: calculator.generate_score_summary,
    hipaa_score: upload.hipaa_score,
    high_risk_percentage: upload.high_risk_percentage.round(2),
    medium_risk_percentage: upload.medium_risk_percentage.round(2),
    low_risk_percentage: upload.low_risk_percentage.round(2),
    no_risk_percentage: upload.no_risk_percentage.round(2),
    organization_name: upload.user&.organization&.name,
    analyzed_results: upload.cvss_reports_as_hash(paginated_rules)
  }
end

def collect_suggestions_for_upload(upload)
  platform = upload.platform
  suggestions_by_rule = {}

  # Get rules that have findings (non-zero analyzed results)
  UserUpload::HIPAA_RULES.each do |rule|
    rule_id = rule[:rule_id]
    rule_results = upload.analyzed_results
                         .where(rule_name: rule_id)
                         .where.not(user_review_status: "ignore")

    next if rule_results.empty?

    # Fetch suggestions for this rule
    rule_suggestions = Suggestion.where(rule_id: rule_id, platform: platform)
    next if rule_suggestions.blank?

    # Extract expectation (from the "All" subrule)
    expectation_record = rule_suggestions.find_by(subrule_id: "All")
    expectation_text = ""
    if expectation_record&.expectations_from_hipaa&.body.present?
      # Strip HTML to plain text for LLM consumption
      expectation_text = ActionController::Base.helpers.strip_tags(
        expectation_record.expectations_from_hipaa.body.to_html
      ).squish
    end

    # Extract per-subrule suggestions (truncated for payload size)
    subrule_suggestions = []
    grouped = rule_suggestions.where.not(subrule_id: "All").group_by(&:subrule_id)
    grouped.each do |subrule_id, items|
      snippets = items.map do |s|
        snippet_text = ActionController::Base.helpers.strip_tags(s.code_snippet.to_s).squish
        {
          snippet: snippet_text.truncate(500),
          pattern: s.patterns
        }
      end

      subrule_suggestions << {
        subrule_id: subrule_id,
        suggestions: snippets
      }
    end

    suggestions_by_rule[rule_id] = {
      expectation: expectation_text,
      subrules: subrule_suggestions
    }
  end

  suggestions_by_rule
end

def forward_to_llm_api(raw_pages, user_upload_id, suggestions_by_rule = {})
  require 'net/http'
  require 'uri'
  require 'json'

  llm_api_url = ENV.fetch("LLM_API_URL", "http://localhost:5000")
  uri = URI("#{llm_api_url}/api/v1/executive-summary/from-raw")

  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = uri.scheme == "https"
  http.read_timeout = 120  # LLM can take up to 2 minutes

  request = Net::HTTP::Post.new(uri.path, {
    "Content-Type" => "application/json",
    "X-Compliance-Framework" => "gdpr"
  })

  payload = {
    user_upload_id: user_upload_id,
    raw_pages: raw_pages,
    force_regenerate: params[:force_regenerate] == "true"
  }

  # Include suggestions if collected
  if suggestions_by_rule.present?
    payload[:raw_pages].first[:suggestions_by_rule] = suggestions_by_rule
  end

  request.body = payload.to_json

  response = http.request(request)
  JSON.parse(response.body)
rescue StandardError => e
  Rails.logger.error("Executive summary LLM API call failed: #{e.message}")
  { status: "error", message: "Failed to generate executive summary: #{e.message}" }
end
```

### Alternative: Simpler Approach (Without Building Pages Manually)

If you prefer to reuse the existing jbuilder views rather than building the hash manually, you can render the jbuilder templates and parse them:

```ruby
def executive_summary
  @user_upload = current_user.user_uploads.find(params[:id])
  @rules2 = UserUpload::HIPAA_RULES.deep_dup

  raw_pages = []
  total_rules = UserUpload::HIPAA_RULES.length

  (1..total_rules).each do |page_num|
    @rules = UserUpload::HIPAA_RULES.deep_dup.paginate(page: page_num, per_page: 1)

    # Render the existing rule_wise jbuilder template to JSON
    page_json = render_to_string(
      template: "api/v2/user_uploads/rule_wise",
      formats: [:json]
    )
    raw_pages << JSON.parse(page_json)
  end

  # Collect suggestions
  suggestions_by_rule = collect_suggestions_for_upload(@user_upload)

  # Forward to LLM API
  response = forward_to_llm_api(raw_pages, @user_upload.id, suggestions_by_rule)

  render json: response, status: response["status"] == "success" ? :ok : :internal_server_error
end
```

> **This simpler approach reuses your existing cached jbuilder views**, so you get the performance benefit of the `rule_wise_cache_key`. It's the recommended path if you want minimal new code.

---

## Change 4: Call LLM API + Cache Invalidation (REQUIRED)

### 4a. The LLM API Call

See the `forward_to_llm_api` method in Change 3 above. Key points:
- POST to `/api/v1/executive-summary/from-raw`
- Timeout: 120 seconds (LLM generation takes 15-45 seconds typically)
- The response JSON has `{ "status": "success", "executive_summary": "...markdown..." }`

### 4b. Cache Invalidation After Re-Scan

When a scan is re-run for a `UserUpload` (e.g., in `ReportGenerationJob`), call our cache invalidation endpoint so the next executive summary request generates fresh output:

```ruby
# Add this at the end of ReportGenerationJob#perform, after all rules are processed
# OR in UploadCodebaseJob#perform after the codebase upload completes

def invalidate_executive_summary_cache(user_upload_id)
  require 'net/http'
  require 'uri'

  llm_api_url = ENV.fetch("LLM_API_URL", "http://localhost:5000")
  uri = URI("#{llm_api_url}/api/v1/executive-summary/invalidate-cache")

  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = uri.scheme == "https"

  request = Net::HTTP::Post.new(uri.path, {
    "Content-Type" => "application/json"
  })
  request.body = { user_upload_id: user_upload_id }.to_json

  http.request(request)
rescue StandardError => e
  Rails.logger.warn("Failed to invalidate LLM executive summary cache: #{e.message}")
end
```

**Where to call it:** In `report_generation_job.rb`, after the line that marks the report as complete:
```ruby
# Existing code that sets completed_rules_count / status
user_upload.update(status: :report_generated)

# ADD: Invalidate executive summary cache
invalidate_executive_summary_cache(user_upload.id)
```

---

## API Contract — What the Python Service Accepts

### Endpoint: `POST /api/v1/executive-summary/from-raw` (RECOMMENDED)

**Request Body:**
```json
{
  "user_upload_id": 88,
  "raw_pages": [
    {
      "user_upload": {
        "id": 88,
        "name": "Pharmeasy",
        "platform": "apk",
        "environment": "app",
        "upload_type": "Healthcare",
        "created_at": "2026-03-28T...",
        "is_form": false,
        "hipaa_score": 72.4,
        "high_risk_percentage": 32.5,
        "medium_risk_percentage": 41.2,
        "low_risk_percentage": 18.1,
        "no_risk_percentage": 8.2,
        "organization_name": "Acme Corp",
        "severity_counts": { "critical_risk": 0, "high_risk": 8, "medium_risk": 15, "low_risk": 5, "no_risk": 2 },
        "hipaa_risk_scores": { "total_risk_score": 18.5, "cvss_risk_score": 24.3, "...": "..." },
        "analyzed_results": [{ "rule_id": "lawfulness_fairness_transparency", "...": "..." }]
      },
      "pagination": { "current_page": 1, "total_pages": 7 },
      "suggestions_by_rule": {
        "lawfulness_fairness_transparency": {
          "expectation": "Under GDPR Article 6, all personal data processing must have a lawful basis...",
          "subrules": [
            {
              "subrule_id": "lawful_basis_art6",
              "suggestions": [
                { "snippet": "Implement a ConsentManager class that tracks user consent...", "pattern": ["consent_regex"] }
              ]
            }
          ]
        }
      }
    },
    { "user_upload": { "...page 2..." } },
    { "user_upload": { "...page 3..." } }
  ],
  "force_regenerate": false
}
```

### Key Fields the Python Service Reads from `user_upload`:

| Field | Source | Required | Notes |
|---|---|---|---|
| `hipaa_score` | `user_upload.hipaa_score` | **YES** | Official score. **No fallback computation.** |
| `severity_counts` | `dashboard_report_as_hash[:severity_counts]` | YES | Already in V2 _row |
| `hipaa_risk_scores` | `HipaaRiskScoreCalculator#generate_score_summary` | YES | Already in V2 _row |
| `is_form` | `user_upload.is_form` | YES | Already in V2 _row |
| `high_risk_percentage` | `user_upload.high_risk_percentage.round(2)` | Recommended | **NEW — Change 1** |
| `medium_risk_percentage` | `user_upload.medium_risk_percentage.round(2)` | Recommended | **NEW — Change 1** |
| `low_risk_percentage` | `user_upload.low_risk_percentage.round(2)` | Recommended | **NEW — Change 1** |
| `no_risk_percentage` | `user_upload.no_risk_percentage.round(2)` | Recommended | **NEW — Change 1** |
| `organization_name` | `user_upload.user&.organization&.name` | Optional | **NEW — Change 2** |
| `analyzed_results` | `cvss_reports_as_hash(paginated_rules)` | YES | Already in rule_wise |

### Key Field at Page Level:

| Field | Source | Required | Notes |
|---|---|---|---|
| `suggestions_by_rule` | `collect_suggestions_for_upload` | Optional (Phase 2) | **NEW — Change 3** |

### Response:
```json
{
  "status": "success",
  "executive_summary": "## Executive Overview\n\nThe Pharmeasy Healthcare APK (Acme Corp) ...",
  "metadata": {
    "user_upload_id": 88,
    "cached": false,
    "generation_time_ms": 22000,
    "model": "gpt-oss-20b-Q4_K_M",
    "payload_was_reduced": true,
    "rules_count": 7,
    "raw_pages_count": 7,
    "context_payload_chars": 5200
  }
}
```

---

## Endpoint Summary

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/v1/executive-summary/from-raw` | POST | ⭐ **RECOMMENDED** — Generate summary from raw rule_wise API pages |
| `/api/v1/executive-summary` | POST | Generate summary from pre-computed context payload |
| `/api/v1/executive-summary/invalidate-cache` | POST | Clear cached summaries for a user_upload_id |
| `/api/v1/executive-summary/health` | GET | Health check (LLM + Redis status) |

---

## Error Handling

| Status | Meaning | Action |
|---|---|---|
| `200` | Success — `executive_summary` field contains Markdown text | Display to user |
| `400` | Bad request — missing or invalid fields | Check request body |
| `500` | LLM generation failed or internal error | Retry or show error |

### Missing `hipaa_score` Behavior
If `hipaa_score` is not in the `user_upload` object, the Python service will:
1. Log a warning: `"hipaa_score not found in API response"`
2. Set `overall_compliance_score` to `"unavailable"` in the LLM context
3. The LLM will note that the official compliance score is not available
4. **The summary will still be generated** — just without the score headline

This is a degraded mode. **Change 1 is required to avoid it.**

---

## What NOT to Send

❌ Do NOT send raw source code files  
❌ Do NOT send `matched_data` arrays (the Python service strips them anyway, but they waste bandwidth)  
❌ Do NOT send `file_wise` API responses (only `rule_wise` is supported)  
❌ Do NOT compute your own compliance score in the payload — the Python service reads `hipaa_score` directly  

✅ DO include all 7 rule_wise pages (or as many as have data)  
✅ DO include the `hipaa_score` field (Change 1)  
✅ DO include `suggestions_by_rule` when available (Change 3)  
✅ DO call cache invalidation after re-scan (Change 4)  

---

## Implementation Order

### Phase 1 (Ship immediately — required for v1 executive summaries)
1. ✅ **Change 1:** Add `hipaa_score` + `risk_percentages` to V2 `_row` partial (5 min)
2. ✅ **Change 2:** Add `organization_name` to V2 `_row` partial (2 min)
3. ✅ **Change 4:** Add `executive_summary` controller action (using the simpler `render_to_string` approach) + cache invalidation in `ReportGenerationJob` (30 min)

### Phase 2 (Enhances remediation quality)
4. ✅ **Change 3:** Add `collect_suggestions_for_upload` and include `suggestions_by_rule` in the payload (1-2 hours)

---

## Testing

### Quick Test (After Change 1 + 4)

```bash
# 1. Verify hipaa_score appears in rule_wise response
curl -s http://localhost:3000/api/v2/user_uploads/88/rule_wise?page=1 \
  -H "Authorization: Bearer YOUR_TOKEN" | jq '.user_upload.hipaa_score'
# Should return a number (e.g., 72.4), NOT null

# 2. Test the executive summary endpoint
curl -s http://localhost:3000/api/v2/user_uploads/88/executive_summary \
  -H "Authorization: Bearer YOUR_TOKEN" | jq '.status'
# Should return "success"

# 3. Verify cache invalidation
curl -X POST http://LLM_API_HOST:5000/api/v1/executive-summary/invalidate-cache \
  -H "Content-Type: application/json" \
  -d '{"user_upload_id": 88}'
# Should return {"status": "success", "deleted_keys": N}
```

### Verify Suggestions (After Change 3)

```bash
# Test with suggestions included
curl -s http://localhost:3000/api/v2/user_uploads/88/executive_summary \
  -H "Authorization: Bearer YOUR_TOKEN" | jq '.executive_summary' | head -20
# The "Remediation Priorities" section should now include specific guidance
# (e.g., "Implement ConsentManager class..." instead of just "Fix consent issues")
```

---

## Questions?

Reach out to the HIPAA-LLM-APIs team. The Python service is deployed on the `deployment` branch and ready for integration testing.
