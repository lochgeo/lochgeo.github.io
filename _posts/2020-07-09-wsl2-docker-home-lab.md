---
layout: post
title:  "WSL 2 - Finally a Real Linux Kernel on My Windows Laptop"
date:   2020-07-09 20:14:00
categories: [Software, Containers]
excerpt_separator: "<!--more-->"
---

Four months into full-time work from home, my living room table is the office, the standup room, and the lab. The corporate laptop still runs Windows because that is what Visual Studio, Outlook, and the VPN client expect. The services I actually ship, though, are Linux containers. For years that meant Hyper-V VMs, slow bind mounts, and a quiet acceptance that "it works on my machine" was a slightly different machine than production.

Windows 10 version 2004 shipped WSL 2 into general availability earlier this year, and Docker Desktop's stable channel caught up with a WSL 2 backend in May. I spent a weekend moving my local .NET Core container workflow onto it. Not because I needed another toy — because the old setup was eating evenings I no longer have spare.

<!--more-->

### What was wrong with the old way

I was already on Docker Desktop with the Hyper-V backend. It worked. It also felt like renting a second computer inside the first one.

- Cold start of the Docker VM was slow enough that I delayed "just spin up the dependency stack" until I was already blocked.
- File sharing from `C:\Users\...` into Linux containers was the classic pain: `dotnet watch`, npm, and anything chatty on the filesystem crawled.
- Memory sat reserved even when I was only editing C# and not running a single container.
- On a laptop with 16 GB RAM, that reserved chunk competed with Teams, Chrome, and Visual Studio at the same time — which is basically every weekday now.

WSL 1 never fixed this for me. System-call translation was clever, but Docker Engine wants a real Linux kernel. Full stop. WSL 2 is the first Microsoft answer that does not feel like a compromise dressed up as a feature.

### What WSL 2 actually changes

WSL 2 runs a real Linux kernel in a lightweight VM managed by Windows. You still open a terminal that feels local. Underneath, you get full system-call compatibility — which is why Docker can finally run *as Docker*, not as a separate Hyper-V appliance fighting for resources beside it.

A few practical differences I noticed in the first week:

- **Dynamic memory.** The WSL 2 VM grows and shrinks. When I stop containers, RAM comes back to Windows instead of sitting locked in a fixed Hyper-V allocation.
- **Filesystem performance inside the distro.** Clone the repo into the Linux filesystem (`~/src/...`), not under `/mnt/c/...`. Cross-OS file access is still the slow path. Same-OS is the point of the upgrade.
- **Faster cold start.** Docker Desktop with the WSL 2 backend comes up in seconds on my machine, not "go make tea" territory.
- **Home edition friendly.** Hyper-V was a Pro-feature tax. WSL 2 lands on Home too, which matters if you are learning on a personal laptop after hours.

Microsoft's own numbers from the WSL team talked about multi-x improvements on file-heavy workloads versus WSL 1. I did not benchmark carefully. I just noticed `dotnet restore` and image builds stopped being the part of the evening I dreaded.

### How I wired Docker Desktop

The path that worked for me, after one false start:

1. Confirm Windows 10 is on version 2004 (or later). `winver` is enough.
2. Enable **Virtual Machine Platform** and **Windows Subsystem for Linux**, then reboot.
3. Install the WSL 2 Linux kernel update package from Microsoft, then `wsl --set-default-version 2`.
4. Install a distro from the Store — I use Ubuntu — and convert it if it came up as version 1: `wsl --set-version Ubuntu 2`.
5. Install current Docker Desktop stable, open Settings, and turn on the WSL 2 based engine. Under Resources → WSL Integration, enable your distro.

After that, `docker version` from the Ubuntu shell talks to the same engine as Docker Desktop on the Windows side. I keep the CLI habit inside WSL and still use Visual Studio on Windows against published ports. That split is fine. Fighting for one pure environment is how weekends disappear.

### Habits that saved me from myself

A few things I wish I had done on day one:

- **Keep source in the Linux filesystem.** I moved active repos to `\\wsl$\Ubuntu\home\...` (or just worked in the Ubuntu terminal). Editing from Windows via the `\\wsl$` path is acceptable; bind-mounting huge trees from `C:\` into containers is not.
- **Do not double-install Docker.** Either Docker Desktop's WSL backend *or* Docker Engine inside the distro. Both at once is how you get mysterious daemon fights and iptables confusion.
- **VPN still comes first.** Corporate VPN clients and WSL 2 networking can be grumpy together. If NuGet or an internal registry only exists behind the tunnel, sort that before blaming Docker. Half of my "WSL is broken" moments were "VPN dropped again."
- **Cap what you run locally.** Just because the stack *can* be twelve containers does not mean your evening needs all twelve. I run the two dependencies the bug actually touches and mock the rest. Home broadband and 16 GB RAM are honest constraints.

### Why this matters more in 2020 than it did last year

In the office I could shrug and use a shared dev environment, or walk over to someone with a Linux box. At home, my laptop *is* the environment. Video calls already tax the machine. A container toolchain that idles politely and builds quickly is not a luxury metric — it is how you still have energy left for the actual bug after the third Teams meeting.

I am still a .NET person on Windows. That has not changed. What changed is that "I need Linux to try this properly" no longer means dual boot, a second laptop, or a cloud VM I forget to turn off. WSL 2 plus Docker Desktop is the first combo that feels like one workstation instead of two operating systems glued together with hope.

If you are on Windows 10 2004, still on the old Hyper-V Docker backend, and your evenings involve containers: spend one weekend moving over. Put the repo in the Linux filesystem, flip the Docker setting, and run your real compose file — not a hello-world. That is the test. For me, the lab finally fits on the same desk as the day job, and that is enough to keep tinkering after the laptop lid should have closed.
