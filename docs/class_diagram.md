```mermaid
classDiagram

%%% -------------------------------
%% Enumerations
%%% -------------------------------
class Role {
    <<enumeration>>
    COMMANDER
    VIEWER
    AUDITOR
}

class ConnectionState {
    <<enumeration>>
    CONNECTED
    RECONNECTING
    DEGRADED
    DISCONNECTED
}

%%% -------------------------------
%% Controllers (API Layer)
%%% -------------------------------
class RobotCommandController {
    +postMove(x:int, y:int, token:String) Response
    +postReset(token:String) Response
}

class TelemetryController {
    +getLiveTelemetry(token:String) Stream
    +getStatus(token:String) JSON
}

class AuditController {
    +getLogs(token:String) JSON
}

%%% -------------------------------
%% Services (Business Logic)
%%% -------------------------------
class AuthService {
    +verifyToken(token:String) User
    +authorize(user:User, action:String) bool
}

class RobotCommandService {
    +move(user:User, x:int, y:int) Result
    +reset(user:User) Result
    +confirmArrival(targetX:int, targetY:int, timeoutSec:int) bool
}

class TelemetryService {
    -ConnectionState state
    +connectWebSocket() void
    +fallbackPolling() void
    +getLatestStatus() RobotStatus
    +getConnectionState() ConnectionState
}

class MapService {
    +refreshMap() RobotMap
    +isObstacle(x:int, y:int) bool
    +inBounds(x:int, y:int) bool
}

%%% -------------------------------
%% Repositories (Data Access)
%%% -------------------------------
class UserRepository {
    +findByUsername(username:String) User
}

class MissionLogRepository {
    +save(log:MissionLog) void
    +findAll() MissionLog[]
}

%%% -------------------------------
%% External Clients (Robot Boundary)
%%% -------------------------------
class RobotApiClient {
    +getStatus() RobotStatus
    +move(x:int, y:int) ApiResponse
    +reset() ApiResponse
    +getMap() RobotMap
}

class RobotWsClient {
    +connect() void
    +onMessage(handler) void
    +reconnectWithBackoff() void
    +disconnect() void
}

%%% -------------------------------
%% Data Models (Entities)
%%% -------------------------------
class User {
    -String username
    -String passwordHash
    -Role role
}

class MissionLog {
    -DateTime timestamp
    -String username
    -int targetX
    -int targetY
    -String action
    -String result
    -String robotStatusBefore
}

class RobotStatus {
    -String id
    -int x
    -int y
    -int battery
    -String status
}

class RobotMap {
    -int width
    -int height
    -int[][] grid
}

%%% -------------------------------
%% Relationships
%%% -------------------------------

%% Controllers depend on AuthService for every request
RobotCommandController --> AuthService : authenticates via
TelemetryController --> AuthService : authenticates via
AuditController --> AuthService : authenticates via

%% Controllers delegate to their services
RobotCommandController --> RobotCommandService : delegates to
TelemetryController --> TelemetryService : delegates to
AuditController --> MissionLogRepository : queries

%% RobotCommandService orchestrates the move flow
RobotCommandService --> MapService : validates via
RobotCommandService --> RobotApiClient : dispatches to
RobotCommandService --> MissionLogRepository : logs to
RobotCommandService --> TelemetryService : reads status for arrival confirmation

%% TelemetryService manages both connection types
TelemetryService --> RobotWsClient : primary connection
TelemetryService --> RobotApiClient : fallback polling

%% MapService fetches map from robot
MapService --> RobotApiClient : fetches map from

%% Auth resolves users
AuthService --> UserRepository : looks up

%% Data ownership (composition)
User --> Role : has
TelemetryService --> ConnectionState : tracks
MissionLogRepository o-- MissionLog : stores
UserRepository o-- User : stores