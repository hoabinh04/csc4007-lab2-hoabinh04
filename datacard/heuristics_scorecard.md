# Data Card Heuristics Scorecard (Module 4 - AUDIT)

**Lab 2 - IMDB Sentiment Classification**  
**Student**: hoabinh04  
**Date**: April 2026  
**Version**: 1.0

---

## Self-Scoring Rubric (1–5 scale)

### Criterion 1: Completeness
**Definition**: All essential sections present (overview, audit, labels, limitations, recommendations)

**Score: 5/5** ⭐⭐⭐⭐⭐

**Rationale**:
- ✅ Dataset overview: Name, description, sources, use cases
- ✅ Data quality audit: 3 major issues with evidence (duplicates, HTML, length variation)
- ✅ Preprocessing transformations: Before/after metrics showing impact
- ✅ Validation results: GE + Cleanlab integration documented
- ✅ Model performance: 3 baselines compared, error analysis with 4 categories
- ✅ Limitations & risks: Sarcasm blindness, shortcut learning, mitigation strategies
- ✅ Recommendations: Clear safe/conditional/unsafe use cases

**Evidence**: data_card.md sections 3.1–3.13 cover all themes. No critical gaps.

---

### Criterion 2: Accuracy
**Definition**: All numbers and claims are factually correct; backed by outputs/logs

**Score: 5/5** ⭐⭐⭐⭐⭐

**Rationale**:
- ✅ "50,000 samples" — verified in `outputs/logs/audit_before.md`: n_rows=50000
- ✅ "824 exact duplicates (1.648%)" — verified: exact_dup_count=824 / 50000 = 0.01648
- ✅ "50% balanced (25k each)" — verified: label_counts={negative:25000, positive:25000}
- ✅ "Median 970 chars" — verified: len_chars_median=970
- ✅ "Min 32, Max 13,704 chars" — verified: len_chars_min=32, len_chars_max=13,704
- ✅ "TF-IDF + LogReg F1=0.9064" — verified: `outputs/metrics/metrics_summary.json`
- ✅ "468 errors on test set" — verified: `outputs/error_analysis/error_analysis_summary.md`
- ✅ "9.36% error rate" — calculated: 468 / 5000 = 0.0936

**Evidence**: All numbers cross-referenced with outputs/ files. 0 discrepancies found.

---

### Criterion 3: Clarity
**Definition**: Written in clear language; non-technical summaries; jargon explained

**Score: 4/5** ⭐⭐⭐⭐

**Rationale**:
- ✅ Executive summary clear: 3 fundamental questions + key issues highlighted
- ✅ Issues explained in 3-part structure: Evidence → Why dangerous → Mitigation
- ✅ Examples provided for each issue (e.g., "`it is great&amp;fun`" for HTML)
- ✅ Tables & bullet points for readability
- ✅ Technical terms explained ("shortcut learning", "TF-IDF", "macro-F1")
- ✅ Recommendations use plain English ("sarcasm detector", "truncate at 500 words")
- ⚠️ Error analysis could benefit from more examples / visualization
- ⚠️ Some sections (heuristics formula) are somewhat academic

**Missing**: 1-2 visual charts (confusion matrix, length distribution) would improve clarity. Text is strong; visualization would add 1 more point.

---

### Criterion 4: Timeliness
**Definition**: Created recently; reflects current dataset state and methodology

**Score: 5/5** ⭐⭐⭐⭐⭐

**Rationale**:
- ✅ Dataset version: Latest IMDB snapshot (open source)
- ✅ Preprocessing code: Matches `src/preprocess.py` v1.0 exactly
- ✅ Model results: Generated April 2026 from `run_lab2.py` in current session
- ✅ Audit metrics: From `outputs/` generated same day as data card
- ✅ Best practices: References modern approaches (BERT, transformers, Cleanlab)
- ✅ Recommendations: Reflect 2024+ NLP standards (no outdated techniques)

**Evidence**: All artifacts timestamped April 2026. No stale information.

---

### Criterion 5: Actionability
**Definition**: Readers can take concrete steps based on recommendations

**Score: 5/5** ⭐⭐⭐⭐⭐

**Rationale**:

**For Data Engineers**:
- ✅ "Remove 824 duplicates before train/test split" — Clear code snippet provided
- ✅ "Apply `html.unescape()` before cleaning" — Specific library & function name
- ✅ "Truncate at 500 words" — Concrete threshold with rationale

**For ML Engineers**:
- ✅ "Use TF-IDF + LogReg for baseline" — Exact configuration provided
- ✅ "Watch for sarcasm failures" — Diagnostic to implement
- ✅ "Try BERT for next iteration" — Research direction with clear motivation

**For Instructors**:
- ✅ "Grading checklist: identify 3 issues, propose mitigations, compare pipelines" — Clear rubric
- ✅ Evidence all present in data_card.md

**For Researchers**:
- ✅ "Length-sentiment correlation analysis" — Specific investigation suggested
- ✅ "Error categories: sarcasm (32%), mixed sentiment (26%), ..." — Data-driven roadmap

**Evidence**: Each section has 3+ concrete actions. No vague recommendations.

---

## Summary Table

| Criterion | Score | Status | Notes |
|-----------|-------|--------|-------|
| **Completeness** | 5/5 | ✅ Excellent | All sections documented |
| **Accuracy** | 5/5 | ✅ Excellent | 8/8 claims verified |
| **Clarity** | 4/5 | ✅ Very Good | Could add 1-2 visualizations |
| **Timeliness** | 5/5 | ✅ Excellent | Generated April 2026 |
| **Actionability** | 5/5 | ✅ Excellent | Concrete steps for all readers |
| **AVERAGE** | **4.8/5** | ⭐⭐⭐⭐⭐ | **Production-ready Data Card** |

---

## Strengths
1. **Comprehensive**: Covers data collection, quality, preprocessing, validation, models, limitations
2. **Evidence-based**: Every claim backed by outputs/ files
3. **Accessible**: Clear language with examples; non-technical readers can understand
4. **Actionable**: Readers know what to do next (remove duplicates, try BERT, check for sarcasm)
5. **Current**: Uses latest datasets, tools, best practices

## Areas for Improvement
1. **Visualization**: Add 2-3 figures (confusion matrix, length distribution, error breakdown pie chart)
2. **Extended section**: Could expand "Related Work" to cite similar sentiment classification studies
3. **Deployment checklist**: Step-by-step production deployment guide (but lower priority)

## Recommended Actions Before Publication
- [ ] Add confusion matrix visualization (from `outputs/figures/`)
- [ ] Add text length distribution histogram
- [ ] Add error category breakdown pie chart
- [ ] Cross-check references to files paths (all current ✅)
- [ ] Solicit feedback from 1-2 peers

---

**Data Card Quality: APPROVED for publication** ✅

Status: Ready to share with classmates, instructors, or publish as documentation.
