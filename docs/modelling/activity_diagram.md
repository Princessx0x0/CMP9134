```mermaid
stateDiagram-v2

[*] --> ReceiveRequest

%% Phase 1: Authentication
ReceiveRequest --> ValidateToken

state ValidateToken <<choice>>
ValidateToken --> CheckRole : Token valid
ValidateToken --> LogRejection : Token invalid/expired

%% Phase 2: Authorisation (RBAC)
state CheckRole <<choice>>
CheckRole --> ValidateBounds : Role = Commander
CheckRole --> LogRejection : Role = Viewer/Auditor

%% Phase 3: Coordinate validation
state ValidateBounds <<choice>>
ValidateBounds --> CheckObstacle : Within 21x21 grid
ValidateBounds --> LogRejection : Out of bounds

%% Phase 4: Obstacle validation
state CheckObstacle <<choice>>
CheckObstacle --> CheckRobotState : Cell = Free
CheckObstacle --> LogRejection : Cell = Obstacle

%% Phase 5: Robot state check (FSM gating)
state CheckRobotState <<choice>>
CheckRobotState --> SendToRobot : Status = IDLE
CheckRobotState --> LogRejection : Status = MOVING/STUCK/LOW_BATTERY

%% Phase 6: API dispatch
SendToRobot --> CheckAPI

state CheckAPI <<choice>>
CheckAPI --> LogCommandAccepted : 200 OK received
CheckAPI --> LogError : Timeout/No response

%% Phase 7: Arrival confirmation
LogCommandAccepted --> AwaitArrival

state AwaitArrival <<choice>>
AwaitArrival --> LogSuccess : Position matches + IDLE (within 60s)
AwaitArrival --> LogTimeout : 60s elapsed without confirmation

%% All paths terminate via audit trail
LogRejection --> [*]
LogError --> [*]
LogSuccess --> [*]
LogTimeout --> [*]