# Auditd Ansible Deployment

Deploy auditd to Ubuntu systems using ansible-pull.

## Usage
```bash
sudo ansible-pull -U https://github.com/BustinItDown/ansible-auditd.git
```

## Verify
```bash
sudo systemctl status auditd
sudo auditctl -l
```
