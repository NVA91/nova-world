# AI Coding Agent Instructions for Nova World Project

## Project Overview
This is a unified Ansible project for deploying self-hosted cloud infrastructure on Proxmox VE, combining local Docker-based testing with production deployment. The architecture follows "Clean House Strategy" with clear separation between hypervisor, network, and applications.

## Key Architecture Components
- **controller/**: Docker-based test environment for isolated, reproducible playbook validation
- **infrastructure/**: Production deployment logic with profile-based configuration
- **shared/**: Common resources (roles, templates, plugins) referenced by both environments
- **VM Structure**: vm-gateway (WireGuard + Traefik), vm-office (data apps), vm-ai-lab (AI apps)

## Critical Workflows
- **Local Testing**: `make build` → `make test` (runs ansible-lint + syntax-check in container)
- **Production Deploy**: `make deploy` variants (standard/full/minimal/custom/repair) using `ansible-playbook site.yml -e "@config/profile_*.yml"`
- **Interactive Custom**: Prompts user for VMs/apps, sets facts like `custom_vms: "{{ vm_input.user_input | split(',') }}"`

## Project-Specific Patterns
- **Profile System**: Use `deployment_profiles[wiz_profile]` dict for VM/app lists (see `infrastructure/config/profile_*.yml`)
- **Role Loops**: Iterate apps with `loop: "{{ apps_config | dict2items }}"` + `when: item.key in apps_to_deploy`
- **Docker Compose Deployment**: Templates in `shared/templates/docker-compose/` copied to `/opt/docker-compose/{{ item.key }}/`
- **Conditional Blocks**: Use `block:` with `when:` for profile-specific tasks (e.g., custom interactive prompts)
- **Host Addition**: `add_host` for WIZARD_CONFIG to pass profile data between plays

## Integration Points
- **Proxmox API**: Via `provision_guests` role for VM creation from cloud images
- **Docker Setup**: `docker_setup` role installs Docker on VMs, `app_deployment` runs compose files
- **External Dependencies**: Apps like Traefik, Paperless-ngx, N8N, PostgreSQL, Whisper, Ollama via compose templates

## Examples
- **Custom Profile Logic**: In `site.yml` Play 1, use `pause` to register user input, then `set_fact` with `split(',')` for lists
- **App Deployment**: `shared/roles/app_deployment/tasks/main.yml` shows template copying with conditional loops
- **Makefile Targets**: `deploy-custom` runs playbook with profile_custom.yml for interactive mode

Reference: `docs/ARCHITECTURE.md`, `README.md`, `infrastructure/site.yml` for full context.