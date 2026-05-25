# Answer Frameworks
This document defines the reusable frameworks used across the playbook.
The goal is to make technical answers clear, structured, and evidence-driven before jumping into commands or one possible root cause.

---

## 1. SCOPE - Troubleshooting Framework
Use this framework for troubleshooting scenarios.

### S - Scope the issue
Clarify the impact before investigating.
Questions to ask:

- Is this affecting one user, one device, one subnet, one site, or everyone?
- When did the issue start?
- Is the issue constant or intermittent?
- Did anything change recently?
- What is the business or user impact?

### C - Confirm expected behavior
Define what should happen.

Questions to ask:

- What service or destination should be reachable?
- From where?
- Using what hostname, IP address, protocol, or port?
- What does successful access look like?

### O - Observe evidence
Collect facts before deciding the root cause.

Possible checks:

- Client IP configuration
- DNS resolution
- Gateway and routing path
- Port reachability
- Firewall or security policy
- Server/service status
- Logs or packet capture

### P - Prioritize hypotheses
Group possible causes by area.

Examples:

- Client-side issue
- DNS issue
- Routing issue
- Firewall/security issue
- Server/service issue
- Provider/external issue

### E - Execute and verify
Make one change at a time and verify using the original failure condition.

Verification should confirm:

- The issue is resolved
- The expected service works
- No unnecessary access was opened
- The root cause and prevention steps are documented

---

## 2. RAMP - Unfamiliar Technology Framework
Use this framework when working with an unfamiliar device, platform, or vendor.

### R - Role and risk
Identify what the system does before touching configuration.

Questions to ask:

- What is the device or system role?
- Is it production, test, or lab?
- What services depend on it?
- What is the risk of making changes?

### A - Access and current state
Collect read-only information first.

Examples:

- Hostname
- Model/version
- Uptime
- Interface status
- Current configuration
- Logs
- Neighbor information
- Backup or rollback option

### M - Map to known concepts
Translate the unfamiliar platform into familiar infrastructure concepts.

Examples:

- Interfaces
- VLANs
- Trunks
- Routing
- ACLs or policies
- Logs
- Services
- Configuration save behavior

### P - Proceed safely
Use low-risk steps first.

Principles:

- Prefer read-only commands first
- Confirm documentation before making changes
- Make one change at a time
- Prepare rollback
- Validate after each step

---

## 3. LAYER - Plain-Language Explanation Framework
Use this framework to explain technical concepts clearly.

### L - Label the concept
Start with a simple one-sentence definition.

### A - Analogy
Use a simple analogy if it helps, but do not depend only on the analogy.

### Y - Why it exists
Explain the purpose of the technology.

### E - Example flow
Walk through a simple example.

### R - Real-world relevance
Connect the concept to troubleshooting, operations, or design.

---

## 4. STAR-R - Operations Reflection Framework
Use this framework for experience-based answers and retrospectives.

### S - Situation
What was the context?

### T - Task
What responsibility did I have?

### A - Action
What did I do?

### R - Result
What changed because of the action?

### R - Reflection

What did I learn, improve, document, automate, or prevent next time?
