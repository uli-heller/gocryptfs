Uli's Notes
===========

Intro
-----

I'm not a GO developer and I do have some memory issues.
So I'll have to keep some
notes in case I have to rebase my work.

Updating The Repo
-----------------

Assuming we have the status of the previous release meaning
my changes are based on tag "v2.6.1", here are the steps to
rebase them onto "v2.7.0":

```
git fetch --all
# Tag v2.7.0 isn't fetched unfortunately

git fetch upstream --tags
# Tag v2.7.0 is fetched unfortunately

git rebase v2.7.0
# Resolve most conflicts, don't care too much about "go.sum"
```

File "go.sum"
-------------

Don't care too much on solving merge conflicts for it.

- Merge it "somehow"
- Re-create "go.sum" afterwards by:
  ```
  rm go.sum
  go mod download
  go mod tidy
  ```

Build And Test
--------------

### Build

Just run `./build.bash`.

### Basic Verification

```
$ ./gocryptfs --version
gocryptfs v2.7.0.17.qrcode; go-fuse v2.12.1-0.20261018182834-2d50fff207d7; 2026-11-15 go1.26.0 linux/amd64
```

### QRCode

Commands to execute:

```
SFX="$(date "+%Y%m%d%H%M%S")"
PASSWD="${SFX}"
mkdir "encrypted.${SFX}"
echo -n "${PASSWD}" >"passwd.${SFX}"
./gocryptfs --init --qrcode --passfile "passwd.${SFX}" "encrypted.${SFX}"
rm -rf "encrypted.${SFX}"
rm -rf "passwd.${SFX}"
```

Hopefully, you see a qr code and no error message!
Similar to [this page (https://uli.heller.cool/blog/2025-01-11_gocryptfs-qrcode/)](https://uli.heller.cool/blog/2025-01-11_gocryptfs-qrcode/)
or the output below:

```
Your master key is:

    9e4d188d-5058e80e-6e4501e3-bfd30236-
    352509d7-b9577742-8ff568a7-3a3f4bb0

QRCode:
█████████████████████████████████████████
█████████████████████████████████████████
████ ▄▄▄▄▄ ██▀█ ▄ █  ██  █▄ ▀█ ▄▄▄▄▄ ████
████ █   █ █▄▀▄ ▄█ ███▄▀▀█ ███ █   █ ████
████ █▄▄▄█ ██▄  ██  ▄▀▀  █   █ █▄▄▄█ ████
████▄▄▄▄▄▄▄█ █▄█▄▀▄█▄▀ █▄▀▄█▄█▄▄▄▄▄▄▄████
████ ▄ ▀▄▄▄▄ ▀██  ███  ██ ███▄ ▀█▀▄█▀████
████  ▄▀█ ▄█ ▀ ▀  █▀ ▀▄▀▄▀█▀▀▀▄▀  ▄▀▄████
████▄  ▀▀█▄▀█▀ ██  ██▄ █▄  █    █ ▀  ████
█████▀▀█▄▀▄ ██ █ ▀▀█▀▀ ▀▄  ▀█▄▄▄▄▀▄ ▄████
████▀█▀  ▄▄▄▀█ █   ██ ██▄  ██▀ ██ ▀▄ ████
████  ▄▀▄▀▄ ▄ ▀██ ██▀▀█▀▄▀▄▀▄▄█▀ █▄█▄████
████▄▄▄ ▄▀▄▄▄ ███▄ ██  █▄  ███  ███  ████
████▄▀ ▄▄█▄ █ ▀▀█▀▄█▀ ▀▀▄ ▄▀▀  ▄▄▀▄▄▄████
████▄██▄█▄▄█▀ ▄█  ███▄▄█▄  █ ▄▄▄ █▀█▀████
████ ▄▄▄▄▄ █ ▀▀██▀ ▀▀▀▀█▄    █▄█  ▄▀▄████
████ █   █ █▄  ██  ██ █ ▄  ▀▄▄▄▄ ▄█ █████
████ █▄▄▄█ █ ▀█▀▀ ▀▀█ ▄▀▀ █▄ ██▀▄ ▄ ▄████
████▄▄▄▄▄▄▄█▄▄▄▄▄▄▄▄█▄▄▄▄▄▄█▄▄▄█▄▄█▄▄████
█████████████████████████████████████████
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀


If the gocryptfs.conf file becomes corrupted or you ever forget your password,
there is only one hope for recovery: The master key. Print it to a piece of
paper and store it in a drawer. This message is only printed once.

The gocryptfs filesystem has been created successfully.
You can now mount it using: gocryptfs e MOUNTPOINT
```

Untested: Does the QR code match the masterkey?

### Multi Fido

Support for multiple fido devices depend on the ability
to change the password of an existing gocryptfs using
fido2 and the existing masterkey:

```
FIDO2_DEVICE="$(fido2-token -L|cut -d : -f 1|head -1)"
if [ -z "${FIDO2_DEVICE}" ]; then
  echo >&2 "No FIDO device found"
else
  SFX="$(date "+%Y%m%d%H%M%S")"
  PASSWD="${SFX}"
  mkdir "encrypted.${SFX}"
  echo -n "${PASSWD}" >"passwd.${SFX}"

  # Create a gocryptfs
  ./gocryptfs --init --passfile "passwd.${SFX}" "encrypted.${SFX}" >/dev/null 2>&2

  # Determine the masterkey
  echo -n "${PASSWD}"|./gocryptfs-xray/gocryptfs-xray --dumpmasterkey "encrypted.${SFX}/gocryptfs.conf" >"masterkey.${SFX}"

  # Create an additional config allowing the gocryptfs to be used with fid02
  cp "encrypted.${SFX}/gocryptfs.conf" "encrypted.${SFX}/gocryptfs-fido2.conf"
  # The options -passfile and -masterkey cannot be used at the same time
  echo -n "${PASSWD}"|./gocryptfs --passwd           \
    --masterkey "$(cat "masterkey.${SFX}")"          \
    --fido2 "${FIDO2_DEVICE}"                        \
    --config "encrypted.${SFX}/gocryptfs-fido2.conf" \
    "encrypted.${SFX}"

  # Cleanup
  rm -rf "encrypted.${SFX}"
  rm -rf "passwd.${SFX}"
  rm -rf "masterkey.${SFX}"
fi
```

You should see output like this:

```
Using config file at custom location /home/uli/git/forked/gocryptfs/encrypted.20261116102200/gocryptfs-fido2.conf
Using explicit master key.
THE MASTER KEY IS VISIBLE VIA "ps ax" AND MAY BE STORED IN YOUR SHELL HISTORY!
ONLY USE THIS MODE FOR EMERGENCIES
Please enter your new password.
Reading Password from stdin (connected to "pipe:[25056458]")
A copy of the old config file has been created at "/home/uli/git/forked/gocryptfs/encrypted.20261116102200/gocryptfs-fido2.conf.bak".
Delete it after you have verified that you can access your files with the new password.
Password changed.
```
