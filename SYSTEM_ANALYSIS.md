# AI Model Ranking System - Comprehensive Analysis & Debugging Report

**Analysis Date**: December 23, 2025  
**System Version**: v8.5  
**Analyzed By**: GitHub Copilot

---

## EXECUTIVE SUMMARY

The system implements a multi-dimensional AI model ranking engine that fetches live data from api.zeroeval.com and computes rankings across 8 distinct tabs. After thorough analysis, **12 critical flaws** have been identified that compromise data integrity, ranking accuracy, and system robustness.

**Key Findings**:
- ✅ **Working**: Confidence variance (52.5%-92.5%), multimodal detection (51 image models), JSON parsing
- ⚠️ **Critical Issues**: Simulated data injection, incomplete modality logic, context window normalization, missing video detection
- 🔴 **Severe Bugs**: Image/video rankings ignore final_score component, value ranking penalizes high performers

---

## I. DATA PIPELINE ARCHITECTURE

### 1.1 End-to-End Flow

```
┌─────────────────┐
│  API Fetch      │ NetworkClient::Get()
│ api.zeroeval.com│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  JSON Parsing   │ json::parse() in Run()
│  Validation     │ Check name, org, scores
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Signal Addition │ AddSignal() - gpqa_score, average_score
│ Base Scoring    │ Weighted: 0.50 + 0.40
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ ComputeAggregates│ Lines 177-217
│  - final_score   │ Weighted average of signals
│  - confidence_1  │ Base confidence calculation
│  - variance      │ Score consistency metric
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Enrich (KB)     │ Lines 383-519 - KnowledgeBase::Enrich()
│  - Modality      │ Image/Video/Text detection
│  - Pricing       │ Parse input_price (per-token → per-1M)
│  - Metrics       │ coding_score, creative_score, speed
│  - Context       │ context_length, release_date
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│RecalculateConfid│ Lines 219-257
│  - confidence_2  │ Score-quality bonuses (+15/+10/-10%)
│  - enterprise    │ +5% for OpenAI/Anthropic/Google/MS
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ ComputeRankings │ Lines 259-307
│  8 Tab Rankings  │ Overall, Value, Coding, Image, Video,
│                  │ Speed, Confidence, Enterprise
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Filter/Sort     │ Modality-based filtering
│ Export          │ CSV (3 files), JSON, HTML
└─────────────────┘
```

### 1.2 Score Normalization

**Input Scores** (from API):
- `gpqa_score`: [0.0, 1.0] - Primary benchmark
- `average_score`: [0.0, 1.0] - Fallback metric
- `coding_score`: [0.0, 1.0] or string - Optional
- `creative_score`: [0.0, 1.0] - Often missing
- `input_price`: $/1M tokens or per-token (requires conversion)
- `throughput`: tokens/sec - Highly variable
- `context_length`: Raw token count - Needs normalization

**Normalization Methods**:
```cpp
// Weighted Signal Aggregation (Line 189)
final_score = weighted_sum / total_weight  // Already normalized [0,1]

// Context Window (Line 483) - ❌ MISSING NORMALIZATION
metrics.context_window = ctx_len_double;  // Raw: 4096-200000+

// Speed (Line 291) - ✅ Normalized to [0,1]
speed_norm = clamp(tokens_per_sec / 150.0, 0.0, 1.0);

// Price Factor (Line 265) - ✅ Logarithmic
price_factor = 1.0 / (1.0 + price_input_1m / 10.0);
```

---

## II. CONFIDENCE SCORE COMPUTATION

### 2.1 Two-Stage Calculation

**Stage 1: ComputeAggregates() - Lines 177-217**
```cpp
conf = 50.0;                    // Base
conf += signals.size() * 10.0;  // Signal count: +10% per signal
conf += 5.0 (if recent);        // Recency: ≤30 days
conf += 10.0 (if versatile);    // Multi-category
conf -= variance * 50.0;        // Consistency penalty
// Result: [10, 99] range
```

**Stage 2: RecalculateConfidence() - Lines 219-257** (After enrichment)
```cpp
conf = 50.0;                    // Re-initialize
conf += signals.size() * 10.0;
conf += 5.0 / 2.5 (recency);    // ≤30 days / ≤90 days
conf += 10.0 (if versatile);    // Updated with enriched metrics
conf += 15/10/5/-10 (score);    // NEW: Quality bonuses
conf -= variance * 50.0;
conf += 5.0 (if enterprise);    // NEW: Enterprise bonus
// Result: [15, 99] range
```

**🔴 ISSUE #1: Duplicate Calculation**
- Confidence is calculated TWICE with similar logic
- First calculation in `ComputeAggregates()` is overwritten
- `confidence_reason` string is only built in first pass (not updated)

**Observed Output**: ✅ Working (52.5%-92.5% range)

### 2.2 Variance Penalty

```cpp
// Lines 195-196, 246-249
double sq_sum = inner_product(scores.begin(), scores.end(), scores.begin(), 0.0,
    plus<double>(), [mean](double x, double y) { return (x - mean) * (y - mean); });
double variance = sqrt(sq_sum / scores.size());
```

**Formula**: Standard Deviation = √(Σ(xi - μ)² / n)

**Impact**: Models with inconsistent signals across benchmarks get confidence penalty
- High variance (0.1-0.2) → -5% to -10% confidence
- Low variance (0.0-0.05) → Minimal penalty

**✅ Correct Implementation**

---

## III. RANKING FORMULAS BY TAB

### 3.1 Overall Ranking (Line 265-271)

```cpp
ranks.overall = (final_score * 0.40)           // Core performance
              + (coding_score * 0.20)          // Programming ability
              + (creative_score * 0.15)        // Creativity
              + (confidence / 100 * 0.15)      // Reliability
              + (price_factor * 0.10);         // Affordability
```

**Weights**: 40% + 20% + 15% + 15% + 10% = 100%

**Eligibility**: All models (no filtering)

**✅ Formula Correctness**: Balanced multi-criteria scoring

**⚠️ Issue**: No penalty for missing data (creative_score defaults to 0.0)

---

### 3.2 Value Ranking (Line 273-278)

```cpp
if (price ≤ 0.0) {
    ranks.value = final_score * 1000.0;  // Free models
} else {
    price_factor = log10(price + 1.0);
    ranks.value = (final_score > 0.5) ? (final_score - 0.5) / (price_factor + 0.1) : 0.0;
}
```

**🔴 ISSUE #2: Penalizes High Performers**
- Formula subtracts 0.5 from score: `(score - 0.5) / price_factor`
- A model with score=0.9 and price=$3 gets: (0.9 - 0.5) / 0.58 = 0.69
- A model with score=0.6 and price=$3 gets: (0.6 - 0.5) / 0.58 = 0.17
- **Problem**: High-scoring expensive models are penalized vs cheap mediocre ones

**Proposed Fix**:
```cpp
ranks.value = (final_score * final_score) / (price_factor + 0.1);
// Quadratic score rewards excellence, log price dampens cost impact
```

---

### 3.3 Coding Ranking (Line 280-283)

```cpp
ranks.coding = (coding_score * 0.6)            // Coding benchmark
             + (reasoning_score * 0.2)         // Problem-solving
             + (context_window * 0.1)          // Context handling
             + (confidence / 100 * 0.1);       // Reliability
```

**🔴 ISSUE #3: Context Window Not Normalized**
- `context_window` is raw token count: 4096, 128000, 200000+
- Adding raw values (128000) to normalized scores (0.6, 0.2, 0.1) breaks scale
- A model with 200K context gets +20000 added to its coding rank!

**Example**:
```
Model A: coding=0.8, reasoning=0.9, context=128000, conf=0.85
rank = 0.48 + 0.18 + 12800 + 0.085 = 12800.745  ❌ BROKEN

Model B: coding=0.9, reasoning=0.9, context=4096, conf=0.90
rank = 0.54 + 0.18 + 409.6 + 0.090 = 410.410     ❌ BROKEN
```

**Proposed Fix**:
```cpp
double ctx_norm = std::clamp(context_window / 200000.0, 0.0, 1.0);
ranks.coding = (coding_score * 0.6) + (reasoning_score * 0.2) 
             + (ctx_norm * 0.1) + (conf_factor * 0.1);
```

**Eligibility**: All models (should filter text-only models?)

---

### 3.4 Image Ranking (Line 285-289)

```cpp
speed_norm = clamp(tokens_per_sec / 150.0, 0.0, 1.0);
ranks.image = (creative_score * 0.6) + (speed_norm * 0.1) + (confidence / 100 * 0.1);
if (modalities.count(Image) == 0) ranks.image = 0;
```

**Weights**: 60% + 10% + 10% = 80% (❌ Missing 20%)

**🔴 ISSUE #4: Missing Core Score Component**
- Formula only uses creative_score, speed, confidence
- **Ignores `final_score`** which is the primary benchmark!
- Image models like DALL-E, Midjourney, Stable Diffusion should be ranked by generation quality

**🔴 ISSUE #5: Speed Normalization Wrong for Image Models**
- `tokens_per_sec` is for LLMs, not image generation
- Image models should use `images_per_minute` or `latency_seconds`
- Current: GPT-4 (text) with 100 tok/s gets higher speed rank than Midjourney

**Proposed Fix**:
```cpp
ranks.image = (final_score * 0.5)              // Generation quality
            + (creative_score * 0.3)           // Aesthetic/creativity
            + (speed_norm * 0.1)               // Speed (if available)
            + (conf_factor * 0.1);             // Reliability
if (modalities.count(Image) == 0) ranks.image = 0;
```

**Eligibility**: ✅ Filtered by `modalities.count(Image)`

**Current Status**: 51 models detected (GPT-4, Claude 3+, Gemini, Grok 2-4, QwenVL, etc.)

---

### 3.5 Video Ranking (Line 291-293)

```cpp
ranks.video = (creative_score * 0.7) + (speed_norm * 0.1) + (confidence / 100 * 0.1);
if (modalities.count(Video) == 0) ranks.video = 0;
```

**Weights**: 70% + 10% + 10% = 90% (❌ Missing 10%)

**🔴 ISSUE #6: Same Issues as Image Tab**
- Missing `final_score` component (0.0 weight!)
- Wrong speed metric for video models
- Only 1 model detected (Sora HD - simulated)

**🔴 ISSUE #7: No Video Model Detection in API**
- API doesn't provide `video_generation` capability field
- Name-based detection only catches "sora", "runway", "gen-2", "gen-3"
- Missing: Pika, Stable Video Diffusion, AnimateDiff, etc.

**Proposed Fix**:
```cpp
ranks.video = (final_score * 0.5)              // Generation quality
            + (creative_score * 0.3)           // Storytelling/coherence
            + (conf_factor * 0.1)              // Reliability
            + (speed_norm * 0.1);              // Render speed
if (modalities.count(Video) == 0) ranks.video = 0;
```

---

### 3.6 Speed Ranking (Line 295-296)

```cpp
ranks.speed = tokens_per_sec;
```

**✅ Correct**: Direct metric, no transformation needed

**⚠️ Note**: Not normalized - used for sorting, not scoring

---

### 3.7 Confidence Ranking (Line 298-299)

```cpp
ranks.confidence = confidence_score;
```

**✅ Correct**: Direct mapping from computed confidence

---

### 3.8 Enterprise Ranking (Line 301-304)

```cpp
ranks.enterprise = (confidence / 100 * 0.4)    // Reliability
                 + (uptime_sla * 0.3)          // Availability guarantee
                 + (org_maturity * 0.3);       // Vendor maturity
```

**Weights**: 40% + 30% + 30% = 100%

**✅ Correct Formula**

**⚠️ Issue**: `uptime_sla` and `org_maturity` are hardcoded estimates (Lines 445-446):
```cpp
if (is_enterprise_ready) { org_maturity = 0.95; uptime_sla = 0.99; }
else { org_maturity = 0.5; uptime_sla = 0.8; }
```

**Not from API** - should fetch from SLA documentation or monitoring data

---

## IV. MODALITY DETECTION LOGIC

### 4.1 Detection Pipeline (Lines 387-427)

```cpp
// Priority 1: API-provided modalities
if (rawItem.contains("modalities") && rawItem["modalities"].is_array()) {
    for (const auto& mod : rawItem["modalities"]) {
        if (mod == "image") modalities.insert(Image);
        if (mod == "video") modalities.insert(Video);
        if (mod == "text") modalities.insert(Text);
    }
}
// Priority 2: Name-based heuristics
else {
    if (/* image keywords */) { modalities.insert(Image); modalities.insert(Text); }
    else if (/* video keywords */) { modalities.insert(Video); modalities.insert(Text); }
    else if (/* multimodal LLMs */) { modalities.insert(Image); modalities.insert(Text); }
    else { modalities.insert(Text); }
}
```

**✅ Correct Priority**: API first, name fallback

**✅ Image Detection** (Line 410-421): Catches GPT-4/5, Claude 3/4, Gemini, Grok 2-4, QwenVL, Llama 3.2 11B/90B, Pixtral, QVQ, "vision", "-vl", "diffusion"

**⚠️ Video Detection** (Line 405-408): Only catches "sora", "runway", "gen-2", "gen-3"
- Missing: pika, animate, stable-video, kling, etc.

**Proposed Enhancement**:
```cpp
// Video models (Lines 405-408)
else if (n.find("sora") != std::string::npos || n.find("runway") != std::string::npos ||
         n.find("gen-2") != std::string::npos || n.find("gen-3") != std::string::npos ||
         n.find("pika") != std::string::npos || n.find("animatediff") != std::string::npos ||
         n.find("stable video") != std::string::npos || n.find("kling") != std::string::npos ||
         n.find("video generation") != std::string::npos) {
    m.modalities.insert(Modality::Video);
    m.modalities.insert(Modality::Text);
}
```

---

## V. SIMULATED DATA INJECTION

### 5.1 EnsureCategoryCoverage() - Lines 868-898

```cpp
if (!hasImage) {
    ModelEntity m("Midjourney v6 (Simulated)", "Midjourney");
    m.AddSignal("Manual", 0.95, 1.0);
    // ... enrich, set creative_score = 0.98
    registry.push_back(m);
}
if (!hasVideo) {
    ModelEntity m("Sora HD (Simulated)", "OpenAI");
    m.AddSignal("Manual", 0.92, 1.0);
    // ... enrich, set creative_score = 0.96
    registry.push_back(m);
}
```

**🔴 CRITICAL ISSUE #8: Hardcoded Mock Data**
- System injects simulated models if API doesn't provide any
- Defeats purpose of "live data" requirement
- Corrupts CSV/JSON exports with fake entries
- Sora HD shows in JSON with `"is_video": true`, `confidence: 90.0`

**User Requirement Violation**: "Remove all simulated, placeholder, or hardcoded values"

**Recommended Action**: **DELETE THIS FUNCTION** or make it debug-only with flag

---

## VI. DATA PARSING & VALIDATION

### 6.1 String/Number Handling - Lines 73-107

```cpp
bool TryGetDouble(const json& j, const std::string& key, double& out) {
    if (!j.contains(key) || j[key].is_null()) return false;
    
    if (j[key].is_number()) {
        out = j[key].get<double>();
        return true;
    } else if (j[key].is_string()) {
        try {
            out = std::stod(j[key].get<std::string>());
            return true;
        } catch (...) { return false; }
    }
    return false;
}
```

**✅ Correct**: Handles both `"0.75"` (string) and `0.75` (number) from API

**Impact**: Fixed 115 skipped models → now processing 154 models

---

### 6.2 Fallback Estimation Logic

**Creative Score** (Lines 468-477):
```cpp
if (!found_in_api) {
    // Estimate from modality and tier
    if (has_image || has_video) creative = final_score * 1.3;
    else if (o == "openai" || o == "anthropic") creative = final_score * 1.15;
    else creative = final_score * 1.0;
}
```

**⚠️ Issue**: Multiplying by 1.3 can exceed 1.0 bound
- Model with final_score=0.85 gets creative=1.105 ❌

**Proposed Fix**:
```cpp
creative = std::min(1.0, final_score * 1.15);  // Cap at 1.0
```

**Coding Score** (Lines 452-458):
```cpp
if (!found_in_api) {
    if (found_humaneval) coding = humaneval;
    else coding = final_score * 1.1;  // ❌ Can exceed 1.0
}
```

**Same Issue**: Needs capping

---

## VII. CRITICAL BUGS SUMMARY

| # | Severity | Issue | Location | Impact |
|---|----------|-------|----------|--------|
| 1 | Medium | Duplicate confidence calculation | Lines 177-217, 219-257 | Wasted computation, inconsistent `confidence_reason` |
| 2 | High | Value formula penalizes high scores | Line 277 | Top models ranked lower than mediocre cheap ones |
| 3 | **CRITICAL** | Context window not normalized | Line 282 | Coding rankings completely broken (values in thousands) |
| 4 | High | Image ranking missing final_score | Line 286 | Image quality not factored into ranking |
| 5 | High | Image ranking uses wrong speed metric | Line 285 | Text LLM speed favored over image generation speed |
| 6 | High | Video ranking missing final_score | Line 292 | Video quality not factored into ranking |
| 7 | Medium | Video detection incomplete | Lines 405-408 | Only 1 video model detected |
| 8 | **CRITICAL** | Simulated data injection | Lines 868-898 | Violates "no hardcoded values" requirement |
| 9 | Medium | Creative score can exceed 1.0 | Line 473 | Invalid score bounds |
| 10 | Medium | Coding score can exceed 1.0 | Line 457 | Invalid score bounds |
| 11 | Low | Enterprise metrics hardcoded | Lines 445-446 | Not from live data |
| 12 | Medium | Image/Video weights don't sum to 100% | Lines 286, 292 | Missing 20%/10% of score |

---

## VIII. MODALITY-SPECIFIC LOGIC CORRECTNESS

### 8.1 Text Models (LLMs)

**Eligibility**: No explicit filter - all models included

**Primary Tabs**: Overall, Coding, Confidence, Enterprise, Value

**Ranking Factors**:
- ✅ final_score (GPQA benchmark)
- ✅ coding_score (HumanEval/MBPP)
- ❌ context_window (not normalized)
- ✅ reasoning_score
- ✅ price_input_1m
- ✅ tokens_per_sec

**Status**: Mostly correct, needs context window fix

---

### 8.2 Image Models

**Eligibility**: `modalities.count(Modality::Image) > 0`

**Primary Tabs**: Image, Overall

**Ranking Factors**:
- ❌ final_score (0% weight - missing!)
- ✅ creative_score (60%)
- ⚠️ tokens_per_sec (wrong metric - should be images/min or latency)
- ✅ confidence (10%)

**Status**: Formula incomplete, wrong speed metric

**Detected Models** (51 total):
- GPT-4, GPT-4.1, GPT-4.5, GPT-5.x (vision-enabled)
- Claude 3 Haiku/Sonnet/Opus, Claude 3.5, Claude 3.7, Claude 4
- Gemini 1.5 Flash/Pro, Gemini 2.0/2.5/3.0 (all variants)
- Grok-2, Grok-3, Grok-4
- QwenVL 32B, Qwen3 VL 30B/32B
- Llama 3.2 11B/90B (vision variants)
- Gemini Diffusion (specialized)

**✅ Detection Working**

---

### 8.3 Video Models

**Eligibility**: `modalities.count(Modality::Video) > 0`

**Primary Tabs**: Video, Overall

**Ranking Factors**:
- ❌ final_score (0% weight - missing!)
- ✅ creative_score (70%)
- ⚠️ tokens_per_sec (wrong metric)
- ✅ confidence (10%)

**Status**: Formula incomplete, detection inadequate

**Detected Models** (1 total):
- Sora HD (Simulated) ❌ Hardcoded

**⚠️ Detection Broken**: API likely doesn't return video models, name detection too narrow

---

## IX. RECOMMENDATIONS & FIXES

### Priority 1: CRITICAL Fixes (Must Fix)

**1. Remove Simulated Data Injection**
```cpp
// DELETE Lines 868-898 (EnsureCategoryCoverage)
// OR wrap in #ifdef DEBUG_MODE
```

**2. Normalize Context Window in Coding Rank**
```cpp
// Line 282 - Replace
double ctx_norm = std::clamp(metrics.context_window / 200000.0, 0.0, 1.0);
ranks.coding = (metrics.coding_score * 0.6) + (metrics.reasoning_score * 0.2) 
             + (ctx_norm * 0.1) + (conf_factor * 0.1);
```

**3. Fix Image Ranking Formula**
```cpp
// Line 286 - Replace
ranks.image = (final_score * 0.5) + (metrics.creative_score * 0.3) 
            + (speed_norm * 0.1) + (conf_factor * 0.1);
```

**4. Fix Video Ranking Formula**
```cpp
// Line 292 - Replace
ranks.video = (final_score * 0.5) + (metrics.creative_score * 0.3) 
            + (conf_factor * 0.1) + (speed_norm * 0.1);
```

---

### Priority 2: HIGH Fixes (Should Fix)

**5. Fix Value Ranking Formula**
```cpp
// Line 277 - Replace
double log_price = std::log10(metrics.price_input_1m + 1.0);
ranks.value = (final_score * final_score) / (log_price + 0.1);
```

**6. Cap Estimated Scores**
```cpp
// Line 473, 457 - Add clamping
metrics.creative_score = std::min(1.0, final_score * multiplier);
metrics.coding_score = std::min(1.0, final_score * 1.1);
```

**7. Enhance Video Detection**
```cpp
// Line 405 - Expand pattern
n.find("pika") != std::string::npos || 
n.find("animatediff") != std::string::npos ||
n.find("stable video") != std::string::npos ||
n.find("kling") != std::string::npos
```

---

### Priority 3: MEDIUM Improvements (Nice to Have)

**8. Consolidate Confidence Calculation**
- Keep RecalculateConfidence(), remove duplicate from ComputeAggregates()
- Update confidence_reason string in RecalculateConfidence()

**9. Fetch Enterprise Metrics from API**
- Replace hardcoded uptime_sla/org_maturity with API fields
- Add fallback to monitoring services (status.openai.com, etc.)

**10. Add Speed Metrics for Image/Video**
- Parse `generation_time_sec` or `images_per_minute` from API
- Use separate normalization for generation models vs LLMs

---

## X. TESTING VERIFICATION

### 10.1 Current Test Results

**✅ Passing**:
- 154 models processed (up from 68 pre-fix)
- 0 type errors (down from 115)
- Confidence variance: 52.5% to 92.5% ✅
- Image models detected: 51 ✅
- Coding models: 155 ✅

**❌ Failing**:
- Video models: 1 (simulated) ❌
- Context window: Raw values in thousands ❌
- Value ranking: Inverted for high performers ❌
- Image/Video: Missing final_score component ❌

### 10.2 Validation Commands

```powershell
# Check confidence distribution
(Get-Content data\leaderboard_all.json | ConvertFrom-Json).models | 
    Select-Object name, @{N='Conf';E={$_.meta.confidence}} | 
    Sort-Object Conf

# Check coding rank values (should be 0-1, currently 100-20000!)
(Get-Content data\leaderboard_all.json | ConvertFrom-Json).models | 
    Select-Object name, @{N='Coding';E={$_.ranks.coding}} | 
    Sort-Object Coding -Descending | Select-Object -First 10

# Verify modality detection
(Get-Content data\leaderboard_all.json | ConvertFrom-Json).models | 
    Where-Object {$_.meta.is_image -eq $true} | 
    Select-Object name, organization
```

---

## XI. ARCHITECTURAL RECOMMENDATIONS

### 11.1 Data Source Diversification

**Current**: Single API (api.zeroeval.com)

**Recommended**: Add redundancy
```cpp
namespace Config {
    const std::vector<APIEndpoint> SOURCES = {
        {"api.zeroeval.com", "/leaderboard/models/full", 1.0},
        {"openai.com/api/models", "/v1/models", 0.5},
        {"huggingface.co/api", "/models", 0.3}
    };
}
```

### 11.2 Caching & Differential Updates

**Current**: Full re-fetch every run

**Recommended**: Cache + timestamp-based updates
```cpp
class CacheManager {
    json LoadCache(const std::string& path);
    bool IsStale(const json& cache, int max_age_hours);
    json MergeDelta(const json& cache, const json& fresh);
};
```

### 11.3 Modular Ranking Strategies

**Current**: Hardcoded formulas in ComputeRankings()

**Recommended**: Strategy pattern
```cpp
class RankingStrategy {
    virtual double ComputeScore(const ModelEntity& m) = 0;
};

class CodingRankingStrategy : public RankingStrategy {
    double ComputeScore(const ModelEntity& m) override {
        return (m.metrics.coding_score * 0.6) + ...;
    }
};

std::map<std::string, std::unique_ptr<RankingStrategy>> strategies;
```

### 11.4 Validation Layer

**Current**: Minimal validation (null checks)

**Recommended**: Schema validation + business rules
```cpp
class ModelValidator {
    bool ValidateScoreBounds(double score) { return score >= 0.0 && score <= 1.0; }
    bool ValidateModality(const ModelEntity& m);
    bool ValidateConsistency(const ModelEntity& m);
};
```

---

## XII. CONCLUSION

The system demonstrates **solid architectural foundation** with proper separation of concerns (networking, parsing, enrichment, ranking, export). However, **12 critical bugs** compromise data integrity:

**Most Severe**:
1. Context window normalization breaks coding rankings (values exceed 1000x expected)
2. Simulated data injection violates live-data requirement
3. Image/Video rankings ignore primary performance metric (final_score)

**Impact**: Current rankings are **mathematically incorrect** for Coding, Image, Video, and Value tabs.

**Effort to Fix**: ~4 hours for Priority 1+2 fixes

**Post-Fix Validation**: Run test suite with assertion checks on rank bounds [0,1]

---

**END OF ANALYSIS**
