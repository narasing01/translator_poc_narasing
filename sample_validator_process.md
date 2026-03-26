{# Sample Jinja2 Document - Validator Process Overview #}
{# This template renders a summary of an LLM-as-Judge validation agent #}

# Validator — Translation Quality Assessment

## Purpose

The validator acts as an **LLM-as-Judge**, evaluating every translation produced
by the translator agent before it is accepted into the final output.

Validator model: **{{ validator_model | default("your-model") }}**
Response language: **{{ response_lang | default("English") }}**

---

## Evaluation Criteria

{% set criteria = [
    {"name": "Jinja2 Preservation", "weight": "Critical", "type": "Pass/Fail"},
    {"name": "Semantic Accuracy", "weight": "High", "type": "0-100"},
    {"name": "Tone Appropriateness", "weight": "Medium", "type": "0-100"},
    {"name": "Markdown Preservation", "weight": "Medium", "type": "0-100"}
] %}

| #  | Criterion              | Weight     | Score Type |
|----|------------------------|------------|------------|
{% for c in criteria %}
| {{ loop.index }} | {{ c.name }} | {{ c.weight }} | {{ c.type }} |
{% endfor %}

---

### 1. Jinja2 Preservation (Pass / Fail)

This is the **most critical** check. Any modification to template syntax results
in an automatic failure.

{% macro syntax_check(label, pattern, example) %}
- **{{ label }}**: `{{ pattern }}` — e.g. `{{ example }}`
{% endmacro %}

Syntax elements verified:

{{ syntax_check("Variables", "{{ var }}", "{{ device.name }}") }}
{{ syntax_check("Control blocks", "{% tag %}", "{% for d in devices %}") }}
{{ syntax_check("Comments", "{# text #}", "{# section heading #}") }}
{{ syntax_check("Filters", "{{ v | filter }}", "{{ items | length }}") }}

> If `jinja_preserved` is **false**, the overall score is forced to **0**.

---

### 2. Semantic Accuracy

Checks whether the translated text conveys the **same meaning** as the original.

{% if sample_issues is defined %}
Known issues from recent runs:
{% for issue in sample_issues %}
  - {{ issue }}
{% endfor %}
{% else %}
No known issues recorded.
{% endif %}

---

### 3. Tone Appropriateness

Evaluates whether the Keigo level matches the requested preset.

Requested preset: **{{ tone_preset | default("enterprise") }}**

---

### 4. Markdown Preservation

Ensures structural elements survive translation:

- Headers (`#`, `##`, `###`)
- Tables (column count, alignment)
- Code blocks (``` fences)
- Ordered and unordered lists

---

## Output Schema

The validator returns a **JSON object** with the following shape:

```json
{
  "valid": {{ is_valid | default(true) | lower }},
  "jinja_preserved": {{ jinja_ok | default(true) | lower }},
  "scores": {
    "semantic_accuracy": {{ scores.semantic | default(95) }},
    "tone_appropriateness": {{ scores.tone | default(90) }},
    "markdown_preservation": {{ scores.markdown | default(100) }},
    "overall": {{ scores.overall | default(95) }}
  },
  "issues": {{ issues | default([]) | tojson }},
  "feedback": "{{ feedback | default('Translation quality is excellent.') }}"
}
```

---

## Scoring Logic

{% set pass_threshold = pass_threshold | default(80) %}

```
IF jinja_preserved == false:
    overall = 0          {# automatic fail #}
ELSE:
    overall = avg(semantic_accuracy, tone_appropriateness, markdown_preservation)

Translation passes when overall >= {{ pass_threshold }}.
```

---

## Retry Flow

{% if retry_enabled | default(true) %}
When validation **fails**, the feedback is sent back to the translator:

1. Validator returns `issues` and `feedback`.
2. Pipeline injects feedback into a retry prompt.
3. Translator regenerates the translation.
4. Validator re-evaluates (up to {{ max_retries | default(3) }} retries).
{% else %}
Retry is currently **disabled**. Failed translations are flagged for manual review.
{% endif %}

---

## Summary

| Setting          | Value                                          |
|------------------|------------------------------------------------|
| Validator Model  | {{ validator_model | default("your-model") }}          |
| Pass Threshold   | {{ pass_threshold | default(80) }}              |
| Max Retries      | {{ max_retries | default(3) }}                  |
| Response Lang    | {{ response_lang | default("English") }}        |

{# End of validator process document #}
