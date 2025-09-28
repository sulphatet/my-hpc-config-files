# Step 1 — macOS SSH: keys + first login

Goal: set up passwordless SSH from your Mac to the Ada login node.

> Replace all `<ADA_USER>` with **your** cluster username.

## 1) Create an SSH key (if you don't have one)

```bash
ssh-keygen -t ed25519 -C "<ADA_USER>@ada" -f ~/.ssh/id_ed25519
# press enter for empty passphrase, or set one if you prefer
```

## 2) Upload your public key to Ada

If `ssh-copy-id` exists:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <ADA_USER>@ada.iiit.ac.in
```

If it doesn't, do it manually:

```bash
# print the key
cat ~/.ssh/id_ed25519.pub
# copy the whole line, then:
ssh <ADA_USER>@ada.iiit.ac.in
# on the remote (Ada):
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "<PASTE_PUBLIC_KEY_LINE_HERE>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

## 3) Test login

```bash
ssh <ADA_USER>@ada.iiit.ac.in
# you should get a shell on Ada without a password prompt
```

If you see a password prompt, re-check `authorized_keys` permissions and content.

**Next:** proceed to [Step 2 — SSH config](../step-02-ssh-config/README.md).