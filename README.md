# Ansible Role Development & Testing Setup

This guide helps you set up a development environment for testing Ansible roles using **Molecule** with **Docker** on a system using the **Fish shell**.

---

## Prerequisites

Make sure you have the following installed:

- Python 3.11+
- Docker & Docker Compose
- Fish shell (`fish`)
- Git

### Check Versions

```fish
python3 --version
docker --version
docker-compose --version
fish --version
git --version
```

---

## Python Virtual Environment Setup

It's recommended to use a Python virtual environment to isolate dependencies:

### Create and Activate Virtual Environment

```fish
# Create a venv
python3 -m venv ~/ansible-molecule-venv

# Activate the venv (fish shell)
source ~/ansible-molecule-venv/bin/activate.fish

# Upgrade pip
pip install --upgrade pip
```

### Deactivate Virtual Environment

```fish
deactivate
```

---

## Installation

### Install Molecule & Docker Driver

With the virtual environment activated:

```fish
pip install molecule molecule-docker docker
```

This will allow you to test Ansible roles using Docker containers as test instances.

### Install Ansible

If Ansible is not installed:

```fish
pip install ansible
```

### Verify Installation

```fish
ansible --version
```

---

## Ansible Galaxy Roles

If your role depends on any Ansible Galaxy roles:

```fish
# Install roles defined in requirements.yml
ansible-galaxy install -r requirements.yml
```

### Example requirements.yml

```yaml
- src: geerlingguy.postgresql
  version: 4.12.0
```

---

## Environment Configuration

### Fish Shell Environment Variables

Fish shell uses `set` instead of `export`. For example:

```fish
# Set database network range
set -x DB_NETWORK_IP_RANGE 10.0.0.0/16

# Set JSON users
set -x PG_USERS_JSON '[{"name": "dbuser", "password": "secret"}]'

# These variables will be available to Ansible using env lookup
```

**Note:** Use `set -U VAR value` to set universal variables that persist across sessions.

---

## Testing with Molecule

### Initialize Molecule Scenario

Inside your role directory (only needed the first time):

```fish
# Initialize molecule scenario
molecule init scenario --scenario-name default -r <role_name> -d docker
```

### Run Tests

```fish
# Test the role
molecule test
```

Molecule will build Docker containers, apply your role, and run any tests defined.

---

## Additional Commands

### Development Commands

```fish
# Apply role without destroying container
molecule converge

# Enter the test container for debugging
molecule login
```

---

## Notes & Tips

- **Always activate your Python virtual environment** before running molecule
- Use `molecule converge` to apply your role without destroying the container
- Use `molecule login` to enter the test container for debugging
- Environment variables in Fish must be exported with `set -x` to be visible to Ansible
- Keep Docker updated for better performance with Molecule tests



Example : https://community.hetzner.com/tutorials/testing-ansible-with-molecule