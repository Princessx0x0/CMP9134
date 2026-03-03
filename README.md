# Robot Management System – CMP9134

## Overview

This project implements a web-based Ground Control Station for a Virtual Robot Simulation.  
The system enables authenticated users to monitor and control a simulated robot operating within a 2D grid environment.

The application demonstrates:

- Role-Based Access Control (RBAC)
- Real-time telemetry handling
- Map-aware navigation validation
- Audit logging
- Layered architecture (Controller → Service → Repository)
- Docker-based simulation integration
- Ethical and dual-use awareness

---

## System Architecture

The following component diagram illustrates the high-level system structure and interaction between the application and the simulation container.

```mermaid
flowchart LR

subgraph GCS["Ground Control Station (Docker Compose)"]
UI[Web UI]
API[Backend API]
DB[(Database)]
end

Robot["Virtual Robot Simulation Container"]

UI --> API
API --> DB
API --> Robot
Robot --> API
```

## Component Diagram 

The following component diagram illustrates the high-level system structure and interaction between the application and the simulation container.

```mermaid
flowchart TB

%% ===== THE USER (external actor) =====
User["User (Browser)"]

%% ===== YOUR SYSTEM — everything you build and deploy =====
subgraph GCS ["Ground Control Station (docker-compose)"]

    subgraph Frontend ["Frontend Container"]
        UI["Web Dashboard (React)"]
    end

    subgraph Backend ["Backend Container"]
        API["REST API Server"]
        AuthModule["Auth Service (JWT + RBAC)"]
        CmdModule["Command Dispatcher"]
        TelemModule["Telemetry Manager"]
    end

    subgraph Data ["Database"]
        DB[("SQLite / PostgreSQL")]
    end

end

%% ===== EXTERNAL DEPENDENCY — provided to you =====
subgraph Robot ["Virtual Robot Container (Docker)"]
    RobotAPI["Robot REST API"]
    RobotWS["WebSocket Server"]
end

%% ===== NETWORK CONNECTIONS =====

%% User to Frontend
User -->|"HTTPS"| UI

%% Frontend to Backend
UI -->|"HTTP/REST"| API

%% Backend internal flow
API --> AuthModule
API --> CmdModule
API --> TelemModule

%% Backend to Database
AuthModule -->|"SQL"| DB
CmdModule -->|"SQL (audit logs)"| DB

%% Backend to Robot — REST (commands + fallback polling)
CmdModule -->|"POST /api/move\nPOST /api/reset"| RobotAPI
CmdModule -->|"GET /api/status\n(arrival confirmation)"| RobotAPI

%% Backend to Robot — WebSocket (live telemetry)
TelemModule <-->|"ws://robot:5000/ws/telemetry\n(real-time)"| RobotWS

%% Backend to Robot — fallback when WebSocket drops
TelemModule -.->|"GET /api/status\n(fallback polling)"| RobotAPI
```