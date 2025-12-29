# Ansible Playbook to deploy Monad Node

This Ansible playbook automates the deployment and configuration of Monad Node. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator and node keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves secret environment file from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`

For example:
- [`monad/encapsulate/consensus/monad-consensus.secrets.env`](roles/consensus/templates/secrets.env.example)
- `monad/encapsulate/consensus/id-secp`
- `monad/encapsulate/consensus/id-bls`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/monad-ansible.git
cd monad-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for testnet.yml

```yaml
---
# maintains testnet inventory grouped by project names
all:
  vars:
    env: testnet
  children:
    monad:
      hosts:
        validator.monad.testnet.encapsulate.xyz:
        consensus.monad.testnet.encapsulate.xyz:
          type: consensus
        execution.monad.testnet.encapsulate.xyz:
          type: execution
          ledger_dir: /opt/monad-consensus/data
          socket_dir: /opt/monad-consensus/socket
        rpc.monad.testnet.encapsulate.xyz:
          type: rpc
          socket_dir: /opt/monad-consensus/socket
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the source urls and configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.

### Usage

1. First, install the dependencies:

  ```bash
   ansible-galaxy install -r requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Then run the playbook:

  ```bash
  ansible-playbook setup_consensus.yml -l consensus.monad.testnet.encapsulate.xyz
  ```

4. Similarly to deploy other services

  ```bash
  ansible-playbook setup_execution.yml -l execution.monad.testnet.encapsulate.xyz
  ansible-playbook setup_rpc.yml -l rpc.monad.testnet.encapsulate.xyz
  ```

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [consensus.monad.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: consensus.monad.testnet.encapsulate.xyz",
        "Groups: ['monad']",
        "Project: monad",
        "Environment: testnet",
        "Type: consensus",
        "Version: 0.1.0-testnet-2-genesis",
        "Username: monad",
        "Service Name: monad",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] ********************************************************************************************************************
Pausing for 40 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [consensus.monad.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [consensus.monad.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: consensus.monad.testnet.encapsulate.xyz",
        "Project: monad",
        "Environment: testnet",
        "Type: consensus"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [consensus.monad.testnet.encapsulate.xyz]
```
