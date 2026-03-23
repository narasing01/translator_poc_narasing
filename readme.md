# Language Translation Service 

## Table of Contents

1. [Project Overview](#1-project-overview)

---

## 1. Project Overview

### Purpose

This is an **AI-powered translation service** that translates **Jinja2 markdown templates** (documentation automation templates) from **English to Japanese**, while **perfectly preserving all Jinja2 template syntax** (`{{ variable }}`, `{% for %}`, `{% if %}`, etc.).

### Problem It Solves

Documentation Automation systems use Jinja2-templated markdown files to generate network device documentation. These templates contain a mix of:

- Human-readable prose → **must be translated**
- Jinja2 variables and control structures → **must NOT be touched**

A naive LLM translation would corrupt the template syntax, breaking the documentation generator. This service solves that with a specialized pipeline that protects template syntax while translating the surrounding text.
