{# Sample Jinja2 Document - Translator Process Overview #}
{# This template renders a summary of a translator agent #}

# {{ project_name | default("My Translation Service") }} — Translator Process

## Overview

The translator agent is responsible for converting Jinja2 markdown templates from
**{{ source_lang | default("English") }}** to **{{ target_lang | default("Japanese") }}**
while preserving all template syntax exactly as written.

---

## How It Works

{% set phases = ["Prompt Construction", "Context Injection", "LLM Invocation", "Output Extraction"] %}

The translation process follows these phases:

{% for phase in phases %}
{{ loop.index }}. **{{ phase }}**
{% endfor %}

### Phase Details

{% macro phase_card(title, description) %}
| Attribute | Value |
|-----------|-------|
| Phase     | {{ title }} |
| Detail    | {{ description }} |
{% endmacro %}

{{ phase_card("Prompt Construction",
   "Builds a system prompt using tone preset (" ~ tone_preset | default("enterprise") ~ ") and language pair.") }}

{{ phase_card("Context Injection",
   "Attaches glossary and chapter previews so the LLM translates consistently.") }}

{{ phase_card("LLM Invocation",
   "Sends the prompt to " ~ model_name | default("your-model") ~ " via the LLM framework.") }}

{{ phase_card("Output Extraction",
   "Returns ONLY the translated content — no explanations or code fences.") }}

---

## Jinja2 Preservation Rules

The translator MUST keep all template syntax **character-for-character identical**.

| Syntax Type       | Example                                  | Translate? |
|-------------------|------------------------------------------|------------|
| Variables         | `{{ device.name }}`                      | **No**     |
| Control blocks    | `{% for d in devices %}`                 | **No**     |
| Filters           | `{{ items | length }}`                   | **No**     |
| Prose text        | Hardware devices are listed below.       | **Yes**    |
| Table headers     | `| Name | Model |`                       | **Yes**    |

---

## Tone Presets

{% if tone_presets is defined %}
{% for preset in tone_presets %}
- **{{ preset.name }}** — {{ preset.description }}
{% endfor %}
{% else %}
Available tone presets: `enterprise`, `government`, `service_provider`.
{% endif %}

---

## Parallel Translation

Files are translated concurrently using `asyncio.gather`. Each file is independent:

```
Total files : {{ file_count | default(5) }}
Concurrency : {{ concurrency | default(3) }}
Timeout/file: {{ timeout_seconds | default(120) }}s
```

{% if retry_enabled | default(true) %}
> **Retry enabled** — If the validator rejects a translation, the translator
> receives feedback and retries up to {{ max_retries | default(3) }} times.
{% endif %}

---

## Configuration Summary

| Setting        | Value                                         |
|----------------|-----------------------------------------------|
| Source Language | {{ source_lang | default("English") }}        |
| Target Language | {{ target_lang | default("Japanese") }}       |
| Tone Preset    | {{ tone_preset | default("enterprise") }}     |
| Model          | {{ model_name | default("your-model") }}       |

{# End of translator process document #}
