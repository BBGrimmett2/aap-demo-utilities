# aap_export

Export Ansible Automation Platform configuration as code using the `infra.aap_configuration_extended.filetree_create` role.

## Description

This role provides a standardized way to export AAP configuration into YAML files suitable for version control and configuration-as-code workflows. It supports exporting all resources, organization-specific resources, or specific resource types.

## Requirements

- Ansible Automation Platform 2.5+
- Collections:
  - `infra.aap_configuration_extended`
  - `ansible.controller`
  - `ansible.platform`

## Public Functions

The role exposes three public functions via `tasks_from`:

### 1. `export_all`

Export all AAP configuration across all organizations.

**Example:**
```yaml
- name: Export all configuration
  ansible.builtin.import_role:
    name: aap_export
    tasks_from: export_all
```

### 2. `export_organization`

Export configuration for a specific organization.

**Required Variables:**
- `aap_export_organization`: Name of the organization to export

**Example:**
```yaml
- name: Export organization configuration
  ansible.builtin.import_role:
    name: aap_export
    tasks_from: export_organization
  vars:
    aap_export_scope: "organization"
    aap_export_organization: "Utilities"
```

### 3. `export_resource`

Export a specific resource type across all organizations.

**Required Variables:**
- `aap_export_resource_type`: Type of resource to export

**Valid Resource Types:**
- `credentials`
- `job_templates`
- `projects`
- `inventories`
- `workflows`
- `schedules`
- `execution_environments`
- `inventory_sources`
- `teams`
- `users`
- `organizations`

**Example:**
```yaml
- name: Export job templates
  ansible.builtin.import_role:
    name: aap_export
    tasks_from: export_resource
  vars:
    aap_export_scope: "resource"
    aap_export_resource_type: "job_templates"
```

## Role Variables

### Required Variables

These must be defined (typically in vault):

- `aap_hostname`: AAP hostname (from `vault_aap_hostname`)
- `aap_token`: AAP API token (from `vault_aap_token`)

### Core Variables

- `aap_export_scope`: Export scope - `all`, `organization`, or `resource` (default: `all`)
- `aap_export_destination`: Output destination - `local` or `git` (default: `local`)
- `aap_export_output_path`: Local output directory (default: `{{ playbook_dir }}/../aap_config`)

### Organization Export Variables

- `aap_export_organization`: Organization name (required when scope is `organization`)

### Resource Export Variables

- `aap_export_resource_type`: Resource type to export (required when scope is `resource`)

### Git Export Variables

- `aap_export_git_repo`: Git repository URL (required when destination is `git`)
- `aap_export_git_branch`: Git branch to commit to (default: `main`)

### Output Formatting Variables

- `aap_export_flatten`: Flatten output structure (default: `false`)
  - `false`: Organized by organization/type
  - `true`: All objects in single directory

### Security Variables

- `aap_export_secrets_as_variables`: Export secrets as Ansible variables (default: `true`)
- `aap_export_secrets_prefix`: Prefix for secret variables (default: `vaulted`)

### Export Behavior Variables

- `aap_export_related_objects`: Export related/dependent objects (default: `true`)
- `aap_export_skip_inventory_sources`: Skip inventory sources (default: `false`)
- `aap_export_skip_inventory_hosts`: Skip inventory hosts (default: `false`)
- `aap_export_skip_inventory_groups`: Skip inventory groups (default: `false`)

### API Configuration

- `aap_export_api_max_objects`: Maximum objects per API query (default: `10000`)
- `aap_validate_certs`: Validate SSL certificates (default: `true`)

## Output Structure

### Non-Flattened (default)

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
│   └── (hub resources)
└── eda/
    └── (eda resources)
```

### Flattened

```
aap_config/
├── all_credentials.yml
├── all_job_templates.yml
├── all_projects.yml
└── ...
```

## Usage Examples

### Export Everything Locally

```yaml
- name: Export all AAP configuration
  hosts: localhost
  gather_facts: true

  tasks:
    - name: Export all configuration
      ansible.builtin.import_role:
        name: aap_export
        tasks_from: export_all
```

### Export Organization to Git

```yaml
- name: Export organization to git
  hosts: localhost
  gather_facts: true

  tasks:
    - name: Export organization configuration
      ansible.builtin.import_role:
        name: aap_export
        tasks_from: export_organization
      vars:
        aap_export_scope: "organization"
        aap_export_organization: "Production"
        aap_export_destination: "git"
        aap_export_git_repo: "https://github.com/myorg/aap-config.git"
        aap_export_git_branch: "main"
```

### Export Specific Resource Type

```yaml
- name: Export credentials only
  hosts: localhost
  gather_facts: true

  tasks:
    - name: Export credentials
      ansible.builtin.import_role:
        name: aap_export
        tasks_from: export_resource
      vars:
        aap_export_scope: "resource"
        aap_export_resource_type: "credentials"
        aap_export_output_path: "/tmp/credentials_backup"
```

## AAP Job Template Integration

Use the corresponding playbooks for AAP Job Templates:

- `playbooks/export_config_all.yml` - Export all configuration
- `playbooks/export_config_org.yml` - Export organization configuration
- `playbooks/export_config_resource.yml` - Export resource type

Configure surveys in AAP to collect:
- Organization name (for org export)
- Resource type (for resource export)
- Destination (local/git)
- Git repository URL (if git destination)

## Dependencies

This role depends on:
- `infra.aap_configuration_extended.filetree_create`

The dependency is automatically loaded via `meta/main.yml`.

## License

GPL-3.0-or-later

## Author Information

AAP Demo Team - Red Hat
