---
title: Logrotate config
weight: 402
menu:
  notes:
    name: Logrotate
    identifier: notes-logrotate
    parent: notes-linux
    weight: 42
---
<div style="display: block; width: 100%; max-width: none;">
{{< note title="Script: add sudo user" >}}
Create a sudo user that won't prompt for password on executing sudo commands.
```bash
#!/bin/bash

# Ensure script is run as root
if [[ $EUID -ne 0 ]]; then
   echo "❌ This script must be run as root"
   exit 1
fi

# Prompt for new username
read -p "Enter new username: " username

# Check if user already exists
if id "$username" &>/dev/null; then
    echo "⚠️ User '$username' already exists."
    exit 1
fi

# Prompt for password (silent input)
read -s -p "Enter password for $username: " password
echo
read -s -p "Confirm password: " password_confirm
echo

# Check passwords match
if [[ "$password" != "$password_confirm" ]]; then
    echo "❌ Passwords do not match."
    exit 1
fi

# Create user with home directory and bash shell
useradd -m -s /bin/bash "$username"

# Set user password
echo "${username}:${password}" | chpasswd

# Add user to sudo group
usermod -aG sudo "$username"

# Create a sudoers file to allow passwordless sudo
echo "$username ALL=(ALL) NOPASSWD:ALL" > "/etc/sudoers.d/$username"
chmod 440 "/etc/sudoers.d/$username"

echo "✅ User '$username' created with bash shell and passwordless sudo access."
```
{{< /note >}}
</div>