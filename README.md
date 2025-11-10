# Auditd Deployment with Ansible

## Overview

This project has an Ansible playbook that sets up and configures **auditd** (the Linux Audit Daemon) automatically on Ubuntu. It's meant to be pulled from GitHub using `ansible-pull`, so your system just grabs the latest setup and applies it on its own.

### Why Auditd?

Auditd is a really important security tool. It:

* Watches system events and logs what happens
* Helps with compliance stuff like PCI-DSS, HIPAA, etc.
* Records detailed logs for investigations
* Can catch suspicious activity or privilege escalation
* Keeps track of file or config changes

Basically, it's used for security, compliance, and figuring out what happened after something goes wrong.

---

## Architecture

![Architecture Diagram](DIAGRAM.png)

The diagram shows how ansible-pull clones this repo from GitHub, runs the playbook on your Ubuntu VM, installs and configures auditd, which then monitors system events and writes logs. An optional cron job can automate daily updates.

---

## Requirements

### OS

* **Ubuntu 24.04 LTS** (tested)
* Ubuntu 22.04 LTS (should also work)
* 1GB RAM minimum
* 10GB free space (logs can get big)

### Software

* **Ansible** 2.9+
* **Git**
* **Python 3.8+**
* **Sudo or root access**

### Network

* Needs internet to reach GitHub
* Access to Ubuntu package repos (for apt install)

---

## Project Layout

```
ansible-auditd/
├── local.yml
├── README.md
├── images/
│   └── architecture-diagram.png
└── roles/
    └── auditd/
        ├── tasks/main.yml
        ├── templates/
        │   ├── auditd.conf.j2
        │   └── audit.rules.j2
        └── handlers/main.yml
```

### What's What

* **local.yml** — the main playbook (entry point)
* **tasks/main.yml** — installs and configures auditd
* **templates/** — config files for auditd and its rules
* **handlers/main.yml** — restarts auditd if config changes

---

## How It Works

1. Your system runs `ansible-pull`
2. It clones this repo to `/root/.ansible/pull/`
3. It runs the playbook (`local.yml`)
4. Ansible installs and configures **auditd**
5. Auditd starts monitoring your system and logging stuff under `/var/log/audit/`

Logs are rotated after 8MB and only root can read them.

---

## Quick Start

First install Ansible and Git, then run ansible-pull to deploy. The playbook clones this repo, installs auditd, and applies all the configs.



---

## What It Monitors

### System Self-Monitoring

Auditd's own log files and configuration files.

### Identity and Authentication

User and password files.

### Logins

Login and logout activity.

### Privilege Escalation

Sudo configuration files.

### SSH Config

SSH settings and configuration.

---

## Testing It

Verify auditd is running and monitoring system events. Check that audit rules are loaded and logs are being written to `/var/log/audit/audit.log`.

---

## Common Problems

### 1. ansible-pull can't clone repo

Check your internet or DNS settings.

### 2. auditd won't start

Check config syntax or logs:

```bash
sudo journalctl -u auditd
```

### 3. No rules loaded

Try:

```bash
sudo augenrules --load
```

### 4. Low disk space

Clear old logs or adjust rotation in `/etc/audit/auditd.conf`.

### 5. Permission denied

Make sure only root owns `/var/log/audit/audit.log`:

```bash
sudo chown root:root /var/log/audit/audit.log
sudo chmod 0600 /var/log/audit/audit.log
```

---

## Maintenance Tips

* **Check logs weekly**

  ```bash
  sudo aureport --start week-ago
  ```

* **Watch disk space**

  ```bash
  df -h /var/log/audit/
  ```

* **Backup logs**

  ```bash
  sudo tar -czf /backup/audit-logs-$(date +%Y%m%d).tar.gz /var/log/audit/
  ```

---

## Compliance Stuff (Short Version)

This setup helps meet parts of:

* **PCI-DSS** (logs all user actions)
* **HIPAA** (audit controls)
* **SOX** (tracks changes and access)

---

## Notes

This playbook is mainly for practice and learning how to automate Linux security tools with Ansible. It's not enterprise-grade, but it gives a good base setup for understanding how `auditd` works and how to manage it through configuration management.
