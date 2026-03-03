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