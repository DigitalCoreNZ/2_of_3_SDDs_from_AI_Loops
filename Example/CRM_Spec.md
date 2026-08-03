# CRM Web Application — Production Implementation Plan

> **For Hermes:** Use subagent-driven-development to implement this plan task-by-task.

**Goal:** Build a production-ready CRM web application with contact management, notes, file storage, email storage, soft-delete auditing, an external API layer, and a browser-based dashboard.

**Architecture:** FastAPI backend with SQLAlchemy ORM + SQLite, Jinja2 server-rendered templates with Alpine.js for interactive UI, Tailwind CSS for styling, and a layered API surface for external app integration.

**Tech Stack:** Python 3.12+, FastAPI, SQLAlchemy 2.0, Alembic, Jinja2, Alpine.js, Tailwind CSS, SQLite, python-multipart, aiofiles

---

## Project Layout

```
CRM/
├── app/
│   ├── __init__.py              # FastAPI app factory
│   ├── config.py                # Settings (SQLite path, upload dir, secret key)
│   ├── database.py              # SQLAlchemy engine + session setup
│   ├── models/
│   │   ├── __init__.py          # Re-export all models
│   │   ├── base.py              # Declarative base + common mixin (SoftDeleteMixin, TimestampMixin)
│   │   ├── contact.py           # Contact model
│   │   ├── contact_note.py      # ContactNote model
│   │   ├── file.py              # File model
│   │   └── email.py             # Email model
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── contact.py           # Pydantic request/response schemas
│   │   ├── note.py
│   │   ├── file.py
│   │   └── email.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── contacts.py          # /api/contacts/* endpoints
│   │   ├── notes.py             # /api/contacts/{id}/notes/*
│   │   ├── files.py             # /api/files/*
│   │   └── emails.py            # /api/emails/*
│   ├── web/
│   │   ├── __init__.py
│   │   ├── contacts.py          # Dashboard routes for contacts
│   │   ├── notes.py             # Dashboard routes for notes
│   │   ├── files.py             # Dashboard routes for file uploads
│   │   ├── emails.py            # Dashboard routes for emails
│   │   └── dashboard.py         # Main dashboard / home
│   ├── services/
│   │   ├── __init__.py
│   │   ├── contact_service.py   # Business logic for contacts
│   │   ├── note_service.py
│   │   ├── file_service.py      # File storage logic (disk + DB)
│   │   └── email_service.py
│   └── templates/
│       ├── base.html            # Layout (Tailwind + Alpine.js)
│       ├── index.html           # Dashboard landing
│       ├── contacts/
│       │   ├── list.html
│       │   ├── detail.html
│       │   └── form.html
│       ├── notes/
│       │   └── _list.html       # Partial for HTMX/Alpine inline
│       ├── files/
│       │   └── _list.html
│       └── emails/
│           └── _list.html
├── migrations/                  # Alembic migrations
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
├── uploads/                     # Uploaded files (gitignored)
├── crm.db                       # SQLite database (gitignored)
├── requirements.txt
├── alembic.ini
├── run.py                       # Entry point: uvicorn app factory
└── .env                         # SECRET_KEY, DATABASE_PATH, UPLOAD_DIR
```

---

## Phase 1 — Project Scaffolding & Database

### Task 1: Create project skeleton and dependencies

**Objective:** Set up directory structure, `requirements.txt`, `.env`, and the app factory.

**Files:**
- Create: `CRM/requirements.txt`
- Create: `CRM/.env`
- Create: `CRM/run.py`
- Create: `CRM/app/__init__.py`
- Create: `CRM/app/config.py`

**Step 1: Write `requirements.txt`**

```
fastapi==0.115.6
uvicorn[standard]==0.34.1
sqlalchemy==2.0.36
alembic==1.14.1
python-multipart==0.0.18
aiofiles==24.1.0
python-dotenv==1.0.1
pydantic==2.10.4
jinja2==3.1.4
itsdangerous==2.2.0
python-jose==3.3.0
passlib[bcrypt]==1.7.4
python-magic==0.4.27
```

**Step 2: Write `.env`**

```
SECRET_KEY=change-this-to-a-long-random-string
DATABASE_PATH=CRM/crm.db
UPLOAD_DIR=CRM/uploads
```

**Step 3: Write `run.py`**

```python
import uvicorn
from app import create_app

app = create_app()

if __name__ == "__main__":
    uvicorn.run("run:app", host="0.0.0.0", port=8080, reload=True)
```

**Step 4: Write `app/config.py`**

```python
from pydantic_settings import BaseSettings
from pathlib import Path

class Settings(BaseSettings):
    secret_key: str = "change-me"
    database_path: str = "CRM/crm.db"
    upload_dir: str = "CRM/uploads"
    debug: bool = False

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}

settings = Settings()
```

**Step 5: Write `app/__init__.py`**

```python
from fastapi import FastAPI
from fastapi.templating import Jinja2Templates
from pathlib import Path

def create_app():
    app = FastAPI(title="CRM", version="1.0.0")

    # Resolve template path
    templates_dir = Path(__file__).resolve().parent / "templates"
    templates = Jinja2Templates(directory=str(templates_dir))
    app.state.templates = templates

    # Mount static files (for uploaded files)
    from fastapi.static import StaticFiles
    from app.config import settings
    upload_path = Path(settings.upload_dir).resolve()
    upload_path.mkdir(parents=True, exist_ok=True)
    app.mount("/uploads", StaticFiles(directory=str(upload_path)), name="uploads")

    # Register routers later after imports
    from app.web.dashboard import router as dash_router
    from app.web.contacts import router as web_contacts_router
    from app.web.notes import router as web_notes_router
    from app.web.files import router as web_files_router
    from app.web.emails import router as web_emails_router
    from app.api.contacts import router as api_contacts_router
    from app.api.notes import router as api_notes_router
    from app.api.files import router as api_files_router
    from app.api.emails import router as api_emails_router

    app.include_router(dash_router)
    app.include_router(web_contacts_router)
    app.include_router(web_notes_router)
    app.include_router(web_files_router)
    app.include_router(web_emails_router)
    app.include_router(api_contacts_router, prefix="/api")
    app.include_router(api_notes_router, prefix="/api")
    app.include_router(api_files_router, prefix="/api")
    app.include_router(api_emails_router, prefix="/api")

    return app
```

**Step 6: Install dependencies**

Run: `pip install -r CRM/requirements.txt`
Expected: all packages installed successfully.

---

### Task 2: Database setup — engine, session, Alembic init

**Objective:** Configure SQLAlchemy engine and Alembic for migrations.

**Files:**
- Create: `CRM/app/database.py`
- Modify: `CRM/alembic.ini` (create after alembic init)

**Step 1: Write `app/database.py`**

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, scoped_session
from app.config import settings

engine = create_engine(
    f"sqlite:///{settings.database_path}",
    connect_args={"check_same_thread": False},
    pool_pre_ping=True,
)

SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

**Step 2: Initialize Alembic**

Run: `cd CRM && alembic init migrations`
Expected: creates `migrations/` directory and `alembic.ini`

**Step 3: Wire Alembic to our engine**

Modify `migrations/env.py` to import `from app.database import engine` and set `target_metadata = Base.metadata`.

---

### Task 3: Base model — common mixins

**Objective:** Create `Base` and mixins for timestamping + soft delete.

**Files:**
- Create: `CRM/app/models/__init__.py`
- Create: `CRM/app/models/base.py`

**Step 1: Write `app/models/base.py`**

```python
from sqlalchemy import Column, Integer, DateTime, Boolean
from sqlalchemy.orm import declarative_base, declared_attr
from datetime import datetime

Base = declarative_base()

class TimestampMixin:
    @declared_attr
    def created_at(cls):
        return Column(DateTime, default=datetime.utcnow, nullable=False)

    @declared_attr
    def updated_at(cls):
        return Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow, nullable=False)

class SoftDeleteMixin:
    @declared_attr
    def deleted_at(cls):
        return Column(DateTime, nullable=True, default=None, index=True)

    def soft_delete(self):
        self.deleted_at = datetime.utcnow()

    def restore(self):
        self.deleted_at = None
```

**Step 2: Write `app/models/__init__.py`**

```python
from .base import Base, TimestampMixin, SoftDeleteMixin
from .contact import Contact
from .contact_note import ContactNote
from .file import File
from .email import Email

__all__ = ["Base", "TimestampMixin", "SoftDeleteMixin", "Contact", "ContactNote", "File", "Email"]
```

---

### Task 4: Contact model

**Objective:** Create the Contact ORM model with all fields and soft-delete support.

**Files:**
- Create: `CRM/app/models/contact.py`

**Step 1: Write `app/models/contact.py`**

```python
from sqlalchemy import Column, Integer, String, Text, DateTime
from sqlalchemy.orm import relationship
from .base import Base, TimestampMixin, SoftDeleteMixin

class Contact(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "contacts"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    email = Column(String(200), nullable=True, index=True)
    phone = Column(String(50), nullable=True)
    company = Column(String(200), nullable=True)
    position = Column(String(200), nullable=True)
    notes = Column(Text, nullable=True)  # short bio / description

    # Relationships
    contact_notes = relationship("ContactNote", back_populates="contact", lazy="dynamic")
    files = relationship("File", back_populates="contact", lazy="dynamic")
    emails = relationship("Email", back_populates="contact", lazy="dynamic")
```

---

### Task 5: ContactNote model

**Objective:** Create the ContactNote ORM model with FK to Contact.

**Files:**
- Create: `CRM/app/models/contact_note.py`

**Step 1: Write `app/models/contact_note.py`**

```python
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime
from sqlalchemy.orm import relationship
from .base import Base, TimestampMixin, SoftDeleteMixin

class ContactNote(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "contact_notes"

    id = Column(Integer, primary_key=True, index=True)
    contact_id = Column(Integer, ForeignKey("contacts.id", ondelete="CASCADE"), nullable=False, index=True)
    title = Column(String(200), nullable=True)
    content = Column(Text, nullable=False)

    contact = relationship("Contact", back_populates="contact_notes")
```

---

### Task 6: File model

**Objective:** Create the File ORM model for tracking uploaded files.

**Files:**
- Create: `CRM/app/models/file.py`

**Step 1: Write `app/models/file.py`**

```python
from sqlalchemy import Column, Integer, String, ForeignKey, DateTime
from sqlalchemy.orm import relationship
from .base import Base, TimestampMixin, SoftDeleteMixin

class File(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "files"

    id = Column(Integer, primary_key=True, index=True)
    contact_id = Column(Integer, ForeignKey("contacts.id", ondelete="SET NULL"), nullable=True, index=True)
    original_name = Column(String(255), nullable=False)
    stored_name = Column(String(255), nullable=False, unique=True)  # UUID filename on disk
    mime_type = Column(String(100), nullable=True)
    size = Column(Integer, nullable=False)  # bytes
    path = Column(String(500), nullable=False)  # relative path inside upload_dir

    contact = relationship("Contact", back_populates="files")
```

---

### Task 7: Email model

**Objective:** Create the Email ORM model.

**Files:**
- Create: `CRM/app/models/email.py`

**Step 1: Write `app/models/email.py`**

```python
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime
from sqlalchemy.orm import relationship
from .base import Base, TimestampMixin, SoftDeleteMixin

class Email(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "emails"

    id = Column(Integer, primary_key=True, index=True)
    contact_id = Column(Integer, ForeignKey("contacts.id", ondelete="SET NULL"), nullable=True, index=True)
    subject = Column(String(500), nullable=False)
    body = Column(Text, nullable=False)
    sender = Column(String(200), nullable=True)
    recipient = Column(String(200), nullable=True)
    sent_at = Column(DateTime, nullable=True)
    received_at = Column(DateTime, nullable=True)

    contact = relationship("Contact", back_populates="emails")
```

---

### Task 8: Auto-generate initial Alembic migration

**Objective:** Generate the first migration that creates all four tables.

**Files:**
- Create: `CRM/migrations/versions/001_initial.py`

**Step 1: Generate migration**

Run: `cd CRM && alembic revision --autogenerate -m "initial"`

**Step 2: Run migration**

Run: `cd CRM && alembic upgrade head`
Expected: tables `contacts`, `contact_notes`, `files`, `emails` created in `crm.db`

---

## Phase 2 — Pydantic Schemas (API Layer)

### Task 9: Contact schemas

**Objective:** Define request/response Pydantic models for contacts.

**Files:**
- Create: `CRM/app/schemas/__init__.py`
- Create: `CRM/app/schemas/contact.py`

**Step 1: Write `app/schemas/__init__.py`**

```python
# Re-export all schemas
```

**Step 2: Write `app/schemas/contact.py`**

```python
from pydantic import BaseModel, EmailStr, field_validator
from datetime import datetime
from typing import Optional, List

class ContactCreate(BaseModel):
    name: str
    email: Optional[str] = None
    phone: Optional[str] = None
    company: Optional[str] = None
    position: Optional[str] = None
    notes: Optional[str] = None

class ContactUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[str] = None
    phone: Optional[str] = None
    company: Optional[str] = None
    position: Optional[str] = None
    notes: Optional[str] = None

class ContactResponse(BaseModel):
    id: int
    name: str
    email: Optional[str] = None
    phone: Optional[str] = None
    company: Optional[str] = None
    position: Optional[str] = None
    notes: Optional[str] = None
    created_at: datetime
    updated_at: datetime
    deleted_at: Optional[datetime] = None

    model_config = {"from_attributes": True}
```

---

### Task 10: Note schemas

**Objective:** Define Pydantic models for contact notes.

**Files:**
- Create: `CRM/app/schemas/note.py`

**Step 1: Write `app/schemas/note.py`**

```python
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class NoteCreate(BaseModel):
    title: Optional[str] = None
    content: str

class NoteUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None

class NoteResponse(BaseModel):
    id: int
    contact_id: int
    title: Optional[str] = None
    content: str
    created_at: datetime
    updated_at: datetime
    deleted_at: Optional[datetime] = None

    model_config = {"from_attributes": True}
```

---

### Task 11: File schemas

**Objective:** Define Pydantic models for file records.

**Files:**
- Create: `CRM/app/schemas/file.py`

**Step 1: Write `app/schemas/file.py`**

```python
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class FileResponse(BaseModel):
    id: int
    contact_id: Optional[int] = None
    original_name: str
    stored_name: str
    mime_type: Optional[str] = None
    size: int
    created_at: datetime
    deleted_at: Optional[datetime] = None

    model_config = {"from_attributes": True}
```

---

### Task 12: Email schemas

**Objective:** Define Pydantic models for emails.

**Files:**
- Create: `CRM/app/schemas/email.py`

**Step 1: Write `app/schemas/email.py`**

```python
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class EmailCreate(BaseModel):
    contact_id: Optional[int] = None
    subject: str
    body: str
    sender: Optional[str] = None
    recipient: Optional[str] = None
    sent_at: Optional[datetime] = None
    received_at: Optional[datetime] = None

class EmailUpdate(BaseModel):
    subject: Optional[str] = None
    body: Optional[str] = None
    sender: Optional[str] = None
    recipient: Optional[str] = None
    sent_at: Optional[datetime] = None
    received_at: Optional[datetime] = None

class EmailResponse(BaseModel):
    id: int
    contact_id: Optional[int] = None
    subject: str
    body: str
    sender: Optional[str] = None
    recipient: Optional[str] = None
    sent_at: Optional[datetime] = None
    received_at: Optional[datetime] = None
    created_at: datetime
    updated_at: datetime
    deleted_at: Optional[datetime] = None

    model_config = {"from_attributes": True}
```

---

## Phase 3 — Service Layer (Business Logic)

### Task 13: Contact service

**Objective:** Implement CRUD + soft-delete logic for contacts.

**Files:**
- Create: `CRM/app/services/__init__.py`
- Create: `CRM/app/services/contact_service.py`

**Step 1: Write `app/services/contact_service.py`**

```python
from sqlalchemy.orm import Session
from datetime import datetime
from typing import Optional, List
from app.models.contact import Contact
from app.schemas.contact import ContactCreate, ContactUpdate

class ContactService:
    def __init__(self, db: Session):
        self.db = db

    def list_active(self, skip: int = 0, limit: int = 100) -> List[Contact]:
        return (
            self.db.query(Contact)
            .filter(Contact.deleted_at.is_(None))
            .order_by(Contact.name.asc())
            .offset(skip)
            .limit(limit)
            .all()
        )

    def get_active(self, contact_id: int) -> Optional[Contact]:
        return (
            self.db.query(Contact)
            .filter(Contact.id == contact_id, Contact.deleted_at.is_(None))
            .first()
        )

    def create(self, data: ContactCreate) -> Contact:
        contact = Contact(**data.model_dump())
        self.db.add(contact)
        self.db.commit()
        self.db.refresh(contact)
        return contact

    def update(self, contact_id: int, data: ContactUpdate) -> Optional[Contact]:
        contact = self.get_active(contact_id)
        if not contact:
            return None
        update_data = data.model_dump(exclude_none=True)
        for key, val in update_data.items():
            setattr(contact, key, val)
        self.db.commit()
        self.db.refresh(contact)
        return contact

    def soft_delete(self, contact_id: int) -> bool:
        contact = self.get_active(contact_id)
        if not contact:
            return False
        contact.soft_delete()
        self.db.commit()
        return True

    def restore(self, contact_id: int) -> bool:
        contact = self.db.query(Contact).filter(Contact.id == contact_id).first()
        if not contact or contact.deleted_at is None:
            return False
        contact.restore()
        self.db.commit()
        return True
```

---

### Task 14: Note service

**Objective:** Implement CRUD + soft-delete for contact notes.

**Files:**
- Create: `CRM/app/services/note_service.py`

**Step 1: Write `app/services/note_service.py`**

```python
from sqlalchemy.orm import Session
from typing import Optional, List
from app.models.contact_note import ContactNote
from app.schemas.note import NoteCreate, NoteUpdate

class NoteService:
    def __init__(self, db: Session):
        self.db = db

    def list_for_contact(self, contact_id: int, skip: int = 0, limit: int = 50) -> List[ContactNote]:
        return (
            self.db.query(ContactNote)
            .filter(ContactNote.contact_id == contact_id, ContactNote.deleted_at.is_(None))
            .order_by(ContactNote.created_at.desc())
            .offset(skip)
            .limit(limit)
            .all()
        )

    def get_active(self, note_id: int) -> Optional[ContactNote]:
        return (
            self.db.query(ContactNote)
            .filter(ContactNote.id == note_id, ContactNote.deleted_at.is_(None))
            .first()
        )

    def create(self, contact_id: int, data: NoteCreate) -> ContactNote:
        note = ContactNote(contact_id=contact_id, **data.model_dump())
        self.db.add(note)
        self.db.commit()
        self.db.refresh(note)
        return note

    def update(self, note_id: int, data: NoteUpdate) -> Optional[ContactNote]:
        note = self.get_active(note_id)
        if not note:
            return None
        update_data = data.model_dump(exclude_none=True)
        for key, val in update_data.items():
            setattr(note, key, val)
        self.db.commit()
        self.db.refresh(note)
        return note

    def soft_delete(self, note_id: int) -> bool:
        note = self.get_active(note_id)
        if not note:
            return False
        note.soft_delete()
        self.db.commit()
        return True
```

---

### Task 15: File service

**Objective:** Handle file uploads to disk and CRUD tracking.

**Files:**
- Create: `CRM/app/services/file_service.py`

**Step 1: Write `app/services/file_service.py`**

```python
import uuid
import os
from sqlalchemy.orm import Session
from typing import Optional, List
from fastapi import UploadFile
from app.models.file import File
from app.config import settings

ALLOWED_TYPES = {
    "image/jpeg", "image/png", "image/gif", "image/webp",
    "application/pdf", "text/plain", "text/csv",
    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
}

class FileService:
    def __init__(self, db: Session):
        self.db = db

    def list_for_contact(self, contact_id: Optional[int] = None, skip: int = 0, limit: int = 50) -> List[File]:
        q = self.db.query(File).filter(File.deleted_at.is_(None))
        if contact_id is not None:
            q = q.filter(File.contact_id == contact_id)
        return q.order_by(File.created_at.desc()).offset(skip).limit(limit).all()

    def upload(self, file: UploadFile, contact_id: Optional[int] = None) -> File:
        # Validate file type
        if file.content_type and file.content_type not in ALLOWED_TYPES:
            raise ValueError(f"Unsupported file type: {file.content_type}")

        stored_name = f"{uuid.uuid4().hex}_{file.filename}"
        upload_dir = settings.upload_dir
        os.makedirs(upload_dir, exist_ok=True)
        dest_path = os.path.join(upload_dir, stored_name)

        # Read and write file
        content = file.file.read()
        with open(dest_path, "wb") as f:
            f.write(content)

        db_file = File(
            contact_id=contact_id,
            original_name=file.filename or "unknown",
            stored_name=stored_name,
            mime_type=file.content_type or "application/octet-stream",
            size=len(content),
            path=stored_name,
        )
        self.db.add(db_file)
        self.db.commit()
        self.db.refresh(db_file)
        return db_file

    def soft_delete(self, file_id: int) -> bool:
        f = self.db.query(File).filter(File.id == file_id, File.deleted_at.is_(None)).first()
        if not f:
            return False
        f.soft_delete()
        self.db.commit()
        return True
```

---

### Task 16: Email service

**Objective:** Implement CRUD + soft-delete for emails.

**Files:**
- Create: `CRM/app/services/email_service.py`

**Step 1: Write `app/services/email_service.py`**

```python
from sqlalchemy.orm import Session
from typing import Optional, List
from app.models.email import Email
from app.schemas.email import EmailCreate, EmailUpdate

class EmailService:
    def __init__(self, db: Session):
        self.db = db

    def list_for_contact(self, contact_id: Optional[int] = None, skip: int = 0, limit: int = 50) -> List[Email]:
        q = self.db.query(Email).filter(Email.deleted_at.is_(None))
        if contact_id is not None:
            q = q.filter(Email.contact_id == contact_id)
        return q.order_by(Email.received_at.desc().nulls_last(), Email.created_at.desc()).offset(skip).limit(limit).all()

    def get_active(self, email_id: int) -> Optional[Email]:
        return self.db.query(Email).filter(Email.id == email_id, Email.deleted_at.is_(None)).first()

    def create(self, data: EmailCreate) -> Email:
        email = Email(**data.model_dump())
        self.db.add(email)
        self.db.commit()
        self.db.refresh(email)
        return email

    def update(self, email_id: int, data: EmailUpdate) -> Optional[Email]:
        email = self.get_active(email_id)
        if not email:
            return None
        update_data = data.model_dump(exclude_none=True)
        for key, val in update_data.items():
            setattr(email, key, val)
        self.db.commit()
        self.db.refresh(email)
        return email

    def soft_delete(self, email_id: int) -> bool:
        email = self.get_active(email_id)
        if not email:
            return False
        email.soft_delete()
        self.db.commit()
        return True
```

---

## Phase 4 — REST API Endpoints (for external apps)

### Task 17: API contacts router

**Objective:** Expose `/api/contacts/` endpoints for external apps (email utility).

**Files:**
- Create: `CRM/app/api/__init__.py`
- Create: `CRM/app/api/contacts.py`

**Step 1: Write `app/api/contacts.py`**

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import List
from app.database import get_db
from app.services.contact_service import ContactService
from app.schemas.contact import ContactCreate, ContactUpdate, ContactResponse

router = APIRouter(tags=["API Contacts"])

@router.get("/contacts", response_model=List[ContactResponse])
def list_contacts(skip: int = Query(0, ge=0), limit: int = Query(100, ge=1, le=500), db: Session = Depends(get_db)):
    svc = ContactService(db)
    return svc.list_active(skip=skip, limit=limit)

@router.get("/contacts/{contact_id}", response_model=ContactResponse)
def get_contact(contact_id: int, db: Session = Depends(get_db)):
    svc = ContactService(db)
    contact = svc.get_active(contact_id)
    if not contact:
        raise HTTPException(404, "Contact not found")
    return contact

@router.post("/contacts", response_model=ContactResponse, status_code=201)
def create_contact(data: ContactCreate, db: Session = Depends(get_db)):
    svc = ContactService(db)
    return svc.create(data)

@router.put("/contacts/{contact_id}", response_model=ContactResponse)
def update_contact(contact_id: int, data: ContactUpdate, db: Session = Depends(get_db)):
    svc = ContactService(db)
    result = svc.update(contact_id, data)
    if not result:
        raise HTTPException(404, "Contact not found")
    return result

@router.delete("/contacts/{contact_id}", status_code=204)
def delete_contact(contact_id: int, db: Session = Depends(get_db)):
    svc = ContactService(db)
    if not svc.soft_delete(contact_id):
        raise HTTPException(404, "Contact not found")
    return None
```

---

### Task 18: API notes router

**Objective:** Expose `/api/contacts/{contact_id}/notes` endpoints.

**Files:**
- Create: `CRM/app/api/notes.py`

**Step 1: Write `app/api/notes.py`**

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import List
from app.database import get_db
from app.services.note_service import NoteService
from app.services.contact_service import ContactService
from app.schemas.note import NoteCreate, NoteUpdate, NoteResponse

router = APIRouter(tags=["API Notes"])

@router.get("/contacts/{contact_id}/notes", response_model=List[NoteResponse])
def list_notes(contact_id: int, skip: int = Query(0, ge=0), limit: int = Query(50, ge=1, le=200), db: Session = Depends(get_db)):
    # Verify contact exists
    cs = ContactService(db)
    if not cs.get_active(contact_id):
        raise HTTPException(404, "Contact not found")
    ns = NoteService(db)
    return ns.list_for_contact(contact_id, skip=skip, limit=limit)

@router.get("/contacts/{contact_id}/notes/{note_id}", response_model=NoteResponse)
def get_note(contact_id: int, note_id: int, db: Session = Depends(get_db)):
    ns = NoteService(db)
    note = ns.get_active(note_id)
    if not note or note.contact_id != contact_id:
        raise HTTPException(404, "Note not found")
    return note

@router.post("/contacts/{contact_id}/notes", response_model=NoteResponse, status_code=201)
def create_note(contact_id: int, data: NoteCreate, db: Session = Depends(get_db)):
    cs = ContactService(db)
    if not cs.get_active(contact_id):
        raise HTTPException(404, "Contact not found")
    ns = NoteService(db)
    return ns.create(contact_id, data)

@router.put("/contacts/{contact_id}/notes/{note_id}", response_model=NoteResponse)
def update_note(contact_id: int, note_id: int, data: NoteUpdate, db: Session = Depends(get_db)):
    ns = NoteService(db)
    note = ns.get_active(note_id)
    if not note or note.contact_id != contact_id:
        raise HTTPException(404, "Note not found")
    result = ns.update(note_id, data)
    if not result:
        raise HTTPException(404, "Note not found")
    return result

@router.delete("/contacts/{contact_id}/notes/{note_id}", status_code=204)
def delete_note(contact_id: int, note_id: int, db: Session = Depends(get_db)):
    ns = NoteService(db)
    note = ns.get_active(note_id)
    if not note or note.contact_id != contact_id:
        raise HTTPException(404, "Note not found")
    ns.soft_delete(note_id)
    return None
```

---

### Task 19: API files router

**Objective:** Expose `/api/files/` endpoints for upload/list/delete.

**Files:**
- Create: `CRM/app/api/files.py`

**Step 1: Write `app/api/files.py`**

```python
from fastapi import APIRouter, Depends, HTTPException, Query, UploadFile, File as FileForm
from sqlalchemy.orm import Session
from typing import List, Optional
from app.database import get_db
from app.services.file_service import FileService
from app.schemas.file import FileResponse

router = APIRouter(tags=["API Files"])

@router.get("/files", response_model=List[FileResponse])
def list_files(
    contact_id: Optional[int] = Query(None),
    skip: int = Query(0, ge=0),
    limit: int = Query(50, ge=1, le=200),
    db: Session = Depends(get_db),
):
    fs = FileService(db)
    return fs.list_for_contact(contact_id=contact_id, skip=skip, limit=limit)

@router.post("/files", response_model=FileResponse, status_code=201)
def upload_file(
    file: UploadFile = FileForm(...),
    contact_id: Optional[int] = Query(None),
    db: Session = Depends(get_db),
):
    fs = FileService(db)
    try:
        return fs.upload(file, contact_id=contact_id)
    except ValueError as e:
        raise HTTPException(400, str(e))

@router.delete("/files/{file_id}", status_code=204)
def delete_file(file_id: int, db: Session = Depends(get_db)):
    fs = FileService(db)
    if not fs.soft_delete(file_id):
        raise HTTPException(404, "File not found")
    return None
```

---

### Task 20: API emails router

**Objective:** Expose `/api/emails/` endpoints.

**Files:**
- Create: `CRM/app/api/emails.py`

**Step 1: Write `app/api/emails.py`**

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import List, Optional
from app.database import get_db
from app.services.email_service import EmailService
from app.schemas.email import EmailCreate, EmailUpdate, EmailResponse

router = APIRouter(tags=["API Emails"])

@router.get("/emails", response_model=List[EmailResponse])
def list_emails(
    contact_id: Optional[int] = Query(None),
    skip: int = Query(0, ge=0),
    limit: int = Query(50, ge=1, le=200),
    db: Session = Depends(get_db),
):
    es = EmailService(db)
    return es.list_for_contact(contact_id=contact_id, skip=skip, limit=limit)

@router.get("/emails/{email_id}", response_model=EmailResponse)
def get_email(email_id: int, db: Session = Depends(get_db)):
    es = EmailService(db)
    email = es.get_active(email_id)
    if not email:
        raise HTTPException(404, "Email not found")
    return email

@router.post("/emails", response_model=EmailResponse, status_code=201)
def create_email(data: EmailCreate, db: Session = Depends(get_db)):
    es = EmailService(db)
    return es.create(data)

@router.put("/emails/{email_id}", response_model=EmailResponse)
def update_email(email_id: int, data: EmailUpdate, db: Session = Depends(get_db)):
    es = EmailService(db)
    result = es.update(email_id, data)
    if not result:
        raise HTTPException(404, "Email not found")
    return result

@router.delete("/emails/{email_id}", status_code=204)
def delete_email(email_id: int, db: Session = Depends(get_db)):
    es = EmailService(db)
    if not es.soft_delete(email_id):
        raise HTTPException(404, "Email not found")
    return None
```

---

## Phase 5 — Web Dashboard UI

### Task 21: Base template (Tailwind + Alpine.js layout)

**Objective:** Create the Jinja2 base template with layout, navigation, and scripts.

**Files:**
- Create: `CRM/app/templates/base.html`

**Step 1: Write `app/templates/base.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CRM — {% block title %}Dashboard{% endblock %}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x/dist/cdn.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
</head>
<body class="bg-gray-50">
    <div class="flex h-screen">
        <!-- Sidebar -->
        <aside class="w-64 bg-white border-r border-gray-200 p-4">
            <div class="text-xl font-bold text-gray-800 mb-6">📇 CRM</div>
            <nav class="space-y-1">
                <a href="/" class="flex items-center gap-3 px-3 py-2 rounded hover:bg-blue-50 text-gray-700">
                    <i class="fas fa-th-large"></i> Dashboard
                </a>
                <a href="/contacts" class="flex items-center gap-3 px-3 py-2 rounded hover:bg-blue-50 text-gray-700">
                    <i class="fas fa-address-book"></i> Contacts
                </a>
                <a href="/files" class="flex items-center gap-3 px-3 py-2 rounded hover:bg-blue-50 text-gray-700">
                    <i class="fas fa-folder"></i> Files
                </a>
                <a href="/emails" class="flex items-center gap-3 px-3 py-2 rounded hover:bg-blue-50 text-gray-700">
                    <i class="fas fa-envelope"></i> Emails
                </a>
            </nav>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 overflow-y-auto p-6">
            {% block content %}{% endblock %}
        </main>
    </div>
</body>
</html>
```

---

### Task 22: Dashboard landing page

**Objective:** Create the home page showing summary counts.

**Files:**
- Create: `CRM/app/web/__init__.py`
- Create: `CRM/app/web/dashboard.py`
- Create: `CRM/app/templates/index.html`

**Step 1: Write `app/web/dashboard.py`**

```python
from fastapi import APIRouter, Depends, Request
from fastapi.responses import HTMLResponse
from sqlalchemy.orm import Session
from app.database import get_db
from app.models.contact import Contact
from app.models.contact_note import ContactNote
from app.models.file import File
from app.models.email import Email

router = APIRouter(tags=["Web Dashboard"])

@router.get("/", response_class=HTMLResponse)
def dashboard(request: Request, db: Session = Depends(get_db)):
    contact_count = db.query(Contact).filter(Contact.deleted_at.is_(None)).count()
    note_count = db.query(ContactNote).filter(ContactNote.deleted_at.is_(None)).count()
    file_count = db.query(File).filter(File.deleted_at.is_(None)).count()
    email_count = db.query(Email).filter(Email.deleted_at.is_(None)).count()
    return request.app.state.templates.render(
        "index.html",
        request=request,
        counts={
            "contacts": contact_count,
            "notes": note_count,
            "files": file_count,
            "emails": email_count,
        },
    )
```

**Step 2: Write `app/templates/index.html`**

```html
{% extends "base.html" %}
{% block title %}Dashboard{% endblock %}
{% block content %}
<div class="grid grid-cols-4 gap-6">
    <div class="bg-white rounded-lg shadow p-6">
        <div class="text-3xl font-bold text-blue-600">{{ counts.contacts }}</div>
        <div class="text-gray-500 mt-1">Contacts</div>
    </div>
    <div class="bg-white rounded-lg shadow p-6">
        <div class="text-3xl font-bold text-green-600">{{ counts.notes }}</div>
        <div class="text-gray-500 mt-1">Notes</div>
    </div>
    <div class="bg-white rounded-lg shadow p-6">
        <div class="text-3xl font-bold text-amber-600">{{ counts.files }}</div>
        <div class="text-gray-500 mt-1">Files</div>
    </div>
    <div class="bg-white rounded-lg shadow p-6">
        <div class="text-3xl font-bold text-purple-600">{{ counts.emails }}</div>
        <div class="text-gray-500 mt-1">Emails</div>
    </div>
</div>
{% endblock %}
```

---

### Task 23: Web contacts router — list and detail

**Objective:** Serve HTML pages for contact list, detail, and create/edit forms.

**Files:**
- Create: `CRM/app/web/contacts.py`
- Create: `CRM/app/templates/contacts/list.html`
- Create: `CRM/app/templates/contacts/detail.html`
- Create: `CRM/app/templates/contacts/form.html`

**Step 1: Write `app/web/contacts.py`**

```python
from fastapi import APIRouter, Depends, Request, Form, HTTPException, Query
from fastapi.responses import HTMLResponse, RedirectResponse
from sqlalchemy.orm import Session
from app.database import get_db
from app.services.contact_service import ContactService
from app.schemas.contact import ContactCreate

router = APIRouter(tags=["Web Contacts"])

@router.get("/contacts", response_class=HTMLResponse)
def list_contacts(request: Request, db: Session = Depends(get_db),
                  q: str = Query(""), skip: int = Query(0, ge=0)):
    svc = ContactService(db)
    contacts = svc.list_active(skip=skip, limit=100)
    return request.app.state.templates.render(
        "contacts/list.html", request=request, contacts=contacts, q=q
    )

@router.get("/contacts/{contact_id}", response_class=HTMLResponse)
def detail_contact(request: Request, contact_id: int, db: Session = Depends(get_db)):
    svc = ContactService(db)
    contact = svc.get_active(contact_id)
    if not contact:
        raise HTTPException(404, "Contact not found")
    return request.app.state.templates.render(
        "contacts/detail.html", request=request, contact=contact
    )

@router.get("/contacts/new", response_class=HTMLResponse)
def new_contact_form(request: Request):
    return request.app.state.templates.render(
        "contacts/form.html", request=request, contact=None
    )

@router.post("/contacts/new")
def create_contact_post(request: Request, name: str = Form(...), email: str = Form(""),
                        phone: str = Form(""), company: str = Form(""),
                        position: str = Form(""), notes: str = Form(""),
                        db: Session = Depends(get_db)):
    svc = ContactService(db)
    data = ContactCreate(
        name=name,
        email=email or None,
        phone=phone or None,
        company=company or None,
        position=position or None,
        notes=notes or None,
    )
    contact = svc.create(data)
    return RedirectResponse(f"/contacts/{contact.id}", status_code=303)

@router.get("/contacts/{contact_id}/edit", response_class=HTMLResponse)
def edit_contact_form(request: Request, contact_id: int, db: Session = Depends(get_db)):
    svc = ContactService(db)
    contact = svc.get_active(contact_id)
    if not contact:
        raise HTTPException(404)
    return request.app.state.templates.render(
        "contacts/form.html", request=request, contact=contact
    )

@router.post("/contacts/{contact_id}/edit")
def update_contact_post(request: Request, contact_id: int,
                        name: str = Form(...), email: str = Form(""),
                        phone: str = Form(""), company: str = Form(""),
                        position: str = Form(""), notes: str = Form(""),
                        db: Session = Depends(get_db)):
    svc = ContactService(db)
    data = ContactCreate(
        name=name, email=email or None, phone=phone or None,
        company=company or None, position=position or None,
        notes=notes or None,
    )
    result = svc.update(contact_id, data)
    if not result:
        raise HTTPException(404)
    return RedirectResponse(f"/contacts/{contact_id}", status_code=303)

@router.post("/contacts/{contact_id}/delete")
def delete_contact_post(contact_id: int, db: Session = Depends(get_db)):
    svc = ContactService(db)
    svc.soft_delete(contact_id)
    return RedirectResponse("/contacts", status_code=303)
```

**Step 2: Write `app/templates/contacts/list.html`**

```html
{% extends "base.html" %}
{% block title %}Contacts{% endblock %}
{% block content %}
<div class="flex justify-between items-center mb-4">
    <h1 class="text-2xl font-bold text-gray-800">Contacts</h1>
    <a href="/contacts/new" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
        <i class="fas fa-plus"></i> New Contact
    </a>
</div>
<div class="bg-white rounded-lg shadow overflow-hidden">
    <table class="w-full">
        <thead class="bg-gray-50 text-gray-600 text-sm">
            <tr>
                <th class="px-4 py-3 text-left">Name</th>
                <th class="px-4 py-3 text-left">Email</th>
                <th class="px-4 py-3 text-left">Company</th>
                <th class="px-4 py-3 text-center">Actions</th>
            </tr>
        </thead>
        <tbody class="divide-y divide-gray-200">
            {% for c in contacts %}
            <tr class="hover:bg-gray-50">
                <td class="px-4 py-3"><a href="/contacts/{{ c.id }}" class="text-blue-600 hover:underline">{{ c.name }}</a></td>
                <td class="px-4 py-3 text-gray-600">{{ c.email or '' }}</td>
                <td class="px-4 py-3 text-gray-600">{{ c.company or '' }}</td>
                <td class="px-4 py-3 text-center">
                    <a href="/contacts/{{ c.id }}/edit" class="text-blue-500 hover:text-blue-700 mr-2"><i class="fas fa-edit"></i></a>
                    <form action="/contacts/{{ c.id }}/delete" method="post" class="inline" onsubmit="return confirm('Delete this contact?')">
                        <button type="submit" class="text-red-500 hover:text-red-700"><i class="fas fa-trash"></i></button>
                    </form>
                </td>
            </tr>
            {% else %}
            <tr><td colspan="4" class="px-4 py-8 text-center text-gray-400">No contacts yet. Create one!</td></tr>
            {% endfor %}
        </tbody>
    </table>
</div>
{% endblock %}
```

**Step 3: Write `app/templates/contacts/detail.html`**

```html
{% extends "base.html" %}
{% block title %}{{ contact.name }}{% endblock %}
{% block content %}
<div class="bg-white rounded-lg shadow p-6">
    <div class="flex justify-between items-start mb-6">
        <div>
            <h1 class="text-2xl font-bold">{{ contact.name }}</h1>
            <div class="text-gray-500 mt-1 text-sm">Created {{ contact.created_at.strftime('%b %d, %Y') }}</div>
        </div>
        <div class="flex gap-2">
            <a href="/contacts/{{ contact.id }}/edit" class="bg-blue-600 text-white px-3 py-1.5 rounded text-sm hover:bg-blue-700">Edit</a>
            <form action="/contacts/{{ contact.id }}/delete" method="post" onsubmit="return confirm('Delete?')">
                <button class="bg-red-600 text-white px-3 py-1.5 rounded text-sm hover:bg-red-700">Delete</button>
            </form>
        </div>
    </div>

    <div class="grid grid-cols-2 gap-4 mb-6">
        <div><span class="font-semibold text-gray-600">Email:</span> {{ contact.email or '—' }}</div>
        <div><span class="font-semibold text-gray-600">Phone:</span> {{ contact.phone or '—' }}</div>
        <div><span class="font-semibold text-gray-600">Company:</span> {{ contact.company or '—' }}</div>
        <div><span class="font-semibold text-gray-600">Position:</span> {{ contact.position or '—' }}</div>
    </div>

    {% if contact.notes %}
    <div class="mb-6">
        <h3 class="font-semibold text-gray-600 mb-2">Notes / Bio</h3>
        <p class="text-gray-700">{{ contact.notes }}</p>
    </div>
    {% endif %}

    <!-- Notes section -->
    <div x-data="{ showNoteForm: false }" class="mb-6">
        <h3 class="font-semibold text-gray-600 mb-2 flex justify-between items-center">
            <span>Interaction Notes</span>
            <button @click="showNoteForm = !showNoteForm" class="text-blue-600 text-sm hover:underline">+ Add Note</button>
        </h3>
        <div x-show="showNoteForm" class="mb-4">
            <form action="/contacts/{{ contact.id }}/notes/new" method="post" class="space-y-2">
                <input name="title" placeholder="Title (optional)" class="w-full border rounded px-3 py-2 text-sm">
                <textarea name="content" rows="3" placeholder="Note content..." class="w-full border rounded px-3 py-2 text-sm" required></textarea>
                <button class="bg-blue-600 text-white px-3 py-1.5 rounded text-sm hover:bg-blue-700">Save</button>
            </form>
        </div>
        <div class="space-y-2">
            {% for note in contact.contact_notes.filter(contact.contact_notes.deleted_at.is_(None)).limit(10).all() %}
            <div class="border rounded p-3 text-sm">
                <div class="font-medium">{{ note.title or 'Note' }}</div>
                <div class="text-gray-600 mt-1">{{ note.content }}</div>
                <div class="text-gray-400 text-xs mt-1">{{ note.created_at.strftime('%b %d, %Y %H:%M') }}</div>
                <form action="/contacts/{{ contact.id }}/notes/{{ note.id }}/delete" method="post" class="inline mt-1">
                    <button class="text-red-500 text-xs hover:underline">Delete</button>
                </form>
            </div>
            {% else %}
            <div class="text-gray-400 text-sm">No notes yet.</div>
            {% endfor %}
        </div>
    </div>

    <!-- Files section -->
    <div class="mb-6">
        <h3 class="font-semibold text-gray-600 mb-2">Files</h3>
        <form action="/contacts/{{ contact.id }}/files/upload" method="post" enctype="multipart/form-data" class="mb-3">
            <input type="file" name="file" class="text-sm border rounded px-3 py-2">
            <button class="bg-blue-600 text-white px-3 py-1.5 rounded text-sm hover:bg-blue-700 ml-2">Upload</button>
        </form>
        <div class="space-y-1">
            {% for f in contact.files.filter(contact.files.deleted_at.is_(None)).limit(10).all() %}
            <div class="text-sm flex justify-between">
                <a href="/uploads/{{ f.stored_name }}" target="_blank" class="text-blue-600 hover:underline">{{ f.original_name }}</a>
                <span class="text-gray-400 text-xs">{{ f.size // 1024 }} KB</span>
            </div>
            {% else %}
            <div class="text-gray-400 text-sm">No files yet.</div>
            {% endfor %}
        </div>
    </div>

    <!-- Emails section -->
    <div>
        <h3 class="font-semibold text-gray-600 mb-2">Emails</h3>
        <div class="space-y-2">
            {% for e in contact.emails.filter(contact.emails.deleted_at.is_(None)).limit(10).all() %}
            <div class="border rounded p-3 text-sm">
                <div class="font-medium">{{ e.subject }}</div>
                <div class="text-gray-600 mt-1 truncate">{{ e.body[:200] }}{% if e.body|length > 200 %}…{% endif %}</div>
                <div class="text-gray-400 text-xs mt-1">
                    {% if e.received_at %}{{ e.received_at.strftime('%b %d, %Y') }} — {% endif %}
                    {{ e.sender or '' }}
                </div>
            </div>
            {% else %}
            <div class="text-gray-400 text-sm">No emails yet.</div>
            {% endfor %}
        </div>
    </div>
</div>
{% endblock %}
```

**Step 4: Write `app/templates/contacts/form.html`**

```html
{% extends "base.html" %}
{% block title %}{% if contact %}Edit{% else %}New{% endif %} Contact{% endblock %}
{% block content %}
<div class="max-w-lg mx-auto bg-white rounded-lg shadow p-6">
    <h1 class="text-2xl font-bold mb-4">{% if contact %}Edit{% else %}New{% endif %} Contact</h1>
    <form action="{% if contact %}/contacts/{{ contact.id }}/edit{% else %}/contacts/new{% endif %}" method="post" class="space-y-4">
        <div>
            <label class="block text-sm font-medium text-gray-700">Name *</label>
            <input name="name" value="{{ contact.name if contact else '' }}" class="w-full border rounded px-3 py-2" required>
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Email</label>
            <input name="email" value="{{ contact.email or '' }}" class="w-full border rounded px-3 py-2">
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Phone</label>
            <input name="phone" value="{{ contact.phone or '' }}" class="w-full border rounded px-3 py-2">
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Company</label>
            <input name="company" value="{{ contact.company or '' }}" class="w-full border rounded px-3 py-2">
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Position</label>
            <input name="position" value="{{ contact.position or '' }}" class="w-full border rounded px-3 py-2">
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Notes / Bio</label>
            <textarea name="notes" rows="4" class="w-full border rounded px-3 py-2">{{ contact.notes or '' }}</textarea>
        </div>
        <div class="flex gap-2">
            <button class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">Save</button>
            <a href="/contacts" class="bg-gray-200 text-gray-700 px-4 py-2 rounded hover:bg-gray-300">Cancel</a>
        </div>
    </form>
</div>
{% endblock %}
```

---

### Task 24: Web notes router — inline CRUD via forms

**Objective:** Handle note create/delete form submissions (inline on contact detail page).

**Files:**
- Create: `CRM/app/web/notes.py`

**Step 1: Write `app/web/notes.py`**

```python
from fastapi import APIRouter, Depends, Request, Form, HTTPException
from fastapi.responses import RedirectResponse
from sqlalchemy.orm import Session
from app.database import get_db
from app.services.note_service import NoteService
from app.services.contact_service import ContactService

router = APIRouter(tags=["Web Notes"])

@router.post("/contacts/{contact_id}/notes/new")
def create_note_post(contact_id: int, title: str = Form(""), content: str = Form(...), db: Session = Depends(get_db)):
    cs = ContactService(db)
    if not cs.get_active(contact_id):
        raise HTTPException(404, "Contact not found")
    ns = NoteService(db)
    ns.create(contact_id, title=title, content=content)
    return RedirectResponse(f"/contacts/{contact_id}", status_code=303)

@router.post("/contacts/{contact_id}/notes/{note_id}/delete")
def delete_note_post(contact_id: int, note_id: int, db: Session = Depends(get_db)):
    ns = NoteService(db)
    ns.soft_delete(note_id)
    return RedirectResponse(f"/contacts/{contact_id}", status_code=303)
```

---

### Task 25: Web files router — upload and delete via forms

**Objective:** Handle file upload/delete on contact detail page.

**Files:**
- Create: `CRM/app/web/files.py`

**Step 1: Write `app/web/files.py`**

```python
from fastapi import APIRouter, Depends, Request, HTTPException, UploadFile, File as FileForm
from fastapi.responses import RedirectResponse, HTMLResponse
from sqlalchemy.orm import Session
from app.database import get_db
from app.services.file_service import FileService
from app.models.file import File

router = APIRouter(tags=["Web Files"])

@router.post("/contacts/{contact_id}/files/upload")
def upload_file_post(contact_id: int, file: UploadFile = FileForm(...), db: Session = Depends(get_db)):
    fs = FileService(db)
    try:
        fs.upload(file, contact_id=contact_id)
    except ValueError:
        raise HTTPException(400, "Unsupported file type")
    return RedirectResponse(f"/contacts/{contact_id}", status_code=303)

@router.post("/contacts/{contact_id}/files/{file_id}/delete")
def delete_file_post(contact_id: int, file_id: int, db: Session = Depends(get_db)):
    fs = FileService(db)
    fs.soft_delete(file_id)
    return RedirectResponse(f"/contacts/{contact_id}", status_code=303)

@router.get("/files", response_class=HTMLResponse)
def list_all_files(request: Request, db: Session = Depends(get_db)):
    fs = FileService(db)
    files = fs.list_for_contact(skip=0, limit=100)
    return request.app.state.templates.render(
        "files/list.html", request=request, files=files
    )
```

**Step 2: Create `app/templates/files/list.html`**

```html
{% extends "base.html" %}
{% block title %}Files{% endblock %}
{% block content %}
<h1 class="text-2xl font-bold mb-4">All Files</h1>
<div class="bg-white rounded-lg shadow overflow-hidden">
    <table class="w-full">
        <thead class="bg-gray-50 text-gray-600 text-sm">
            <tr>
                <th class="px-4 py-3 text-left">Name</th>
                <th class="px-4 py-3 text-left">Type</th>
                <th class="px-4 py-3 text-left">Size</th>
                <th class="px-4 py-3 text-left">Contact</th>
                <th class="px-4 py-3 text-center">Actions</th>
            </tr>
        </thead>
        <tbody class="divide-y divide-gray-200">
            {% for f in files %}
            <tr class="hover:bg-gray-50">
                <td class="px-4 py-3"><a href="/uploads/{{ f.stored_name }}" target="_blank" class="text-blue-600 hover:underline">{{ f.original_name }}</a></td>
                <td class="px-4 py-3 text-gray-600">{{ f.mime_type }}</td>
                <td class="px-4 py-3 text-gray-600">{{ f.size // 1024 }} KB</td>
                <td class="px-4 py-3 text-gray-600">
                    {% if f.contact %}<a href="/contacts/{{ f.contact.id }}" class="text-blue-600 hover:underline">{{ f.contact.name }}</a>{% endif %}
                </td>
                <td class="px-4 py-3 text-center">
                    <form action="/contacts/{{ f.contact_id or 0 }}/files/{{ f.id }}/delete" method="post" class="inline">
                        <button class="text-red-500 hover:text-red-700"><i class="fas fa-trash"></i></button>
                    </form>
                </td>
            </tr>
            {% else %}
            <tr><td colspan="5" class="px-4 py-8 text-center text-gray-400">No files uploaded yet.</td></tr>
            {% endfor %}
        </tbody>
    </table>
</div>
{% endblock %}
```

---

### Task 26: Web emails router — list and detail views

**Objective:** Create the emails dashboard pages.

**Files:**
- Create: `CRM/app/web/emails.py`
- Create: `CRM/app/templates/emails/list.html`

**Step 1: Write `app/web/emails.py`**

```python
from fastapi import APIRouter, Depends, Request, HTTPException, Query, Form
from fastapi.responses import HTMLResponse, RedirectResponse
from sqlalchemy.orm import Session
from app.database import get_db
from app.services.email_service import EmailService
from app.schemas.email import EmailCreate

router = APIRouter(tags=["Web Emails"])

@router.get("/emails", response_class=HTMLResponse)
def list_emails(request: Request, db: Session = Depends(get_db)):
    es = EmailService(db)
    emails = es.list_for_contact(skip=0, limit=100)
    return request.app.state.templates.render(
        "emails/list.html", request=request, emails=emails
    )

@router.get("/emails/new", response_class=HTMLResponse)
def new_email_form(request: Request):
    return request.app.state.templates.render(
        "emails/form.html", request=request
    )

@router.post("/emails/new")
def create_email_post(request: Request, subject: str = Form(...), body: str = Form(...),
                      sender: str = Form(""), recipient: str = Form(""),
                      contact_id: int = Form(0), db: Session = Depends(get_db)):
    es = EmailService(db)
    data = EmailCreate(
        subject=subject,
        body=body,
        sender=sender or None,
        recipient=recipient or None,
        contact_id=contact_id or None,
    )
    es.create(data)
    return RedirectResponse("/emails", status_code=303)

@router.post("/emails/{email_id}/delete")
def delete_email_post(email_id: int, db: Session = Depends(get_db)):
    es = EmailService(db)
    es.soft_delete(email_id)
    return RedirectResponse("/emails", status_code=303)
```

**Step 2: Write `app/templates/emails/list.html`**

```html
{% extends "base.html" %}
{% block title %}Emails{% endblock %}
{% block content %}
<div class="flex justify-between items-center mb-4">
    <h1 class="text-2xl font-bold text-gray-800">Emails</h1>
    <a href="/emails/new" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">+ New Email</a>
</div>
<div class="bg-white rounded-lg shadow overflow-hidden">
    <table class="w-full">
        <thead class="bg-gray-50 text-gray-600 text-sm">
            <tr>
                <th class="px-4 py-3 text-left">Subject</th>
                <th class="px-4 py-3 text-left">Sender</th>
                <th class="px-4 py-3 text-left">Recipient</th>
                <th class="px-4 py-3 text-left">Date</th>
                <th class="px-4 py-3 text-center">Actions</th>
            </tr>
        </thead>
        <tbody class="divide-y divide-gray-200">
            {% for e in emails %}
            <tr class="hover:bg-gray-50">
                <td class="px-4 py-3 font-medium">{{ e.subject }}</td>
                <td class="px-4 py-3 text-gray-600">{{ e.sender or '' }}</td>
                <td class="px-4 py-3 text-gray-600">{{ e.recipient or '' }}</td>
                <td class="px-4 py-3 text-gray-600">{{ e.received_at.strftime('%b %d, %Y') if e.received_at else e.created_at.strftime('%b %d, %Y') }}</td>
                <td class="px-4 py-3 text-center">
                    <form action="/emails/{{ e.id }}/delete" method="post" class="inline">
                        <button class="text-red-500 hover:text-red-700"><i class="fas fa-trash"></i></button>
                    </form>
                </td>
            </tr>
            {% else %}
            <tr><td colspan="5" class="px-4 py-8 text-center text-gray-400">No emails stored yet.</td></tr>
            {% endfor %}
        </tbody>
    </table>
</div>
{% endblock %}
```

---

## Phase 6 — Testing & Validation

### Task 27: Write smoke tests for API endpoints

**Objective:** Create pytest tests covering CRUD + soft-delete for all resources.

**Files:**
- Create: `CRM/tests/__init__.py`
- Create: `CRM/tests/conftest.py`
- Create: `CRM/tests/test_api.py`

**Step 1: Write `tests/conftest.py`**

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.database import get_db
from app.models.base import Base
from app import create_app
import tempfile, os

@pytest.fixture
def db_session():
    # In-memory SQLite for tests
    engine = create_engine("sqlite:///:memory:", connect_args={"check_same_thread": False})
    Base.metadata.create_all(bind=engine)
    TestSession = sessionmaker(bind=engine)
    session = TestSession()
    yield session
    session.close()

@pytest.fixture
def client(db_session):
    app = create_app()

    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    return TestClient(app)
```

**Step 2: Write `tests/test_api.py`**

```python
from app.schemas.contact import ContactCreate, ContactResponse
from app.schemas.note import NoteCreate, NoteResponse
from app.schemas.file import FileResponse
from app.schemas.email import EmailCreate, EmailResponse

def test_create_and_list_contacts(client):
    # Create
    resp = client.post("/api/contacts", json={"name": "Alice", "email": "alice@test.com"})
    assert resp.status_code == 201
    data = resp.json()
    assert data["name"] == "Alice"

    # List
    resp = client.get("/api/contacts")
    assert resp.status_code == 200
    assert len(resp.json()) == 1

def test_soft_delete_contact(client):
    client.post("/api/contacts", json={"name": "Bob"})
    resp = client.delete("/api/contacts/1")
    assert resp.status_code == 204

    # Should be gone from list
    resp = client.get("/api/contacts")
    assert len(resp.json()) == 0

    # But record still exists in DB — verify by direct query in another test

def test_create_and_delete_note(client):
    # Create contact first
    client.post("/api/contacts", json={"name": "Alice"})
    # Create note
    resp = client.post("/api/contacts/1/notes", json={"content": "Meeting notes"})
    assert resp.status_code == 201
    assert resp.json()["content"] == "Meeting notes"

    # Soft delete
    resp = client.delete("/api/contacts/1/notes/1")
    assert resp.status_code == 204

    # Should be gone
    resp = client.get("/api/contacts/1/notes")
    assert len(resp.json()) == 0

def test_upload_and_list_file(client):
    client.post("/api/contacts", json={"name": "Alice"})
    resp = client.post("/api/files?contact_id=1", files={"file": ("test.txt", b"hello world", "text/plain")})
    assert resp.status_code == 201
    file_id = resp.json()["id"]

    resp = client.get("/api/files?contact_id=1")
    assert resp.status_code == 200
    assert len(resp.json()) == 1

    resp = client.delete(f"/api/files/{file_id}")
    assert resp.status_code == 204

def test_create_email(client):
    resp = client.post("/api/emails", json={
        "subject": "Hello", "body": "Test body", "sender": "alice@test.com", "recipient": "bob@test.com"
    })
    assert resp.status_code == 201
    assert resp.json()["subject"] == "Hello"

    # Soft delete
    resp = client.delete("/api/emails/1")
    assert resp.status_code == 204
```

**Step 3: Run tests**

Run: `cd CRM && pip install pytest pytest-cov && python -m pytest tests/ -v`
Expected: 5 passed

---

## Phase 7 — Production Hardening

### Task 28: Add API key authentication for external endpoints

**Objective:** Protect `/api/*` routes with a simple API key header check for external apps.

**Files:**
- Modify: `CRM/app/config.py` (add `api_key` field)
- Create: `CRM/app/api/auth.py` (dependency function)

**Step 1: Update `app/config.py`**

```python
class Settings(BaseSettings):
    secret_key: str = "change-me"
    api_key: str = "change-me-to-a-secure-api-key"  # NEW
    database_path: str = "CRM/crm.db"
    upload_dir: str = "CRM/uploads"
    debug: bool = False
    ...
```

**Step 2: Create `app/api/auth.py`**

```python
from fastapi import HTTPException, Header
from app.config import settings

def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != settings.api_key:
        raise HTTPException(401, "Invalid API key")
    return True
```

**Step 3: Add dependency to each API router**

In each `app/api/*.py`, add `api_key: bool = Depends(verify_api_key)` as a hidden param to every route. Or create a shared `protected_router`:

```python
from app.api.auth import verify_api_key

router = APIRouter(dependencies=[Depends(verify_api_key)], tags=["API Contacts"])
```

---

### Task 29: Add request logging middleware

**Objective:** Log all API requests to stdout.

**Files:**
- Create: `CRM/app/middleware.py`
- Modify: `CRM/app/__init__.py` (register middleware)

**Step 1: Write `app/middleware.py`**

```python
from fastapi import Request
import logging

logger = logging.getLogger("crm")

async def log_requests(request: Request, call_next):
    logger.info(f"{request.method} {request.url.path}  params={dict(request.query_params)}")
    response = await call_next(request)
    logger.info(f"{request.method} {request.url.path} -> {response.status_code}")
    return response
```

**Step 2: Register in `app/__init__.py`**

```python
from app.middleware import log_requests
app.middleware("http")(log_requests)
```

---

### Task 30: Startup/shutdown + .env validation

**Objective:** Validate config on startup and create upload directory.

**Files:**
- Modify: `CRM/app/__init__.py`

**Step 1: Add lifespan events**

```python
from contextlib import asynccontextmanager
from app.config import settings

@asynccontextmanager
async def lifespan(app, **kwargs):
    # Validate critical settings
    if settings.secret_key == "change-me":
        raise ValueError("SECRET_KEY must be changed from default")
    if settings.api_key == "change-me-to-a-secure-api-key":
        raise ValueError("API_KEY must be changed from default")
    # Create upload dir
    from pathlib import Path
    Path(settings.upload_dir).mkdir(parents=True, exist_ok=True)
    yield

app = FastAPI(title="CRM", version="1.0.0", lifespan=lifespan)
```

---

### Task 31: CORS middleware for API access from external apps

**Objective:** Allow cross-origin requests to the API.

**Files:**
- Modify: `CRM/app/__init__.py`

**Step 1: Add CORS**

```python
from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # restrict in production
    allow_methods=["*"],
    allow_headers=["*"],
    allow_credentials=True,
)
```

---

### Task 32: Final run and verify

**Objective:** Start the server, verify dashboard and API work.

**Step 1: Run the app**

Run: `cd CRM && python run.py`
Expected: Uvicorn starts on `http://0.0.0.0:8080`

**Step 2: Verify dashboard**

Run: `curl -s http://localhost:8080/ | head -20`
Expected: HTML with CRM Dashboard title and count cards

**Step 3: Verify API**

Run: `curl -s http://localhost:8080/api/contacts`
Expected: `[]` (empty array — no contacts yet)

**Step 4: Test full cycle via API**

```bash
curl -X POST http://localhost:8080/api/contacts \
  -H "Content-Type: application/json" \
  -H "x-api-key: change-me-to-a-secure-api-key" \
  -d '{"name": "Test Contact", "email": "test@example.com"}'
```
Expected: JSON with id, name, created_at

---

## Files Changed Summary

| File | Action |
|---|---|
| `CRM/requirements.txt` | Create |
| `CRM/.env` | Create |
| `CRM/run.py` | Create |
| `CRM/app/__init__.py` | Create |
| `CRM/app/config.py` | Create |
| `CRM/app/database.py` | Create |
| `CRM/app/middleware.py` | Create |
| `CRM/app/models/base.py` | Create |
| `CRM/app/models/contact.py` | Create |
| `CRM/app/models/contact_note.py` | Create |
| `CRM/app/models/file.py` | Create |
| `CRM/app/models/email.py` | Create |
| `CRM/app/models/__init__.py` | Create |
| `CRM/app/schemas/contact.py` | Create |
| `CRM/app/schemas/note.py` | Create |
| `CRM/app/schemas/file.py` | Create |
| `CRM/app/schemas/email.py` | Create |
| `CRM/app/schemas/__init__.py` | Create |
| `CRM/app/services/contact_service.py` | Create |
| `CRM/app/services/note_service.py` | Create |
| `CRM/app/services/file_service.py` | Create |
| `CRM/app/services/email_service.py` | Create |
| `CRM/app/services/__init__.py` | Create |
| `CRM/app/api/auth.py` | Create |
| `CRM/app/api/contacts.py` | Create |
| `CRM/app/api/notes.py` | Create |
| `CRM/app/api/files.py` | Create |
| `CRM/app/api/emails.py` | Create |
| `CRM/app/api/__init__.py` | Create |
| `CRM/app/web/dashboard.py` | Create |
| `CRM/app/web/contacts.py` | Create |
| `CRM/app/web/notes.py` | Create |
| `CRM/app/web/files.py` | Create |
| `CRM/app/web/emails.py` | Create |
| `CRM/app/web/__init__.py` | Create |
| `CRM/app/templates/base.html` | Create |
| `CRM/app/templates/index.html` | Create |
| `CRM/app/templates/contacts/list.html` | Create |
| `CRM/app/templates/contacts/detail.html` | Create |
| `CRM/app/templates/contacts/form.html` | Create |
| `CRM/app/templates/files/list.html` | Create |
| `CRM/app/templates/emails/list.html` | Create |
| `CRM/app/templates/emails/form.html` | Create |
| `CRM/tests/conftest.py` | Create |
| `CRM/tests/test_api.py` | Create |
| `CRM/alembic.ini` | Create (via alembic init) |
| `CRM/migrations/env.py` | Modify (wire our engine + Base) |
| `CRM/migrations/versions/001_initial.py` | Create (via alembic) |

---

## Risks, Tradeoffs & Open Questions

**Risks:**
1. **SQLite concurrency** — SQLite is single-writer. For a single-user CRM this is fine, but if the email utility writes many records concurrently, consider switching to PostgreSQL. Mitigation: WAL mode (`PRAGMA journal_mode=WAL`) can be enabled for better concurrent reads.
2. **File storage on disk** — uploaded files are stored on the local filesystem. In production, migrate to S3/MinIO. The service layer abstracts this; swap the upload implementation without changing the API.
3. **API key in .env** — the API key is static. For production, add per-app API keys stored in a DB table so each external app gets its own key.

**Tradeoffs:**
- **Jinja2 templates vs SPA** — Server-rendered HTML with Alpine.js was chosen over Vue/React for simplicity. The detail page is richer than a list page would need. If the UI grows complex (drag-drop, real-time), migrate to a SPA frontend.
- **Soft delete via `deleted_at`** — simple and auditable. Downside: every query must filter `deleted_at IS NULL`. Mitigated by service-layer methods. Admin restore endpoints can be added later.
- **No pagination on web UI** — list pages show up to 100 items. Add pagination controls when contacts exceed that.

**Open Questions:**
1. Should file uploads be scoped to contacts only, or global? — Currently supports both (contact_id optional).
2. Email schema — does the external email utility need to store raw MIME or just subject/body? Assumed subject+body is sufficient; add `raw_mime` field if needed.
3. Search — should the contact list support text search? The API `list_contacts` could accept a `q` query param for name/email search. Add in v2.

---

## Execution Plan

After you review and approve this spec, the implementation follows this order:

1. **Phase 1** — Scaffold project structure, install deps, create models, run migration (Tasks 1–8)
2. **Phase 2** — Pydantic schemas (Tasks 9–12)
3. **Phase 3** — Service layer with soft-delete logic (Tasks 13–16)
4. **Phase 4** — REST API endpoints for external app integration (Tasks 17–20)
5. **Phase 5** — Web dashboard templates + routes (Tasks 21–26)
6. **Phase 6** — Test suite and validation (Task 27)
7. **Phase 7** — Production hardening: auth, logging, CORS, lifecycle (Tasks 28–32)

Each task is 2–5 minutes of focused work with TDD where applicable.

---

**Plan complete and saved. Ready to execute using subagent-driven-development — I'll dispatch a fresh subagent per task with two-stage review (spec compliance then code quality). Shall I proceed?**
