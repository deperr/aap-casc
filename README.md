# AAP Config-as-Code

Configuration as Code for Ansible Automation Platform 2.6 using the `infra.aap_configuration` collection.

## Repository Structure

```
├── setup.yml                        # Consolidated CaC playbook
├── controller/                      # Automation Controller definitions
│   ├── credential_types.yml
│   ├── credentials.yml
│   ├── execution_environments.yml
│   ├── inventories.yml
│   ├── inventory_groups.yml
│   ├── inventory_sources.yml
│   ├── job_templates.yml
│   ├── labels.yml
│   ├── projects.yml
│   └── workflow_job_templates.yml
├── hub/                             # Private Automation Hub definitions
│   ├── collection_remotes.yml
│   └── collection_repositories.yml
├── eda/                             # Event-Driven Ansible definitions
│   ├── credentials.yml
│   ├── projects.yml
│   └── rulebook_activations.yml
└── platform/                        # Platform-level definitions
    └── teams.yml
```

## Prerequisites

- Ansible Automation Platform 2.6
- `infra.aap_configuration` collection installed in the execution environment
- A **Red Hat Ansible Automation Platform** credential configured in AAP with access to the target platform components

## Setup in AAP

### 1. Create the Project

Create a new Project in Automation Controller pointing to this repository.

| Field | Value |
|---|---|
| **Name** | AAP Config-as-Code |
| **Source Control Type** | Git |
| **Source Control URL** | *(this repository's URL)* |

### 2. Create the Job Template

Create a Job Template using the consolidated `setup.yml` playbook.

| Field | Value |
|---|---|
| **Name** | AAP CaC - Setup |
| **Inventory** | localhost (or any inventory; the play targets localhost) |
| **Project** | AAP Config-as-Code |
| **Playbook** | `setup.yml` |
| **Credential** | *(your Red Hat Ansible Automation Platform credential)* |
| **Execution Environment** | *(an EE with `infra.aap_configuration` installed)* |

### 3. Configure the Survey

Add a survey to the Job Template with the following question:

| Field | Value |
|---|---|
| **Question** | Which component to configure? |
| **Answer Variable Name** | `casc_target` |
| **Answer Type** | Multiple Choice (single select) |
| **Multiple Choice Options** | `controller`, `hub`, `eda`, `platform`, `all` |
| **Default Answer** | `all` |
| **Required** | Yes |

### Survey Options

| Value | Description |
|---|---|
| `controller` | Configures Automation Controller resources (credentials, projects, templates, inventories, etc.) |
| `hub` | Configures Private Automation Hub resources (collection remotes and repositories) |
| `eda` | Configures Event-Driven Ansible resources (credentials, projects, rulebook activations) |
| `platform` | Configures platform-level resources (teams) |
| `all` | Configures all components sequentially |

## Adding Configuration

To add or modify resources, edit the YAML files under the appropriate component directory. Each file maps to a top-level variable consumed by the `infra.aap_configuration.dispatch` role:

| Directory | Variable Prefix | Example |
|---|---|---|
| `controller/` | `controller_*` | `controller_templates`, `controller_projects` |
| `hub/` | `hub_*` | `hub_collection_remotes` |
| `eda/` | `eda_*` | `eda_projects`, `eda_rulebook_activations` |
| `platform/` | `aap_*` | `aap_teams` |

Refer to the [infra.aap_configuration collection documentation](https://galaxy.ansible.com/ui/repo/published/infra/aap_configuration/) for the full list of supported variables and their schemas.

## Running Locally

```bash
ansible-playbook setup.yml -e casc_target=controller
```

Provide the required connection variables (`aap_hostname`, `aap_token`, `aap_validate_certs`) via extra vars, environment variables, or a local vars file.
