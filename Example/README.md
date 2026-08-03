# An Example Project

This README describes how I use Hermes Agent to: 

*   Produce a Spec file using the `/plan` SKILL, and

*   Create the SDDs (Software Development Documents) using the `/dcv` SKILL.

## Spec Files

I can easily create Spec files by calling on the built-in Hermes Agent SKILL called `/plan`.

*   I make a directory on my Desktop:

```bash
mkdir ~/Desktop/CRM
```

*   I start Hermes Agent:

```bash
hermes
```

*   I use the following prompt to help me create an Spec file for the CRM app:

`/plan I want to create a CRM app. The CRM is a full web app built with Python + Flask/FastAPI. The features include contact management with CRUD operations, contact notes with CRUD operations, file storage with CRUD operations, and email storage with CRUD operations. Delete operations are soft, which stops the data from being viewed while still being retained for auditing purposes. The CRM also has an API endpoint so that other apps, like an email utility, can use the data to personalise the content when sending mass email messages. The GUI is displayed as a browser-based web dashboard and SQLite is used for data storage. Create a production-ready plan that you save as the "~/Desktop/CRM/crm-spec.md" Markdown file.`

## Hermes Agent Response

```plaintext
╭─ ⚕ Hermes ───────────────────────────────────────────────────────────────────╮
    Now I will write the comprehensive plan.
╰──────────────────────────────────────────────────────────────────────────────╯
  ┊ ✍️ preparing write_file…
  ┊ ✍️  write     /home/brian/Desktop/CRM/crm-spec.md  0.2s
  ┊ review diff
a//home/brian/Desktop/CRM/crm-spec.md → b//home/brian/Desktop/CRM/crm-spec.md
@@ -0,0 +1,2127 @@
+# CRM Web Application — Production Implementation Plan
+
+> **For Hermes:** Use subagent-driven-development to implement this plan task-by-task.
+
+**Goal:** Build a production-ready CRM web application with contact management, notes,
+file storage, email storage, soft-delete auditing, an external API layer, and a
+browser-based dashboard.
+
+**Architecture:** FastAPI backend with SQLAlchemy ORM + SQLite, Jinja2 server-rendered
+templates with Alpine.js for interactive UI, Tailwind CSS for styling, and a layered API
+surface for external app integration.
+
+**Tech Stack:** Python 3.12+, FastAPI, SQLAlchemy 2.0, Alembic, Jinja2, Alpine.js,
+Tailwind CSS, SQLite, python-multipart, aiofiles
+
+---
+
+## Project Layout
+
+CRM/
+├── app/
+│   ├── __init__.py              # FastAPI app factory
+│   ├── config.py                # Settings (SQLite path, upload dir, secret key)
+│   ├── database.py              # SQLAlchemy engine + session setup
+│   ├── models/
+│   │   ├── __init__.py          # Re-export all models
+│   │   ├── base.py              # Declarative base + common mixin (SoftDeleteMixin,
+|   │   │                        # TimestampMixin)
+│   │   ├── contact.py           # Contact model
+│   │   ├── contact_note.py      # ContactNote model
+│   │   ├── file.py              # File model
+│   │   └── email.py             # Email model
+│   ├── schemas/
+│   │   ├── __init__.py
+│   │   ├── contact.py           # Pydantic request/response schemas
+│   │   ├── note.py
+│   │   ├── file.py
+│   │   └── email.py
+│   ├── api/
+│   │   ├── __init__.py
+│   │   ├── contacts.py          # /api/contacts/* endpoints
+│   │   ├── notes.py             # /api/contacts/{id}/notes/*
+│   │   ├── files.py             # /api/files/*
+│   │   └── emails.py            # /api/emails/*
+│   ├── web/
+│   │   ├── __init__.py
+│   │   ├── contacts.py          # Dashboard routes for contacts
+│   │   ├── notes.py             # Dashboard routes for notes
+│   │   ├── files.py             # Dashboard routes for file uploads
+│   │   ├── emails.py            # Dashboard routes for emails
+│   │   └── dashboard.py         # Main dashboard / home
+│   ├── services/
+│   │   ├── __init__.py
+│   │   ├── contact_service.py   # Business logic for contacts
+│   │   ├── note_service.py
+│   │   ├── file_service.py      # File storage logic (disk + DB)
+│   │   └── email_service.py
+│   └── templates/
+│       ├── base.html            # Layout (Tailwind + Alpine.js)
+│       ├── index.html           # Dashboard landing
+│       ├── contacts/
+│       │   ├── list.html
+│       │   ├── detail.html
+│       │   └── form.html
+│       ├── notes/
+│       │   └── _list.html       # Partial for HTMX/Alpine inline
+│       ├── files/
+│       │   └── _list.html
+│       └── emails/
+│           └── _list.html
+├── migrations/                  # Alembic migrations
+│   ├── env.py
+│   ├── script.py.mako
+│   └── versions/
+├── uploads/                     # Uploaded files (gitignored)
+├── crm.db                       # SQLite database (gitignored)
+├── requirements.txt
+├── alembic.ini
+├── run.py                       # Entry point: uvicorn app factory
+└── .env                         # SECRET_KEY, DATABASE_PATH, UPLOAD_DIR
+
+---
… omitted 2049 diff line(s) across 1 additional file(s)/section(s)

╭─ ⚕ Hermes ───────────────────────────────────────────────────────────────────╮
    Plan saved to /media/brian/Apps/01_lena_v0.5.0/Publish/CRM/crm-spec.md
    (71 KB, 32 tasks across 7 phases).
    
    Summary of what is planned:
    
    - FastAPI backend with SQLAlchemy + SQLite
    - 4 models (Contact, ContactNote, File, Email) — all with soft-delete
    via deleted_at
    - Service layer separating business logic from routes
    - REST API under /api/* for external app integration (email
    utility), protected by API key auth
    - Web dashboard using Jinja2 templates + Tailwind CSS + Alpine.js for
    interactivity
    - Pydantic schemas for request validation and response serialization
    - Test suite with pytest + TestClient covering CRUD and soft-delete
    - Production hardening: CORS, request logging, config validation,
    upload directory setup
    
    32 bite-sized tasks, each 2–5 minutes. Ready to execute using
    subagent-driven-development when you give the go-ahead.
╰──────────────────────────────────────────────────────────────────────────────╯
 ⚕ deepseek-v4-flash │ 71.3K/1M │ [█░░░░░░░░░] 7% │ 1h 38m │ ⏲ 44m 1s │ ✓ 0s...
────────────────────────────────────────────────────────────────────────────────
❯ 
```

## Software Development Documents

Once the Spec file is created, I can switch to making the SDDs:

```bash
/dcv ~/Desktop/CRM/crm-spec.md
```
