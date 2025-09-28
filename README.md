My setup to call a SLURM node, launch **OpenVSCode Server**, optionally **sync data** from the login node’s long-term storage to node-local scratch, auto-tunnel a port, and open the IDE — all from a **single command on my Mac**.

Works great on **my university’s HPC - Ada**, but it should be generalizable to similar clusters.

---

## What you get

- `ada-vscode`: from your **laptop**
  - SSH → Ada
  - `srun` an interactive allocation via `gnode_vscode` (defaults baked in; overridable)
  - (optional) **`--sync <sub/dir>`**: rsync from `ada:/share1/<ADA_USER>/<sub/dir>` to `/scratch/<ADA_USER>/<sub/dir>` (or **`--ssd`** → `/ssd_scratch/...`)
  - start **OpenVSCode Server** on the node
  - open a **tunnel** to `http://localhost:<port>` in your browser

- `ada-vscode-stop`: (optionally) sync **back to /share1** and cleanly stop
  - `--sync <sub/dir>` copies from `/scratch|/ssd_scratch` on the node → `ada:/share1/<ADA_USER>/<sub/dir>`
  - cancels the Slurm job and kills the local tunnel

- Clear, stepwise docs for macOS SSH setup, ProxyJump, OpenVSCode Server, Slurm wrapper, sync patterns, and (optional) containerized AI env.

---

## Quick start (tl;dr)

> **Prereqs**: macOS with [Homebrew](https://brew.sh/)

1. **Clone this repo** on your Mac, then install the laptop scripts:
   ```bash
   git clone https://github.com/<you>/ada-vscode-starter.git
   cd ada-vscode-starter
   mkdir -p ~/bin
   cp scripts/ada-vscode scripts/ada-vscode-stop ~/bin/
   chmod +x ~/bin/ada-vscode ~/bin/ada-vscode-stop


Make sure `~/bin` is on your `PATH`.

2. **Set up SSH** (passwordless; ProxyJump):

   * Follow **[step-01-macos-ssh](step-01-macos-ssh/README.md)** and **[step-02-ssh-config](step-02-ssh-config/README.md)**.
   * Use the provided `scripts/templates/ssh_config.sample` as a base.

3. **Install OpenVSCode Server on Ada**:

   * Follow **[step-03-openvscode-server](step-03-openvscode-server/README.md)** to put the binary under `~/tools/openvscode/...`.
   * Add the `myvscode` function to your `~/.bashrc` (template provided in `ada/myvscode.example`).

4. **Install the Slurm wrapper on Ada**:

   * Follow **[step-04-slurm-wrapper](step-04-slurm-wrapper/README.md)** to install `ada/gnode_vscode` as `~/bin/gnode_vscode` on Ada (`chmod +x`).

5. **Launch a dev node with VS Code (from your Mac)**:

   ```bash
   # 1 GPU, 10 CPUs, 24G RAM, 4h; port 8090; no sync
   ada-vscode

   # Add a dataset sync from /share1/<ADA_USER>/datasets/foo -> /scratch/<ADA_USER>/datasets/foo
   ada-vscode --sync datasets/foo

   # Heavier session + SSD scratch + multiple syncs
   ada-vscode -g 4 -c 32 -m 64G -t 12:00:00 --ssd \
              --sync LMA_SLM/packed/seq1024 --sync tokenizers
   ```

6. **Stop the session (and sync back if you want)**:

   ```bash
   # just stop
   ada-vscode-stop

   # sync back to /share1 and then stop
   ada-vscode-stop --sync datasets/foo
   # or, if you had used --ssd:
   ada-vscode-stop --ssd --sync LMA_SLM/packed/seq1024
   ```

---

## What a successful run looks like

```text
$ ada-vscode --sync datasets/foo
[ada] Requesting node: ~/bin/gnode_vscode -g 1 -c 10 -m 24G -t 4:00:00 -p 8090
[ada] Allocated node: gnode047
[sync] Destination root on gnode047: /scratch/<ADA_USER>
[sync] ada:/share1/<ADA_USER>/datasets/foo  -->  gnode047:/scratch/<ADA_USER>/datasets/foo
sending incremental file list
... (rsync progress with stats) ...
[ssh] Tunneling localhost:8090 -> gnode047:8090 (via Ada)
[ok] VS Code is live at http://localhost:8090
```

And stopping with sync-back:

```text
$ ada-vscode-stop --sync datasets/foo
[sync-back] gnode047:/scratch/<ADA_USER>/datasets/foo/  -->  ada:/share1/<ADA_USER>/datasets/foo/
... (rsync progress with stats) ...
[stop] Cancelling Slurm job(s) named 'vscode' (if any)
[stop] Killing local tunnel on port 8090
[ok] VS Code allocation and tunnel shut down.
```

---

## How it works

1. **Laptop**: `ada-vscode` contacts **Ada** and runs `~/bin/gnode_vscode`.
2. **Ada**: `gnode_vscode` requests a node via `srun` (with your defaults), launches `myvscode` on the compute node inside a tmux, and prints the node name.
3. **(Optional) Sync**: laptop tells the node to `rsync` from `ada:/share1/<ADA_USER>/<sub/dir>` → `/scratch|/ssd_scratch/<ADA_USER>/<sub/dir>`.
4. **Tunnel**: laptop opens an SSH tunnel (`localhost:<port> → node:<port>`) and opens your browser to OpenVSCode Server.

> `/share1` is only on Ada’s **login** node; compute nodes cannot see it. That’s why we stage data with `rsync`. For many files, see [sync tips](step-06-sync-patterns/README.md).

---

## Defaults & overrides

**Server-side `gnode_vscode` defaults** (tweak in `~/bin/gnode_vscode` on Ada):

* `PARTITION`: `u22` (change for your cluster)
* `ACCOUNT`: `research`
* `QOS`: `medium`
* `CONSTRAINT`: *unset* (add your own if needed)
* `EXCLUDE`: *unset*
* `PORT`: `8090`
* `JOB_NAME`: `vscode`

**Client-side `ada-vscode` flags**:

```
-g, --gpus <N>       default 1
-c, --cpus <N>       default 10
-m, --mem <SIZE>     default 24G
-t, --time <HH:MM:SS>  default 4:00:00
-p, --port <PORT>    default 8090
--sync <sub/dir>     repeatable (e.g., --sync datasets/foo)
--ssd                use /ssd_scratch/<ADA_USER> instead of /scratch
--dry-run            print actions, do nothing
-q, --quiet          terser output
```

**Stop helper `ada-vscode-stop`**:

```
-p, --port <PORT>    which tunnel to kill (default 8090)
-n, --name <JOBNAME> Slurm job name (default vscode)
--sync <sub/dir>     copy back to /share1 from node local storage
--ssd                source is /ssd_scratch instead of /scratch
--node <gnodeXX>     specify node if job already ended
--dry-run            print actions, do nothing
-q, --quiet
