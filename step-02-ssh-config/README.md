# Step 2 — SSH config (ProxyJump + ControlMaster)

Goal: short hostnames, jump through the Ada login node to any compute node, and faster re-connections.

## 1) Create/update `~/.ssh/config` on your Mac

Copy from the repo template:

```
scripts/templates/ssh_config.sample
```

Then edit and place into:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cp scripts/templates/ssh_config.sample ~/.ssh/config
sed -i.bak "s/<ADA_USER>/YOUR_USERNAME/g" ~/.ssh/config
```

Final shape should look like:

```sshconfig
Host ada
  HostName ada.iiit.ac.in
  User <ADA_USER>
  ForwardAgent yes
  ServerAliveInterval 30
  ServerAliveCountMax 120
  ControlMaster auto
  ControlPath ~/.ssh/cm-%r@%h:%p
  ControlPersist 10m

Host gnode-*
  User <ADA_USER>
  ProxyJump ada
  ServerAliveInterval 30
  ServerAliveCountMax 120
```

## 2) Test it

```bash
ssh ada   # should log you in as <ADA_USER> without password
```

You won't be able to `ssh gnode-XYZ` until you actually have an allocation, but the alias is ready for the tools.

**Next:** install OpenVSCode Server on Ada in [Step 3](../step-03-openvscode-server/README.md).