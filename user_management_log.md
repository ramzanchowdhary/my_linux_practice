# Linux User and Group Management Practice

## Date: 2025-06-12

### User Management:
- Created a new user 'alam':
  `sudo useradd -m alam`
- Set password for 'alam':
  `sudo passwd alam`
- Modified 'alam' to add to 'sudo' group (later removed for specific access):
  `sudo usermod -aG sudo alam`
- Removed 'alam' from 'sudo' group:
  `sudo gpasswd -d alam sudo`
- Granted 'alam' specific sudo access to 'updatedb':
  `sudo visudo -f /etc/sudoers.d/alam-updatedb`
  (Added line: `alam ALL= (root) NOPASSWD: /usr/bin/updatedb`)

### Group Management:
- Created a new group 'devs':
  `sudo groupadd devs`
- Added user 'nadim' to 'devs' group:
  `sudo usermod -aG devs nadim`

## Learnings:
- Git tracks files within its directory. System changes need to be documented.
- Used `visudo` to safely edit sudoers files.
