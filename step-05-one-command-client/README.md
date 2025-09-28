# Step 5 — One-command client (Mac): `ada-vscode` & `ada-vscode-stop`

Goal: from your **Mac**, run a single command that:
- SSH → Ada → `gnode_vscode` (gets you a gnode allocation),
- **optionally syncs** data from `/share1/<ADA_USER>/<sub/dir>` to the node's local `/scratch` (or `/ssd_scratch`),
- opens a local **tunnel** and launches the browser to OpenVSCode Server.

Also provide a stop command that can sync **back** and tear everything down.

## 1) Install the scripts

```bash
mkdir -p ~/bin
cp scripts/ada-vscode scripts/ada-vscode-stop ~/bin/
chmod +x ~/bin/ada-vscode ~/bin/ada-vscode-stop
```

Ensure `~/bin` is on your PATH:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

## 2) Usage

### Launch

```bash
# default: 1 GPU, 10 CPU, 24G, 4h, port 8090
ada-vscode

# sync a dataset from /share1/<ADA_USER>/datasets/foo to /scratch/<ADA_USER>/datasets/foo
ada-vscode --sync datasets/foo

# heavier session with SSD scratch and multiple syncs
ada-vscode -g 4 -c 32 -m 64G -t 12:00:00 --ssd \
           --sync LMA_SLM/packed/seq1024 --sync tokenizers
```

### Stop

```bash
# just stop
ada-vscode-stop

# sync back from /scratch to /share1 before stopping
ada-vscode-stop --sync datasets/foo

# if you had used --ssd:
ada-vscode-stop --ssd --sync LMA_SLM/packed/seq1024
```

## 3) Flags (client)

`ada-vscode`:

* `-g, --gpus <N>` (default 1)
* `-c, --cpus <N>` (default 10)
* `-m, --mem <SIZE>` (default 24G)
* `-t, --time <HH:MM:SS>` (default 4:00:00)
* `-p, --port <PORT>` (default 8090)
* `--sync <sub/dir>` (repeatable)
* `--ssd` (use `/ssd_scratch/<ADA_USER>` instead of `/scratch/<ADA_USER>`)
* `--dry-run`, `-q|--quiet`

`ada-vscode-stop`:

* `-p, --port <PORT>` (tunnel to kill; default 8090)
* `-n, --name <JOBNAME>` (Slurm job name; default `vscode`)
* `--sync <sub/dir>` (repeatable; syncs back to `/share1/<ADA_USER>`)
* `--ssd` (source is `/ssd_scratch`)
* `--node <gnodeXX>` (if job already ended)
* `--dry-run`, `-q|--quiet`

## 4) What success looks like

```text
$ ada-vscode --sync datasets/foo
[ada] Requesting node: ~/bin/gnode_vscode -g 1 -c 10 -m 24G -t 4:00:00 -p 8090
[ada] Allocated node: gnode047
[sync] Destination root on gnode047: /scratch/<ADA_USER>
[sync] ada:/share1/<ADA_USER>/datasets/foo  -->  gnode047:/scratch/<ADA_USER>/datasets/foo
... (rsync progress) ...
[ssh] Tunneling localhost:8090 -> gnode047:8090 (via Ada)
[ok] VS Code is live at http://localhost:8090
```

**Next:** learn sync best practices in [Step 6](../step-06-sync-patterns/README.md).