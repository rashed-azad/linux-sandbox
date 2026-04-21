# Sandbox Build Environment with schroot

A lightweight, isolated environment for building software without polluting your real system. Everything stays inside the chroot — easy to remove when done.

## Prerequisites

Install the two required host packages:

```bash
sudo apt install schroot debootstrap
```

## Step 1 — Create the chroot base

```bash
mkdir -p $HOME/Downloads/temp
sudo debootstrap bookworm $HOME/Downloads/temp/my-build
```

This downloads a minimal Debian bookworm (~300MB) into that folder.

## Step 2 — Configure schroot

```bash
sudo tee /etc/schroot/chroot.d/my-build.conf <<EOF
[my-build]
description=build environment
directory=/home/YOUR_USERNAME/Downloads/temp/my-build
users=YOUR_USERNAME
root-groups=root
type=directory
EOF
```

Replace `YOUR_USERNAME` with your actual username.

## Step 3 — Enter the chroot

As your user:
```bash
schroot -c my-build
```

As root (needed for apt):
```bash
schroot -c my-build -u root
```

Your prompt will change to `(my-build)` indicating you are inside the sandbox.

## Step 4 — Build inside the chroot

Your home directory is automatically mounted inside the chroot, so you can clone source code into `~/Downloads/temp/` and it will be visible from both inside and outside.

```bash
# Inside chroot as root
apt update
apt install <build-dependencies>

# Build your project
cd ~/Downloads/temp/my-project
meson setup build --prefix=/usr/local
ninja -C build
ninja -C build install
```

## Step 5 — Copy binaries to real system

After building, exit the chroot and copy only what you need:

```bash
exit
sudo cp ~/Downloads/temp/my-build/usr/local/bin/my-app /usr/local/bin/
```

## Cleanup

Remove everything when done:

```bash
# Remove the chroot
sudo rm -rf $HOME/Downloads/temp/my-build

# Remove the config
sudo rm /etc/schroot/chroot.d/my-build.conf

# Remove schroot and debootstrap
sudo apt remove --purge schroot debootstrap -y
sudo apt autoremove --purge -y
```

## Notes

- The chroot shares your home directory — source code cloned in `~/Downloads/temp/` is visible both inside and outside.
- Dependencies installed inside the chroot never touch your real system.
- Multiple chroots can coexist with different configs.
- Always enter as root (`-u root`) when running `apt` inside the chroot.