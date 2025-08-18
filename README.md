# Ansible Role Development & Testing Setup

This guide helps you set up a development environment for testing Ansible roles using **Molecule** with **Docker** on a system using the **Fish shell**.

---

## 1. Prerequisites

Make sure you have the following installed:

- Python 3.11+
- Docker & Docker Compose
- Fish shell (`fish`)
- Git

Check versions:

```fish
python3 --version
docker --version
docker-compose --version
fish --version
git --version





2. Create a Python Virtual Environment

It's recommended to use a Python virtual environment to isolate dependencies:

# Create a venv
python3 -m venv ~/ansible-molecule-venv

# Activate the venv (fish shell)
source ~/ansible-molecule-venv/bin/activate.fish

# Upgrade pip
pip install --upgrade pip


To deactivate the virtual environment:

deactivate

3. Install Molecule & Docker Driver

With the venv activated:

pip install molecule molecule-docker docker


This will allow you to test Ansible roles using Docker containers as test instances.

4. Install Ansible

If Ansible is not installed:

pip install ansible


Check version:

ansible --version

5. Install Ansible Galaxy Roles

If your role depends on any Ansible Galaxy roles:

# Install roles defined in requirements.yml
ansible-galaxy install -r requirements.yml


Example requirements.yml:

- src: geerlingguy.postgresql
  version: 4.12.0

6. Configure Environment Variables in Fish

Fish shell uses set instead of export. For example:

# Set database network range
set -x DB_NETWORK_IP_RANGE 10.0.0.0/16

# Set JSON users
set -x PG_USERS_JSON '[{"name": "dbuser", "password": "secret"}]'

# These variables will be available to Ansible using env lookup


Use set -U VAR value to set universal variables that persist across sessions.

7. Testing Ansible Role with Molecule

Inside your role directory:

# Initialize molecule scenario (only first time)
molecule init scenario --scenario-name default -r <role_name> -d docker

# Test the role
molecule test


Molecule will build Docker containers, apply your role, and run any tests defined.

8. Notes & Tips

Always activate your Python virtual environment before running molecule.

Use molecule converge to apply your role without destroying the container.

Use molecule login to enter the test container for debugging.

Environment variables in Fish must be exported with set -x to be visible to Ansible.

Keep Docker updated for better performance with Molecule tests.