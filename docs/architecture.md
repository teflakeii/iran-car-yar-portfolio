# IranCarYar Architecture

## System Overview

```mermaid
flowchart TB
  subgraph Clients
    C["Customer Web / PWA"]
    R["Responder Web / PWA"]
    A["Admin Panel"]
    M["Android Shells"]
  end

  subgraph Backend
    API["FastAPI REST API"]
    Auth["JWT / OTP / Role Guards"]
    Domain["Service Request Domain"]
    Realtime["WebSocket Connection Manager"]
    Payment["Payment Flow"]
    SMS["SMS Provider Adapter"]
  end

  subgraph Data
    Models["SQLAlchemy Models"]
    Migrations["Alembic Migrations"]
    Store["SQLite Local / PostgreSQL Target"]
  end

  C --> API
  R --> API
  A --> API
  M --> C
  M --> R
  API --> Auth
  API --> Domain
  API --> Payment
  API --> SMS
  Domain --> Models
  Models --> Store
  Migrations --> Store
  Domain --> Realtime
  Realtime --> C
  Realtime --> R
  Realtime --> A
```

## Request Lifecycle

```mermaid
sequenceDiagram
  participant Customer
  participant API as FastAPI
  participant DB as Database
  participant Admin
  participant Responder
  participant WS as WebSocket

  Customer->>API: Create service request
  API->>DB: Persist pending request
  API->>WS: Publish request created
  WS-->>Admin: Notify dashboard
  Admin->>API: Assign responder
  API->>DB: Update assignment
  API->>WS: Publish assigned event
  WS-->>Responder: Notify active request
  Responder->>API: Start and complete work
  API->>DB: Update lifecycle state
  API->>WS: Publish status updates
  WS-->>Customer: Track request status
```

## Deployment Topology

```mermaid
flowchart LR
  Browser["Browser / PWA"] --> Nginx["Nginx Static Frontend"]
  Nginx --> API["FastAPI Backend"]
  API --> Postgres["PostgreSQL"]
  API --> Provider["External SMS / Payment Providers"]
```

## Design Notes

- REST endpoints carry customer, responder, and admin workflows.
- WebSocket events keep lifecycle updates synchronized.
- Role guards separate customer, responder, and admin access.
- Alembic owns schema evolution in the private source repository.
- Docker Compose is used as a validation baseline, not as a complete production platform.

