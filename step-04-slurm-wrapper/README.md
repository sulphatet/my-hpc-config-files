# Step 4 — Slurm wrapper (`gnode_vscode`) on Ada

Goal: a server-side helper that:
- requests a compute node via `srun` with your usual defaults,
- starts OpenVSCode Server (via `myvscode`),
- prints the compute node name,
- keeps the allocation alive in a tmux session.

## 1) Install the wrapper

On Ada:
```bash
mkdir -p ~/bin
cp ada/gnode_vscode ~/bin/
chmod +x ~/bin/gnode_vscode
```

## 2) Defaults (overridable via env)

* `PARTITION` (default `u22`)
* `ACCOUNT` (default `research`)
* `QOS` (default `medium`)
* `EXCLUDE` (empty by default)
* `JOB_NAME` (default `vscode`)
* `PORT` (default `8090`)
* resource flags `-g <GPUS> -c <CPUS> -m <MEM> -t <TIME>`

> **Note:** there is **no hard-coded GPU `--constraint`** in this script. Add your own if you need to, but the repo ships without it.

## 3) Smoke test on Ada

```bash
# This will allocate a node, start VS Code there, and print the node name
~/bin/gnode_vscode -g 1 -c 10 -m 24G -t 4:00:00 -p 8090
# You should see something like: NODE=gnode047
# Detach/stop later with: tmux ls ; tmux attach -t vsc-8090 ; Ctrl+C ; exit
```

**Next:** install the laptop helpers in [Step 5](../step-05-one-command-client/README.md).