# Architecture

A high-level overview of how the RA Nutrition & Lifestyle App is structured. This document intentionally describes the system at a conceptual level and **does not** cover scoring logic or the internal rule engine.

---

## System Design

The application follows a conventional three-tier architecture with a clear separation between the client, the API, and persistent storage.

```
┌─────────────────────┐        HTTPS / JSON        ┌─────────────────────┐
│      Frontend       │ ─────────────────────────▶ │       Backend       │
│  React + Vite       │                            │      FastAPI        │
│  Tailwind CSS       │ ◀───────────────────────── │   (Python, REST)    │
│  React Query        │        JSON responses      │                     │
└─────────────────────┘                            └──────────┬──────────┘
                                                               │ SQL
                                                               ▼
                                                    ┌─────────────────────┐
                                                    │     PostgreSQL      │
                                                    │  (logical schemas)  │
                                                    └─────────────────────┘
```

### Frontend
A single-page application built with **React** and **Vite**, styled with **Tailwind CSS**. **React Query** manages server state — caching, background refetching, and synchronization with the API. The frontend communicates exclusively through the backend's REST API and holds no direct database access.

### Backend
A **FastAPI** service exposing a JSON REST API. It is responsible for authentication, request validation, business logic, and persistence. The backend is the single source of truth for all data operations and is the only tier permitted to talk to the database.

### Database
A **PostgreSQL** instance organized into logical schemas (see below). Schema changes are managed through versioned migrations rather than ad-hoc DDL.

---

## Database Schema Overview

Data is organized into four logical schemas, each with a distinct responsibility:

| Schema | Responsibility |
|--------|----------------|
| **core** | Foundational entities such as users and accounts. |
| **static** | Reference and lookup data that changes infrequently (e.g. the food library). |
| **tracking** | Time-series user activity such as meal logs and symptom entries. |
| **intelligence** | Derived data supporting recommendations and analytics. |

> Only schema names and responsibilities are described here. Table-level details, scoring fields, and rule-engine internals are intentionally omitted.

---

## API Structure

The API is organized into resource-oriented route groups, each backed by a service layer that encapsulates business logic. Endpoints follow REST conventions and exchange JSON.

```
┌──────────────────────────────────────────────┐
│                  FastAPI App                   │
├──────────────────────────────────────────────┤
│  Routers (HTTP layer)                          │
│    • Auth & users                              │
│    • Foods & meal library                      │
│    • Meal logging                              │
│    • Symptom tracking                          │
│    • Recommendations                           │
│    • Analytics                                 │
├──────────────────────────────────────────────┤
│  Services (business logic)                     │
├──────────────────────────────────────────────┤
│  Models / persistence (PostgreSQL)             │
└──────────────────────────────────────────────┘
```

- **Routers** handle HTTP concerns: request parsing, validation, and response shaping.
- **Services** contain the business logic and orchestrate persistence.
- **Models** map to the logical schemas described above.

This layering keeps the HTTP surface thin and the business logic testable and independent of transport details.
