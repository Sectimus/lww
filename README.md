# LWW — Linux, Windows, Whenever

*aka "Linus/Luke Was Wrong"*

High performance NTFS setup for Linux, primarily for gaming. Built in Rust.

---

## What is this?

The common wisdom is that you shouldn't use NTFS on Linux for anything serious, and you definitely shouldn't try to run a Steam library off it. That advice is outdated. The `ntfs3` kernel driver is fast, stable, and perfectly capable of handling a gaming workload — it just needs to be configured correctly, which is fiddly enough that most people either get it wrong or give up.

LWW handles the setup for you. It gives you a GUI to configure your NTFS mounts safely, manages the dirty bit, writes correct fstab entries, runs health checks before touching anything, and keeps an eye on your Proton versions — so you can run your Windows game library through Proton on an NTFS drive without reformatting anything or maintaining two copies of the same games.

Linus was wrong. Luke was wrong. NTFS on Linux is fine.

---

## Background

This came out of about a year of daily use on Arch Linux with ntfs3, mounting Windows drives for gaming through Proton without a single instance of data loss or corruption. A lot of the horror stories you'll find online trace back to specific scenarios — Fast Startup, hibernation, wrong mount flags — rather than the driver itself being unreliable.

The setup this was developed and tested on is Wayland + Arch + Nvidia + KDE — **W.A.N.K** — a combination with a well-earned reputation for being painful for reasons well beyond NTFS. If it works reliably here, it'll work anywhere. And with LWW, the NTFS side of W.A.N.K is at least one less thing to worry about.

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

Beyond NTFS configuration, LWW also:

- Detects installed Proton and Proton-GE versions across your Steam library locations
- Checks for available updates and flags anything out of date
- Shows which Proton version each game is configured to use
- Highlights known compatibility improvements relevant to your installed titles where possible

The goal is to keep everything in one place — your drives are mounted correctly, your Proton versions are current, and you can just play.

---

## Safety checks

Before mounting or modifying anything, LWW inspects the volume and tells you what it found:

| What's detected | What happens |
|---|---|
| Only the dirty bit set | Offers to clear it and mount — explains this is normal |
| Dirty bit + log file needs reset | Offers to reset the log, explains Windows probably crashed |
| MFT inconsistencies | Blocks the mount, tells you to run `chkdsk` from Windows first |
| Hibernation state suspected | Hard warning, offers read-only only, explains the risk clearly |

If anything looks wrong, you'll be prompted to back up the volume before proceeding. LWW won't quietly do things to a volume that looks damaged.

---

## Why ntfs3?

`ntfs3` is the in-kernel NTFS driver that landed
