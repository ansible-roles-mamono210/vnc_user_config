---
[![CircleCI](https://circleci.com/gh/ansible-roles-mamono210/vnc_user_config/tree/main.svg?style=svg)](https://circleci.com/gh/ansible-roles-mamono210/vnc_user_config/tree/main)

Role Description
================

Configures per-user TigerVNC settings and starts the VNC systemd service on Enterprise Linux 8/9 (e.g. CentOS Stream 9).

This role:

- Configures `~/.vnc` for an existing user
- Sets VNC password
- Creates `/etc/tigervnc/vncserver.users`
- Enables and starts `vncserver@.service`

This role does **NOT** install packages or GUI components.
Package installation must be handled by a separate role (e.g. `mamono210.vnc_install`) during Golden Image creation.

---

Requirements
------------

- Enterprise Linux 8/9
- `tigervnc-server` must already be installed
- Target user must already exist (recommended)

This role is intended to run **after VM boot**, not during image build.

---

Role Variables
--------------

### vnc_user

Target user to configure.

```yaml
vnc_user: your_user
````

---

### vnc_display

VNC display number (format `:1`, `:2`, etc.).

```yaml
vnc_display: ":1"
```

---

### vnc_geometry

Screen resolution.

```yaml
vnc_geometry: "1280x800"
```

---

### vnc_depth

Color depth.

```yaml
vnc_depth: "24"
```

---

### vnc_session_command

Desktop session command.

Examples:

```yaml
vnc_session_command: "gnome-session"
```

---

### vnc_password

Password for VNC access.

⚠ This value must be provided via Vault or runtime variables.
It must not be committed to the repository.

Example:

```yaml
vnc_password: "{{ vault_vnc_password }}"
```

---

### vnc_manage_user

Whether the role should create the user if it does not exist.

Default:

```yaml
vnc_manage_user: false
```

Recommended: keep `false` and manage users separately.

---

### vnc_manage_service

Whether to enable and start the VNC systemd service.

Default:

```yaml
vnc_manage_service: true
```

---

## Dependencies

None.

This role intentionally does not install TigerVNC packages.

---

## Example Playbook

```yaml
---
- hosts: all
  become: true

  roles:
    - role: mamono210.vnc_user_config
      vars:
        vnc_user: your_user
        vnc_display: ":1"
        vnc_geometry: "1280x800"
        vnc_depth: "24"
        vnc_session_command: "gnome-session"
        vnc_password: "{{ vault_vnc_password }}"
        vnc_manage_user: false
        vnc_manage_service: true
```

---

## Operational Model

This role is designed for a two-phase deployment model.

### Phase 1 – Image Build

Use a separate role:

```yaml
mamono210.vnc_install
```

Responsibilities:

* Install GUI
* Install TigerVNC packages
* Do NOT configure users
* Do NOT set passwords
* Do NOT start services

---

### Phase 2 – VM Runtime Configuration

Use this role:

```yaml
mamono210.vnc_user_config
```

Responsibilities:

* Configure `~/.vnc`
* Set VNC password
* Map display to user
* Enable and start service

---

## Security Notes

* `vnc_password` is handled with `no_log: true`
* Do not execute this role during Golden Image creation
* Do not store passwords in defaults or group_vars
* Use Ansible Vault or CI secret injection

---

## License

BSD
