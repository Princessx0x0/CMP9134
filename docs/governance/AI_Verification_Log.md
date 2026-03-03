# AI Verification Log

## Entry 1 – Stakeholder Safety Review (Task 4)

**Date:** 22/02/2026  
**Task Category:** Requirements Validation  
**AI Tool Used:** ChatGPT  
**Persona Used:** Chief Safety Officer  

---

### 1. Prompt Used
Act as the **Chief Safety Officer** for a warehouse deploying a new Virtual Robot Management System.

### Context

- The robot is controlled via a dashboard.
- Commands are sent using `POST /api/move` with JSON body `{ "x": int, "y": int }`.
- Telemetry is read from `GET /api/status` returning:
  - `id`
  - `position {x, y}`
  - `battery (0–100)`
  - `status` (`IDLE`, `MOVING`, `LOW_BATTERY`, `STUCK`)
- The environment map comes from `GET /api/map`:
  - `width` and `height` (21x21)
  - `grid` where `0 = free space` and `1 = obstacle`
- The system must handle network latency and dropouts gracefully.

---

I am the lead software engineer. Please review the following User Story and Acceptance Criteria from your perspective and tell me:

1. What safety, security, or usability edge cases are missing?
2. Is the language clear enough for a non-technical stakeholder to approve?
3. Propose **five improvements** to the acceptance criteria that are testable with clear pass/fail outcomes.


---

### 2. Summary of AI Feedback

The AI identified several safety and usability gaps in the move command requirements, including:

- No obstacle validation before issuing movement commands.
- No confirmation that the robot successfully arrives at the target coordinate.
- No timeout handling if the robot fails to reach the destination.
- Lack of in-transit command protection.
- Missing stakeholder-friendly wording.

---

### 3. Modifications Implemented

The following acceptance criteria were added to the Trello board:

- **Obstacle Rejection:** If the target cell in `/api/map` grid equals `1`, the dashboard blocks the move and displays:  
  _"Target is blocked — select a free cell."_  
  No API request is sent.

- **Arrival Confirmation:** After a successful move command, the dashboard polls `/api/status` every ≤2 seconds until:
  - The robot position matches the target coordinates, and
  - The robot status returns to `IDLE`.

- **Timeout Handling:** If the robot does not reach the destination within 60 seconds, the dashboard displays:  
  _"Move timed out — check robot status."_

---

### 4. Validation Performed

- Tested movement to obstacle coordinates and confirmed the dashboard blocks the request.
- Tested movement to a valid free cell and confirmed arrival message appears only when position matches the target.
- Simulated connection interruption and verified timeout handling logic activates.

---

### 5. Reflection

The AI feedback helped shift the move command from a simple API interaction to a state-aware control process.  
This improved the safety, usability, and operational clarity of the system requirements.