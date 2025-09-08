I have created 
docker-compose.yml (no secrets inline).
.env.example (what keys are required — this file should be committed but contain no secrets).
Ansible playbook deploy-3tier.yml.
Template env.j2 used by Ansible to render the .env on target servers.
Notes: how to create/handle vault, run playbook, alternatives (Docker secrets / Swarm / Kubernetes).
Key point: all secrets and credentials are referenced via environment variables (${VAR}) — they must come from an external .env populated at deploy time (not checked into the repo).

3) Ansible playbook: deploy-3tier.yml
This playbook:
Installs Docker and the docker compose plugin on Debian/Ubuntu (adjust package tasks for RHEL/CentOS if needed).
Creates the project dir and uploads docker-compose.yml from your control node (you can also template it, but we keep compose static).
Renders .env on the server from an Ansible template (env.j2) using values stored in an Ansible Vault.
Runs docker compose up -d to start services.
Notes:
The playbook uses the community.docker.docker_compose Ansible module (install ansible-galaxy collection install community.docker on control node) — or you can replace with a command: docker compose up -d if you prefer.
Adjust installation tasks if you target CentOS/RHEL (use yum and add repo accordingly).

7) How to run (control machine)

Ensure you have Ansible and community.docker collection:

pip install ansible
ansible-galaxy collection install community.docker


Create the vault file:

ansible-vault create vault.yml
# Paste the YAML for secrets and save; you'll be prompted for a vault password


Place docker-compose.yml under files/docker-compose.yml (we copy it to the server using the playbook copy task).

Run playbook:

ansible-playbook -i inventory deploy-3tier.yml --ask-vault-pass


Or if you store vault password in a file:

ansible-playbook -i inventory deploy-3tier.yml --vault-password-file ~/.vault_pass
