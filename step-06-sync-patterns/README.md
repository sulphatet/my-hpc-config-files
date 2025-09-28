# Step 6 — Sync patterns & best practices

Compute nodes don't see `/share1`. The pattern is:
- stage **from** `ada:/share1/<ADA_USER>/<sub/dir>` **to** `/scratch|/ssd_scratch/<ADA_USER>/<sub/dir>` on the compute node **before** you work,
- do your work on node-local storage,
- (optional) sync **back** to `/share1` when done.

The client scripts do this for you via `--sync` (forward) and `ada-vscode-stop --sync` (backward).

## 1) Why rsync?

- Incremental (only changed files), safe resume with `--partial --inplace`.
- Over SSH, no special privileges.
- Verbose progress/stats (`--info=stats2,progress2`).

## 2) Recommended flags (already used by the scripts)

```
-avh --delete --partial --inplace --info=stats2,progress2 --no-inc-recursive
--exclude .git/ --exclude **pycache**/
```

**Notes:**
- Avoid `-z` on fast LAN; compression can slow things down for already-compressed data.
- Add `--checksum` only when you suspect corruption (slower; reads all bytes both sides).

## 3) SSD vs HDD scratch

Use `--ssd` to land under `/ssd_scratch/<ADA_USER>` if your node has it and you need faster I/O. Otherwise default is `/scratch/<ADA_USER>`.

Examples:
```bash
ada-vscode --ssd --sync datasets/foo
ada-vscode-stop --ssd --sync outputs/run42
```

## 4) Many tiny files? Consider tar-first (optional)

For very large trees of tiny files, an initial tar stream can be faster:

**Initial clone:**

```bash
# from node, pull via tar stream
ssh ada "cd /share1/<ADA_USER>/datasets/foo && tar -cf - ." | tar -xf - -C "/scratch/<ADA_USER>/datasets/foo"
```

**Subsequent updates:** use `ada-vscode --sync datasets/foo` (rsync becomes efficient).

## 5) Per-node warm cache (advanced)

If you run multiple jobs on the **same node**, keep a read-only cache and link-copy per job:

```bash
CACHE=/scratch/<ADA_USER>/cache/datasets/foo
rsync -a --delete ada:/share1/<ADA_USER>/datasets/foo/ "$CACHE/"
RUN="$CACHE.runs/${SLURM_JOB_ID:-manual}"
rsync -a --link-dest="$CACHE" "$CACHE/" "$RUN/"
# train using $RUN; write outputs elsewhere
```

## 6) Sync back on stop

```bash
# copy results back to /share1 then stop the session
ada-vscode-stop --sync results/myexp
# if you used SSD:
ada-vscode-stop --ssd --sync results/myexp
```

## 7) Common pitfalls

* **Wrong username:** your Mac user may differ from `<ADA_USER>`. The scripts auto-detect your Ada username; if you hard-run rsync, be sure to use the right path.
* **Source doesn't exist:** verify path on Ada: `ssh ada 'ls -lah /share1/<ADA_USER>/<sub/dir>'`.
* **Quota:** `ssh ada` → check quota banner; free space on `/share1` before big sync-back.
* **Stale tunnels:** if a local port is busy, pass `-p 9001` (both start & stop).