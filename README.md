<div align="center">

<img src="https://static.djangoproject.com/img/logos/django-logo-positive.svg" width="260" alt="Django Logo"/>

<br/>

# Django Documentation

**High-level reference documentation for Django**
Organized, beginner-friendly, and built for intermediate developers.

<br/>

![Django](https://img.shields.io/badge/Django-5.x-092E20?style=for-the-badge&logo=django&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Markdown](https://img.shields.io/badge/Docs-Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-22C55E?style=for-the-badge)

</div>

---

## 🗂️ What is this?

This repository contains structured Markdown documentation covering Django concepts — from settings and configuration to deployment workflows. Each file is self-contained and written to be scanned quickly, not read like a textbook.

---

## 📚 Contents

| # | Topic | Description | File |
|---|-------|-------------|------|
| 01 | **Static Files** | `STATIC_URL`, `STATICFILES_DIRS`, `STATIC_ROOT` | [Static\_Files\_settings.md](./Static_Files_settings.md) |

> 💡 More topics will be added as the repository grows. Each file includes its own Table of Contents for in-file navigation.

---


## 🚀 How to Use

1. Click any link in the **Contents** table to jump directly to that topic
2. Each doc has its own **Table of Contents** at the top for section-level navigation
3. Mermaid diagrams render natively on GitHub — no plugins needed

---

## 🏗️ Django Architecture at a Glance

```
Browser Request
      │
      ▼
  urls.py  ──────────────────► 404 Not Found
      │
      ▼
  views.py
      │
      ├──► models.py ──► Database
      │
      └──► templates/ ──► HTML Response
                │
                └──► static/  ──► CSS · JS · Images
```

---

<div align="center">

<img src="https://static.djangoproject.com/img/logos/django-logo-negative.svg" width="80" alt="Django"/>

**Built by [Ratan1103](https://github.com/Ratan1103)**
*Personal reference · Open learning resource*

</div>