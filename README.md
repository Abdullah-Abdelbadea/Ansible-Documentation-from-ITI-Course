#  Ansible – Complete Detailed Documentation

This repository contains **complete, structured, and production‑ready Ansible documentation**.  
It is designed to be used as:
- Study notes
- Interview preparation
- Real‑world reference
- Enterprise‑style Ansible guide

---

## 1. What is Ansible?

**Ansible** is an open-source automation engine designed to manage systems by enforcing a **desired state**.

It is widely used in DevOps and SRE environments to:
- Configure operating systems
- Install and manage software packages
- Deploy applications
- Orchestrate multi-tier systems

Unlike traditional scripting, Ansible focuses on **state-driven automation**, not step-by-step instructions.

Key idea:
> You describe *how the system should look*, and Ansible figures out *how to get there*.

---

## 2. How Ansible Works (Architecture)

Ansible uses a **push‑based, agentless architecture**.

### Core Components
- **Control Node**: The machine where Ansible is installed and executed.
- **Managed Nodes**: Target machines managed by Ansible.
- **Connection**: SSH (Linux) or WinRM (Windows).

 No agents, no daemons, no database.

---

## 3. Why Ansible?

### Agentless
- No software installed on managed nodes
- Lower overhead
- Easier maintenance

### Idempotent
Running the same playbook multiple times results in the same system state.

```yaml
- user:
    name: ali
    state: present
```

### Declarative
You define **what the system should look like**, not how to reach it.

---

## 4. Inventory

### What is Inventory?
Inventory defines **which hosts Ansible manages**.

### Default Inventory File
```bash
/etc/ansible/hosts
```

### Static Inventory Example
```ini
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

### Inventory with Variables
```ini
[webservers]
192.168.1.10 nginx_port=80
192.168.1.11 nginx_port=8080
```

### Group Hierarchy
```ini
[production:children]
webservers
dbservers
```

---

## 5. Ansible Configuration File (`ansible.cfg`)

Controls Ansible behavior globally.

### Common Locations
- `./ansible.cfg`
- `~/.ansible.cfg`
- `/etc/ansible/ansible.cfg`

### Example Configuration
```ini
[defaults]
inventory = ./inventory
remote_user = ubuntu
host_key_checking = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

---

## 6. Variables in Ansible

### Variable Precedence (High → Low)
1. Extra vars (`-e`)
2. Playbook vars
3. host_vars
4. group_vars
5. Inventory vars
6. Role defaults

### Inventory Variables
```ini
192.168.1.10 app_name=myapp
```

### Playbook Variables
```yaml
vars:
  app_name: myapp
```

### Extra Variables
```bash
ansible-playbook play.yml -e "app_name=myapp"
```

---

## 7. group_vars and host_vars (Best Practice)

### Purpose
- Separate configuration from logic
- Improve scalability
- Reduce duplication

### Recommended Structure
```text
inventory/
├── hosts
├── group_vars/
│   └── webservers.yml
├── host_vars/
│   └── server1.yml
```

---

## 8. Ad‑Hoc Commands

Used for quick, one‑time operations.

### Syntax
```bash
ansible -i inventory hosts -m module -a "arguments"
```

### Examples
```bash
ansible all -m ping
ansible webservers -m command -a "uptime"
```

---

## 9. Playbooks

### What is a Playbook?
A **playbook** is a YAML file that defines **one or more plays**.

Each play maps:
- A group of hosts
- To a set of tasks or roles

Playbooks are the **entry point** of Ansible automation.

### Core Elements of a Playbook
- `hosts`: target machines
- `tasks`: actions to perform
- `vars`: variables scoped to the play
- `handlers`: reactive tasks
- `roles`: modular task groups

### Example Explained
```yaml
- name: Configure Web Servers
  hosts: webservers
  become: true
  tasks:
    - name: Ensure nginx is installed
      apt:
        name: nginx
        state: present
```

Explanation:
- `hosts`: defines the inventory group
- `become`: enables privilege escalation
- `tasks`: list of ordered operations

Tasks run **sequentially**, host by host.

---

## 10. Modules vs Command

### Bad Practice
```yaml
- command: apt install nginx -y
```

### Best Practice
```yaml
- apt:
    name: nginx
    state: present
```

### Why Modules Are Better
- Idempotent
- Structured output
- Built‑in error handling

---

## 11. Loops

### Simple Loop
```yaml
- user:
    name: "{{ item }}"
    state: present
  loop:
    - ali
    - sam
```

### Loop with Dictionaries
```yaml
- user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
  loop:
    - { name: ali, uid: 1001 }
    - { name: sam, uid: 1002 }
```

---

## 12. register

Stores task output in a variable.

```yaml
- command: echo hello
  register: cmd_result
  changed_when: false
```

Stored values include `stdout`, `stderr`, `rc`, and `changed`.

---

## 13. changed_when

Controls when a task reports `changed`.

```yaml
changed_when: false
```

---

## 14. failed_when

Controls when a task is considered failed.

```yaml
failed_when: result.rc != 0
```

---

## 15. Conditionals (`when`)

```yaml
- apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"
```

---

## 16. Handlers and notify

### Why Handlers Exist
Some operations (like restarting services) should only happen **when a change occurs**.

Handlers solve this problem by being:
- Event-driven
- Conditional
- Idempotent

---

### notify

`notify` links a task to a handler.

```yaml
- name: Deploy nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx
```

Behavior:
- If the file content changes → handler is queued
- If no change → handler is skipped

---

### Handlers

Handlers are defined separately and run:
- Only if notified
- Only once per play
- At the end of the play execution

```yaml
handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

If multiple tasks notify the same handler, it still runs **only once**.

---

## 17. Templates (Jinja2)

### What Are Templates?

Templates are **dynamic configuration files** generated using **Jinja2**.

Instead of copying static files, templates allow Ansible to:
- Inject variables
- Apply logic (conditions, loops)
- Generate different configs for different hosts or environments

Templates are rendered **on the control node**, then transferred to the managed node.

---

### Why Templates Are Important

Without templates:
- You duplicate configuration files
- You hardcode values
- Scaling becomes painful

With templates:
- One file works for many servers
- Configuration becomes environment-aware
- Roles become reusable

---

### Basic Template Example

#### Template file: `nginx.conf.j2`
```jinja2
server {
  listen {{ nginx_port }};
  server_name {{ server_name }};

  location / {
    root {{ document_root }};
    index index.html index.htm;
  }
}
```

#### Variables (group_vars or host_vars)
```yaml
nginx_port: 80
server_name: example.com
document_root: /var/www/html
```

#### Playbook Task
```yaml
- name: Deploy nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-enabled/default
    owner: root
    group: root
    mode: '0644'
  notify: Restart nginx
```

---

### How Template Rendering Works (Execution Flow)

1. Ansible loads the template file (`.j2`)
2. Variables are resolved (inventory → group_vars → host_vars → play vars)
3. Jinja2 renders the final file
4. Ansible compares rendered file with destination
5. If content differs:
   - File is updated
   - Task reports `changed`
   - Handlers are notified

---

### Conditional Logic in Templates

Templates can contain conditions.

```jinja2
{% if enable_ssl %}
listen 443 ssl;
{% else %}
listen 80;
{% endif %}
```

Variables:
```yaml
enable_ssl: true
```

---

### Loops Inside Templates

Example: multiple upstream servers

```jinja2
upstream backend {
{% for server in backend_servers %}
  server {{ server }};
{% endfor %}
}
```

Variables:
```yaml
backend_servers:
  - 10.0.0.10
  - 10.0.0.11
```

---

### Using Filters in Templates

Filters transform variables.

```jinja2
server_name {{ domain | upper }};
```

Common filters:
- `upper`
- `lower`
- `default`
- `length`

---

### Safe Defaults in Templates

```jinja2
listen {{ nginx_port | default(80) }};
```

Prevents failures when a variable is missing.

---

### Templates vs Copy Module

| Feature | template | copy |
|------|---------|------|
| Variable substitution | Yes | No |
| Logic (if/for) | Yes | No |
| Dynamic configs | Yes | No |
| Static files | No | Yes |

---

### Common Template Mistakes

- Hardcoding values inside templates
- Using templates when `copy` is sufficient
- Not using `default()` filter
- Restarting services without `notify`

---

### Best Practices for Templates

- Keep templates simple
- Put logic in variables, not templates
- Always use handlers with templates
- Use defaults for safety

---

## 18. Gathering Facts (Ansible Facts)

### What is Fact Gathering?

**Fact gathering** is the process where Ansible automatically collects information about managed nodes **before executing tasks**.

These collected details are called **Ansible Facts**.

Facts describe the **current state of the system**, such as:
- Operating system
- IP addresses
- CPU and memory
- Disk layout
- Network interfaces
- Kernel version

---

### When Facts Are Gathered

By default, Ansible gathers facts:
- At the **start of every play**
- Before any task is executed

This behavior is controlled by:
```yaml
gather_facts: true
```

---

### Example: Default Behavior

```yaml
- hosts: all
  tasks:
    - debug:
        var: ansible_os_family
```

Even though no task gathers facts explicitly, Ansible already knows `ansible_os_family`.

---

### Disabling Fact Gathering

Fact gathering can be disabled to:
- Speed up execution
- Reduce overhead
- Optimize large inventories

```yaml
- hosts: all
  gather_facts: false
  tasks:
    - debug:
        msg: "Facts are disabled"
```

---

### Commonly Used Facts

```yaml
ansible_os_family
ansible_distribution
ansible_distribution_version
ansible_hostname
ansible_facts['default_ipv4']['address']
ansible_processor_vcpus
ansible_memtotal_mb
```

---

### Using Facts in Conditions

Facts are commonly used with `when` conditions.

```yaml
- name: Install nginx on Debian systems
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"
```
### Different OS = Different Package Manager
```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  when: ansible_facts.os_family == "Debian"

- name: Install nginx
  yum:
    name: nginx
    state: present
  when: ansible_facts.os_family == "RedHat"
```

---

### Using Facts in Templates

Facts can be injected directly into templates.

```jinja2
# Generated on {{ ansible_hostname }}
# OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
server_name {{ ansible_facts.hostname }};

{% if ansible_facts.memtotal_mb > 4096 %}
worker_processes auto;
{% endif %}


```

---

### Custom Facts

Custom facts can be defined on managed nodes.

Example file:
```text
/etc/ansible/facts.d/app.fact
```

Content:
```ini
[app]
version=1.2.3
environment=production
```

Accessed as:
```yaml
ansible_facts['app']['version']
```

---

### setup Module (Manual Fact Gathering)

The `setup` module is responsible for collecting facts.

```bash
ansible all -m setup
```

To limit facts:
```bash
ansible all -m setup -a "filter=ansible_distribution*"
```

---

### Performance Considerations

- Fact gathering can be slow on large inventories
- Disable it when facts are not needed
- Use `filter` to limit collected data

---

### Best Practices for Fact Gathering

- Disable facts for simple ad-hoc tasks
- Use facts for OS-specific logic
- Avoid overusing facts in templates
- Prefer explicit variables when possible

---

## 19. Roles (Implementation Layer)

### What is a Role?
A **role** is a standardized directory structure that encapsulates:
- Tasks
- Variables
- Templates
- Handlers
- Files

Roles are the foundation of **modular and reusable Ansible code**.

Each role should:
- Have a single responsibility
- Be environment-agnostic
- Be configurable through variables

---

### Creating a Role
```bash
ansible-galaxy init nginx
```

---

### Role Directory Structure Explained

```text
roles/nginx/
├── tasks/main.yml      # Main execution logic
├── handlers/main.yml   # Reactive actions
├── templates/          # Jinja2 templates
├── defaults/main.yml   # Lowest-priority variables
├── vars/main.yml       # High-priority variables
├── files/              # Static files
```

#### tasks/
Contains ordered, idempotent tasks.

#### defaults/
Safe default values meant to be overridden.

#### vars/
Role-internal values that should not be overridden lightly.

---

## 20. Plays Folder (Orchestration Layer)

Defines **what runs where**, not how.

```text
plays/
├── web.yml
├── db.yml
```

```yaml
- hosts: webservers
  become: true
  roles:
    - nginx
    - php
```

---

## 21. Plays vs Roles Execution Flow

```text
ansible-playbook plays/web.yml
        ↓
Select hosts
        ↓
Apply roles
        ↓
Run role tasks
        ↓
Trigger handlers if notified
```

---
---

## Ansible Galaxy

### What is Ansible Galaxy?
A public repository for reusable Ansible roles.

### Commands
```bash
ansible-galaxy install username.role_name
ansible-galaxy init role_name
```
---
## 22. Best Practices Summary

- Use modules instead of command
- Keep playbooks thin
- Use roles for implementation
- Separate variables
- Avoid hardcoding values
- Think idempotent
- Use notify and handlers correctly

---

## Final Mental Model

**Playbooks decide WHAT runs and WHERE**  
**Roles decide HOW things are done**

---

# Author : Abdullah Abdelbadea 


