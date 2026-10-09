# Pimo AI — Locked Role & Engineering Agent Architecture

**Status:** LOCKED  
**Decision date:** 2026-10-09

## 1. Role hierarchy

Pimo uses the following application-level role hierarchy:

1. **SUPER_ADMIN**
2. **ADMIN**
3. **MANAGER**
4. **EMPLOYEE**
5. **READ_ONLY / AUDITOR**

Authorization remains default-deny. Role membership alone does not bypass domain, object, action, sensitivity, or approval controls.

## 2. SUPER_ADMIN

SUPER_ADMIN is the platform-owner/developer role. Initially this role is assigned only to the system owner responsible for continuous development, maintenance, upgrades, security, and platform architecture.

SUPER_ADMIN may access:
- global user and role administration
- agent creation, configuration, enable/disable controls
- infrastructure and platform configuration
- deployments and upgrades
- PostgreSQL, Redis, Qdrant, n8n and service-level maintenance controls
- security/RBAC architecture
- secrets and privileged recovery procedures
- global audit and diagnostic views
- the private **Pimo Engineering Agent**

Only an existing SUPER_ADMIN may grant or revoke SUPER_ADMIN. An ADMIN may never promote themselves or another user to SUPER_ADMIN.

## 3. ADMIN

ADMIN is the highest normal operational role and is intentionally below SUPER_ADMIN.

ADMIN may manage, within policy:
- users
- normal role assignments excluding SUPER_ADMIN
- department and object access
- business configuration
- operational workflows
- approvals
- business-facing agent availability
- operational dashboards
- audit-log viewing where permitted

ADMIN does **not** receive:
- Pimo Engineering Agent access
- root/server access
- source-code modification authority
- infrastructure secrets
- unrestricted database administration
- production deployment authority
- permission to alter the SUPER_ADMIN role

## 4. MANAGER

MANAGER receives business-domain and object-scoped permissions according to assigned responsibilities. Managers can supervise workflows, approvals, teams, properties, cases, or departments within their authorized scope but cannot administer the Pimo platform itself.

## 5. EMPLOYEE

EMPLOYEE is the standard staff role. Access is limited to approved business agents, tools, objects, documents, and actions required for assigned work.

## 6. READ_ONLY / AUDITOR

READ_ONLY / AUDITOR provides non-mutating access to approved information, reports, and audit data. This role cannot execute business actions or alter system state.

## 7. Pimo Engineering Agent

A dedicated internal specialist agent named **Pimo Engineering Agent** is added outside the normal business-agent hierarchy.

**Access rule:** `SUPER_ADMIN only`.

Normal users and ADMIN users must not see or invoke this agent.

### Responsibilities

The Pimo Engineering Agent may:
- inspect application and infrastructure logs
- diagnose failed workflows and services
- inspect Docker/service health
- diagnose PostgreSQL, Redis, Qdrant and n8n issues
- inspect Pimo source code and configuration
- generate code patches and migrations
- run tests and validation checks
- prepare deployments and upgrades
- re-run approved failed jobs
- perform approved maintenance actions
- monitor platform health
- assist with continuous Pimo development and technical maintenance

## 8. Engineering action levels

The Engineering Agent follows a risk-tiered execution model.

### Level E0 — Observe
May execute without approval:
- read logs
- inspect service/container state
- inspect monitoring data
- query approved diagnostic metadata
- inspect code/configuration

### Level E1 — Diagnose/Test
May execute without separate approval where sandboxed/non-destructive:
- run health checks
- run tests
- perform read-only database diagnostics
- reproduce errors
- validate candidate fixes

### Level E2 — Prepare Change
May prepare but not deploy sensitive changes:
- generate patch
- prepare migration
- modify staging/worktree copies
- prepare configuration changes
- produce deployment plan

### Level E3 — Low-risk repair
May be eligible for pre-approved automatic execution after controls are implemented:
- restart an unhealthy non-critical service
- re-run a failed idempotent job
- clear explicitly safe temporary/cache state
- execute approved maintenance scripts

Every E3 action must be auditable and constrained by allowlists.

### Level E4 — Privileged/high-risk change
Requires explicit SUPER_ADMIN approval:
- production code deployment
- database schema migration
- destructive data operation
- security/RBAC change
- secret modification
- network/public exposure change
- user/role privilege escalation
- infrastructure configuration change
- deletion of production resources

## 9. Approval pattern

Preferred privileged-change flow:

1. Engineering Agent detects or receives an issue.
2. Agent diagnoses the root cause.
3. Agent prepares a fix.
4. Automated tests/health checks run.
5. Agent presents change, impact and rollback plan to SUPER_ADMIN.
6. SUPER_ADMIN explicitly approves or rejects.
7. Agent executes only the approved change.
8. Post-change health validation runs.
9. Complete audit event is recorded.

## 10. Audit identity

Engineering actions must have their own audit identity, separate from the human user.

Recommended fields include:

- `actor_type = AGENT`
- `actor_id = pimo_engineer`
- `authorized_by = <super_admin_user_id>` when human approval is required
- action/risk level
- target resource
- approval record
- execution result
- rollback/result metadata

Raw private chain-of-thought is never stored in audit logs; audits capture decisions, actions, inputs necessary for traceability, outputs/status, and authorization evidence.

## 11. Agent visibility model

Business users continue to access normal Pimo business agents through the orchestrator. The Engineering Agent is isolated behind the SUPER_ADMIN policy boundary.

```text
Normal user
  -> Identity/RBAC
  -> Pimo Orchestrator
  -> Business agents

SUPER_ADMIN
  -> Identity/RBAC
  -> Pimo Orchestrator / Admin Portal
  -> Pimo Engineering Agent
  -> controlled engineering tools
```

## 12. Locked implementation rule

This role hierarchy and Engineering Agent access model are now the production baseline. Changes to this architecture require an explicit future architecture decision rather than ad-hoc permission changes.
