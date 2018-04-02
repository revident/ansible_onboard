https://asciinema.org/a/174003
[![asciicast](https://asciinema.org/a/174003.png)](https://asciinema.org/a/174003)

# Goals of this playbook

1. Set the hostname.
2. Create a dedicated unprivileged user.
3. Make it so that unprivileged user can only be logged into with SSH keys.
4. Modify PAM and sudo to challenge a forwarded ssh agent using SSH keys instead of needing the unprivileged users password (which is not set).
5. Leave regular sudo and password logins unchanged for normal authentication of all users.

## SSH
Make sure you have an ssh-agent running. Most modern Desktop Environments will run one for you.
```
# env | egrep -e 'SSH_AGENT_PID' -e 'SSH_AUTH_SOCK'
SSH_AUTH_SOCK=/tmp/ssh-kk1nhBkr22z5/agent.2530
SSH_AGENT_PID=2553
```

Add your SSH key to the agent.
```
# ssh-add
Enter passphrase for /home/revident/.ssh/id_rsa:
Identity added: /home/revident/.ssh/id_rsa (/home/revident/.ssh/id_rsa)
# ssh-add -l
2048 SHA256:iVYAfZHYb3KBWPZKOVVHjcTZ3dkzcQSY3DuUflypvH8 /home/revident/.ssh/id_rsa (RSA)
```

And Make sure you have agent forwarding turned on in your ssh config.
```
# grep ForwardAgent ~/.ssh/config
ForwardAgent yes
```

## Ansible

### First Run

The very first run of the playbook requires you connect as root, as on a new system that is the only privilaged user.
```
ansible-playbook onboard.yml -u root -k
```

### Later Runs
When using ansible 'become' to escalate to a privileged account, ansible needs you to give a password for sudo.
The changes made by this playbook make using a sudo password irrelevant. PAM will instead check the unprivileged users authorized_keys file. PAM will then use those public keys to challenge that the forwarded SSH agent has the private key, and grant authorization to sudo. So, give ansible any invalid text as the sudo password, it won't be used.

```
ansible-playbook onboard.yml -u svcuser -K 
SUDO password: # hit anykey and <return>
```

## Customize this Playbook

All the variables that define the service account (svcuser in this example), are defined in:
```
/roles/onboard/vars/main.yml
```
Please change those to something suitable for your environment.
