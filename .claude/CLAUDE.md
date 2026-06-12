# CLAUDE.md — AI Agent Instructions for aap-demo-projects

## Repository Identity

**Name**: aap-demo-projects (likely to be renamed to `aap-demo-platform-utilities`)
**Purpose**: Platform-level utilities for Ansible Automation Platform (AAP)
**Target Environment**: [aap-demo](https://github.com/RedHatOfficial/aap-demo) deployment
**Deployment Method**: Automated via Custom Resource (CR) in aap-demo
**Future Location**: Red Hat Community of Practice (CoP) organization

---

## Mission Statement

This repository provides **out-of-the-box platform utilities** for AAP administrators and demo presenters. These are operational tools for managing AAP itself, not application automation.

### Core Utilities (Priority Order)

1. **Export AAP Configuration as Code** (TOP PRIORITY)
   - Export all AAP resources OR organization-level OR specific resources
   - Generate ready-to-use `aap_config/` YAML structure
   - Support exporting to developer's local machine OR their Git repository
   - Organize by component (controller/, gateway/, hub/, eda/)

2. **Organization Onboarding**
   - Quick organization creation with survey inputs:
     - Organization name
     - Repository URL
     - SCM credentials (from vault)
   - Optionally pull SCM credentials from developer's environment at deployment time

3. **Red Hat Console Token Management**
   - Retrieve and manage tokens from console.redhat.com
   - Support for offline tokens and API access

4. **Other Platform Utilities** (Future)
   - Platform health checks
   - Backup/restore helpers
   - User/RBAC management
   - Integration setup assistance

---

## Repository Role in Ecosystem

This repo establishes the **template pattern** for the aap-demo ecosystem:

- **This Repo**: Platform utilities for AAP administration
- **Future Add-on Repos**: Customer demo scenarios (network, Windows, cloud, etc.)
- **Pattern**: All use same structure (`aap_config/`, `configure_aap.yml`, etc.)
- **Discovery**: Future GitHub Pages with project info and examples

### Template Repository

Reference structure and patterns from:
- <https://github.com/Megalith-Development/Ansible-Template-Repo>

---

## Deployment Model

### Automated CR Integration

- Deployed automatically via Custom Resource in aap-demo
- CR references this GitHub repo URL and a **release version branch**
- `configure_aap.yml` runs automatically after aap-demo is up
- Must be fully idempotent and handle failures gracefully

### Secrets Management

- Use `vault.yml` for secrets storage (current approach)
- Secrets include:
  - `vault_aap_hostname`
  - `vault_aap_token`
  - `vault_github_username`
  - `vault_github_token`
  - PAH/Red Hat Console credentials (for collection downloads)

### AAP Resources Created

The repo configures these AAP objects via `aap_config/`:

- **Organization**: "Utilities" (for platform admin tools)
- **Project**: Points to this repository
- **Credentials**: SCM credentials for pulling this repo
- **Job Templates**: One per utility playbook (defined in code)

---

## Repository Structure

```
.
├── .claude/                    # AI agent instructions (this file)
├── .github/workflows/          # CI/CD pipelines
│   └── validate.yml           # Primary validation workflow (ACTIVE)
├── aap_config/                # AAP configuration as code
│   ├── controller/            # Controller resources (projects, credentials, job templates)
│   ├── gateway/               # Gateway resources (organizations, auth)
│   ├── hub/                   # Hub resources (future)
│   └── eda/                   # EDA resources (future)
├── collections/
│   └── requirements.yml       # Galaxy collections (source of truth for dependencies)
├── execution-environment/     # EE definition
├── inventories/               # Inventory files
├── playbooks/                 # Executable playbooks (AAP entry points)
│   └── configure_aap.yml      # Bootstrap playbook for CR deployment
├── roles/                     # Reusable roles (business logic)
├── rulebooks/                 # EDA rulebooks
├── vault.yml                  # Ansible vault secrets
├── AGENTS.md                  # Development standards (AUTHORITATIVE)
└── README.md                  # Human-facing documentation
```

---

## Development Standards

### Authoritative References

All development MUST follow:

1. **AGENTS.md** (this repository) — Ansible development conventions
2. **Red Hat CoP Best Practices** — <https://redhat-cop.github.io/automation-good-practices/>

### Role Design Pattern

Roles are designed as **classes with public functions**:

```yaml
# Playbook: playbooks/export_config.yml
- name: Export AAP Configuration
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Export configuration
      ansible.builtin.import_role:
        name: aap_export
        tasks_from: export_all  # Public function
```

**Role Structure**:
```
roles/aap_export/
├── defaults/main.yml          # Variable initialization
├── tasks/
│   ├── main.yml              # Default orchestration
│   ├── validate.yml          # Input validation (REQUIRED)
│   ├── export_all.yml        # Public function
│   ├── export_org.yml        # Public function
│   └── export_resource.yml   # Public function
├── filter_plugins/           # Custom filters for complex data transforms
├── meta/
│   └── argument_specs.yml    # Optional schema validation
└── README.md                 # Role documentation with public functions listed
```

### Validation Requirements

Every role MUST include `tasks/validate.yml`:

- Validate required external variables (surveys, extra vars, inventory vars)
- Check formats and constraints
- Validate variable relationships
- Fail fast with clear error messages
- Use `ansible.builtin.assert` with meaningful messages

### Playbook Design

- **Thin playbooks**: Business logic stays in roles
- **Clear naming**: Describe the operation (e.g., `export_config_all.yml`)
- **One playbook per Job Template**: AAP executes playbooks, not roles
- **Use `import_role` with `tasks_from`**: Call specific role functions

### Idempotency

- MUST aim for idempotent tasks
- NOT an absolute rule—use judgment
- Document non-idempotent operations clearly

### Data Transformation

- **Jinja**: Maximum 4 chained filters; use `json_query` when appropriate
- **Complex Logic**: Create Python filter plugin in `filter_plugins/`
- Keep transformations readable and testable

### Command Usage

Avoid `shell`, `command`, `raw` modules:
- MUST be justified if used
- MUST include safeguards: `changed_when`, `failed_when`, `creates`, `removes`
- Explain why module-based approach wasn't used

---

## AAP Configuration as Code

### Structure

All AAP resources defined in `aap_config/`:

```
aap_config/
├── controller/
│   ├── projects.yml
│   ├── credentials.yml
│   ├── job_templates.yml
│   ├── inventories.yml
│   └── workflows.yml
├── gateway/
│   ├── auth.yml
│   └── organizations.yml
├── hub/
│   └── (future)
└── eda/
    └── (future)
```

### Dispatch Role Usage

Bootstrap playbook uses `infra.aap_configuration.dispatch`:

```yaml
- name: Apply AAP configuration
  ansible.builtin.include_role:
    name: infra.aap_configuration.dispatch
  vars:
    aap_configuration_collect_logs: true
```

This automatically processes all YAML files in `aap_config/`.

### Resource Naming Conventions

Follow Red Hat CoP standards:
- Use descriptive names
- Include purpose/function
- Organization prefix where applicable
- Example: `AAP Demo Projects - Utilities`

---

## Branching Strategy

- **main**: Stable, production-ready, used by CRs
- **release branches**: Tagged versions for CR pinning (e.g., `release/v1.0`)
- **stage**: Stable but for testing before release
- **devel**: Development branch (current)
- **feature/***: Feature development branches

### Release Process

1. Develop on `devel` or `feature/*`
2. Merge to `stage` for stability testing
3. Merge to `main` when verified
4. Tag releases for CR version pinning

---

## Testing & CI/CD

### Active CI

- `.github/workflows/validate.yml` — PRIMARY validation workflow
- Other workflows present but can be disabled until needed

### Testing Requirements

Before release:
- Syntax validation: `ansible-playbook --syntax-check`
- Lint: `ansible-lint` and `yamllint` (when applicable)
- Check mode: `ansible-playbook --check` (where supported)
- Functional testing in stage environment

### Pre-commit Checks

Not enforced but encouraged:
- YAML linting
- Ansible syntax checks
- Collection dependency validation

---

## Dependencies

### Collections (Source of Truth)

See `collections/requirements.yml`:

```yaml
collections:
  - name: infra.aap_configuration_extended
  - name: ansible.controller
  - name: ansible.hub
  - name: ansible.platform
  - name: ansible.eda
  - name: infra.aap_configuration
```

**Important**:
- This repo requires PAH/Red Hat Console access
- Credentials stored in `vault.yml`
- Must be available in aap-demo deployment secrets

### Execution Environment

- Defined in `execution-environment/`
- Must include all required collections
- Built via GitHub Actions workflow

---

## Export Config-as-Code Utility (Priority #1)

### Requirements

**Functionality**:
1. Export all AAP resources (complete dump)
2. Export organization-level resources only
3. Export specific resource types (credentials, job templates, etc.)

**Output Structure**:
```
aap_config/
├── controller/
│   ├── credentials.yml
│   ├── job_templates.yml
│   ├── projects.yml
│   ├── inventories.yml
│   └── workflows.yml
├── gateway/
│   ├── organizations.yml
│   └── auth.yml
├── hub/
│   └── (as applicable)
└── eda/
    └── (as applicable)
```

**Export Targets**:
- Local filesystem (developer's machine)
- Git repository (automated commit/push)

**Design**:
- Role: `roles/aap_export/`
- Playbooks:
  - `playbooks/export_config_all.yml`
  - `playbooks/export_config_org.yml`
  - `playbooks/export_config_resource.yml`
- Survey parameters:
  - Export scope (all/org/resource)
  - Organization name (if org scope)
  - Resource type (if resource scope)
  - Output destination (local/git)
  - Git repo URL (if git destination)

---

## Organization Onboarding Utility

### Requirements

**Survey Inputs**:
- Organization name (required, validated)
- Repository URL (required, validated URL format)
- SCM credential source (vault or environment variable)

**Functionality**:
- Create organization in AAP
- Create project pointing to repository
- Attach SCM credentials from vault
- Optionally pull credentials from developer environment at deployment time

**Design**:
- Role: `roles/org_onboarding/`
- Playbook: `playbooks/onboard_organization.yml`
- Use `infra.aap_configuration` collections
- Generate `aap_config/` structure dynamically based on survey input

---

## Documentation Requirements

### Role README

Each role MUST have a README documenting:
- Purpose
- Public functions (tasks_from options)
- Required variables
- Optional variables
- Examples

### Repository README

Update when:
- Adding new utilities
- Changing integration patterns
- Adding AAP Job Template entry points

DO NOT automatically rewrite—suggest updates only.

---

## Agent Decision Making

### When Multiple Approaches Exist

The agent MUST:
1. Present options clearly
2. Explain trade-offs
3. Ask the user for direction

### When Deviating from AGENTS.md

The agent MUST:
1. Explain the deviation
2. Justify the reasoning
3. Request user confirmation

### When Uncertain About Collections

If unsure whether an appropriate collection exists:
- MUST ask the user how to proceed
- Prefer supported collections over raw API calls

---

## Agnostic Design Philosophy

**Goal**: Utilities should work in any AAP environment while being optimized for aap-demo.

**Approach**:
- Avoid hardcoding aap-demo-specific values
- Use variables for hostnames, organizations, etc.
- Provide sensible defaults for aap-demo context
- Document standalone usage in READMEs

**Benefit**: Utilities become reusable by broader community.

---

## Future Considerations

### GitHub Pages Documentation

Future development includes:
- Project catalog and discovery
- Usage examples and walkthroughs
- Integration guides for aap-demo

### Additional Demo Repos

This repo establishes the pattern for:
- Network automation demos
- Windows management demos
- Cloud provisioning demos
- Application deployment demos

Each will follow the same structure and integration model.

---

## Summary for AI Agents

When working in this repository:

1. **Read AGENTS.md first** — it's authoritative for Ansible standards
2. **Follow the role-as-class pattern** — playbooks call role functions
3. **Validate all inputs** — every role needs `tasks/validate.yml`
4. **Export config is priority #1** — focus here for maximum impact
5. **Keep it agnostic** — works in aap-demo but not limited to it
6. **Use infra.aap_configuration** — leverage the CoP collection
7. **Ask when uncertain** — don't guess at critical decisions
8. **Test before release** — maintain stability for CR deployments
9. **Document public functions** — role READMEs are essential
10. **Think ecosystem** — this is a template for future demo repos

---

## Getting Help

- **Standards Questions**: See AGENTS.md
- **AAP Config Pattern**: Use `/rh-aap-config-as-code` skill
- **Community Standards**: <https://redhat-cop.github.io/automation-good-practices/>
- **Template Reference**: <https://github.com/Megalith-Development/Ansible-Template-Repo>
