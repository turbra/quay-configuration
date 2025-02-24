# Quay Configuration Ansible Playbook

This repository contains an Ansible playbook that leverages the
`infra.quay_configuration` Ansible collection to initialize a Quay Container Registry
super user and create a new organization in Quay.

## Prerequisites

- **Ansible** (version 2.18.2 or later)
- Access to your Quay Container Registry instance
- The `infra.quay_configuration` collection installed. Install it using:

```bash
ansible-galaxy collection install infra.quay_configuration
```


## Configuration

### 1. Update Quay Host

Edit `group_vars/all.yml` and update the Quay API host variable. For example:

```yaml
quay_org_host: "https://registry-quay-quay-operator.apps.example.com"
quay_token: ""   # Leave this empty until after the admin-init step.
```

### 2. Update Super User Credentials

Edit `roles/admin-init/defaults/main.yml` with your super user details:

```yaml
# roles/admin-init/defaults/main.yml
quay_admin_username: "admin"
quay_admin_password: "changeme"
quay_admin_email: "quayadmin@example.com"
```

### 3. Update Organization Details

Edit `roles/org-init/defaults/main.yml` with your organization details:

```yaml
# roles/org-init/defaults/main.yml
org_name: "test-org"
org_email: "testquayadmin@example.com"
# (Other optional organization variables can be left as defaults or empty)
```

## Execution

### Step 1: Initialize Quay Admin

Run the playbook with the `admin-init` tag to initialize your Quay super user and generate an OAuth token:

```bash
ansible-playbook playbook.yml --tags admin-init
```

**Result Example:**

```
TASK [admin-init : Display the generated OAuth access token] **************************
ok: [localhost] => {
    "msg": "Access token: 1T8GQCRLF2XMWJZGMV1XKPQ7CZR2QU4ZM85FEWS7"
}
```
After this step, the playbook automatically updates your `group_vars/all.yml` file with the new access token. This ensures that subsequent playbook runs will use the current token without manual intervention.


### Step 2: Create Organization

Run the playbook with the `org-init` tag to create your organization and repository ("test-org"):

```bash
ansible-playbook playbook.yml --tags org-init
```

**Result Example:**
```
PLAY [Run specific Quay roles] ************************************************************************************************************************************************

TASK [org-create : Debug Quay connection and organization variables] **********************************************************************************************************
ok: [localhost] => {
    "msg": "quay_org_host is 'https://registry-quay-quay-operator.apps.example.com,', quay_org_token is 'OOQUB2UO4O2KHC9QCA3HLL510W65RSLV1X9GFBKJ', org_name is 'test-org', org_email is 'testquayadmin@example.com'"
}
```

This will create the organization using the details provided in `roles/quay-config/tasks/organization.yml`and a repository using the details provided in `roles/quay-config/tasks/repository.yml`.
