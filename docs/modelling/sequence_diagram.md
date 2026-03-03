```mermaid
sequenceDiagram
    actor C as Commander
    participant UI as Web Dashboard
    participant RC as RobotCommandController
    participant Auth as AuthService
    participant Svc as RobotCommandService
    participant Map as MapService
    participant Sim as Virtual Robot (Docker)
    participant DB as MissionLogRepository

    C->>UI: Enter X,Y and click "Move"
    UI->>RC: POST /move {x,y} + Auth Token
    activate RC

    %% Authentication + Authorisation
    RC->>Auth: verifyToken(token)
    Auth-->>RC: user(role=Commander)

    RC->>Auth: authorize(user, "MOVE")
    Auth-->>RC: allowed

    %% Delegate to service layer
    RC->>Svc: move(user, x, y)
    activate Svc

    %% Validation chain
    Svc->>Map: inBounds(x,y)?
    Map-->>Svc: yes

    Svc->>Map: isObstacle(x,y)?
    Map-->>Svc: no

    %% Robot state check
    Svc->>Sim: GET /api/status
    Sim-->>Svc: {status: "IDLE"}

    %% Dispatch command
    Svc->>Sim: POST /api/move {x,y}
    Sim-->>Svc: 200 OK (accepted)

    %% Audit log
    Svc->>DB: save MissionLog(user, x, y, result="ACCEPTED")
    DB-->>Svc: saved

    Svc-->>RC: success response
    deactivate Svc

    RC-->>UI: 200 OK + "Command accepted"
    deactivate RC

    UI->>C: Display "Moving to (x,y)..."

    %% Arrival confirmation (async phase)
    Note over Svc,Sim: Async arrival confirmation begins

    loop Poll every 1s until arrival or 60s timeout
        Svc->>Sim: GET /api/status
        Sim-->>Svc: {position: {x,y}, status: "..."}
    end

    alt Arrival confirmed (position matches + IDLE)
        Svc->>DB: update MissionLog(arrivalConfirmed=true)
        Svc-->>UI: Arrival confirmed
        UI->>C: Display "Robot arrived at (x,y)"
    else Timeout (60s elapsed)
        Svc->>DB: save MissionLog(result="TIMEOUT")
        Svc-->>UI: Timeout warning
        UI->>C: Display "Arrival not confirmed"
    end

    Note over UI,Sim: Telemetry updates continue via WebSocket independently