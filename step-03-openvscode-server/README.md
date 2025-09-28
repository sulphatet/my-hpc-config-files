# Step 3 — OpenVSCode Server on Ada

Goal: install OpenVSCode Server in your home directory and provide a `myvscode` function to launch it bound to localhost.

## 1) Download & unpack OpenVSCode Server (on Ada)

```bash
ssh ada
mkdir -p ~/tools
cd ~/tools
# example using latest x64 linux tarball; adjust if version changes
curl -L https://github.com/gitpod-io/openvscode-server/releases/latest/download/openvscode-server-linux-x64.tar.gz -o openvscode-server.tar.gz
tar xf openvscode-server.tar.gz
mv openvscode-server-* openvscode
```

Directory after install:

```
~/tools/openvscode/bin/openvscode-server
```

## 2) Define the launcher function `myvscode`

Append the example from this repo to your `~/.bashrc` (on Ada):

```bash
cat ada/myvscode.example >> ~/.bashrc
source ~/.bashrc
```

The function will:

* start OpenVSCode Server on `127.0.0.1:<PORT>` (defaults to 8090),
* print simple instructions (the laptop script will handle tunneling automatically).

## 3) Quick local test (optional)

From the Ada login node (not a compute node):

```bash
PORT=8090 myvscode
# then, from your Mac in another terminal:
ssh -L 8090:localhost:8090 <ADA_USER>@ada.iiit.ac.in
# open http://localhost:8090
```

You should see VS Code. Stop it with `Ctrl+C` on the Mac terminal that created the tunnel.

**Next:** install the Slurm wrapper in [Step 4](../step-04-slurm-wrapper/README.md).