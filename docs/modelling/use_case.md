```mermaid
flowchart TB

%% Actors 
C[Commander]
V[Viewer]
A[Auditor]

subgraph GCS ["Ground Control Station"]
    direction LR
    Login((Authenticate))
    Move((Move Robot))
    Reset((Reset Robot))
    Telem((View Live Telemetry))
    Map((View Grid Map))
    Audit((View Audit Trail))
end


C --> Login
C --> Move
C --> Reset
C --> Telem
C --> Map
C --> Audit

V --> Login
V --> Telem
V --> Map
V --> Audit

A --> Login
A --> Telem
A --> Audit