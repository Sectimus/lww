# ntfs-mount-manager

A Rust GUI tool for safely mounting and managing NTFS volumes on Linux, with Proton integration for gaming.

---

## What is this?

The main use case is simple: you want to play games through Proton on an NTFS drive, the same way you would on Windows, without juggling two separate game libraries or copying hundreds of gigabytes across filesystems. That should just work, and mostly it does — but the setup to get there safely is more involved than it should be.

This tool handles the NTFS side of that: configuring mounts correctly, checking volume health, managing the dirty bit, and making sure your fstab isn't going to cause problems. It also checks your installed Proton versions and lets you know if anything is out of date, so your games are running on the best available compatibility layer without you having to manually track releases.

---

## Background

The main motivation was wanting to run a Steam library on an NTFS drive through Proton without maintaining two separate copies of everything or reformatting drives that already had games on them from Windows.

This was developed and tested on Wayland + Arch + Nvidia + KDE — **W.A.N.K** — a combination with a well-earned reputation for being painful. If it holds up here, it'll hold up anywhere. And with this tool, the NTFS side of W.A.N.K is at least one less thing to worry about.

---

## Features

- Detects available NTFS partitions and lets you configure mount points through a GUI
- Runs a non-destructive pre-mount health check on every volume
- Distinguishes between a routine dirty bit and actual structural problems (MFT inconsistencies, bad journal state, suspected hibernation)
- Warns you about problems before mounting, with plain explanations of what they mean and what to do
- Prompts for a backup before touching anything if the volume looks questionable
- Writes fstab entries with safe, correct ntfs3 flags
- Lets you view and toggle mount flags with descriptions of what each one does
- Offers read-only mode if you don't need writes
- Never silently applies `force` — if you want to override a dirty bit check, you have to explicitly opt in and confirm
- Distro agnostic — no assumptions about your init system, desktop, or package manager

---

## Proton integration

Beyond NTFS configuration, the tool also:

- Detects installed Proton and Proton-GE versions across your Steam library locations
- Checks for available updates and flags anything out of date
- Shows which version each game is configured to use
- Highlights known compatibility improvements relevant to your installed titles where possible

The goal is to keep everything in one place — your drives are mounted correctly, your Proton versions are current, and you can just play.

---

## Safety checks

Before mounting or modifying anything, the tool inspects the volume and tells you what it found:

| What's detected | What happens |
|---|---|
| Only the dirty bit set | Offers to clear it and mount — explains this is normal |
| Dirty bit + log file needs reset | Offers to reset the log, explains Windows probably crashed |
| MFT inconsistencies | Blocks the mount, tells you to run `chkdsk` from Windows first |
| Hibernation state suspected | Hard warning, offers read-only only, explains the risk clearly |

If anything looks wrong, you'll be prompted to back up the volume before proceeding. The tool won't quietly do things to a volume that looks damaged.

---

## Why ntfs3?

`ntfs3` is the in-kernel NTFS driver that landed in Linux 5.15. It's faster than the older FUSE-based `ntfs-3g` and is actively maintained as part of the kernel. It also refuses to mount dirty volumes by default, which is actually the safer behaviour — `ntfs-3g` silently ignores the dirty bit, which is arguably worse.

This tool targets ntfs3 only.

---

## Building

### Requirements

- Rust (via [rustup](https://rustup.rs))
- Linux kernel 5.15+
- `ntfs-3g` userspace tools (`ntfsfix`, `ntfsinfo`, `ntfsck`)
- Root / sudo access

```bash
git clone https://github.com/yourusername/ntfs-mount-manager
cd ntfs-mount-manager
cargo build --release
sudo ./tar
get/release/ntfs-mount-manager
