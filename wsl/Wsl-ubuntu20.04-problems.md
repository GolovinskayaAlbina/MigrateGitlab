---
title: WSL2 Ubuntu 20.04 problems
weight: 1
---

Recent versions of Windows 10 now include Windows Subsystem for Linux (WSL) as an optional Windows feature. The WSL supports running a Linux environment within Windows.

{{% toc %}}

## Vagrant

Vagrant support for WSL is still in development and should be considered beta.

[Source](https://www.vagrantup.com/docs/other/wsl.html)

> Using Vagrant within the Windows Subsystem for Linux is an advanced topic that only experienced Vagrant users who are reasonably comfortable with Windows, WSL, and Linux should approach.

### Vagrant Installation

* Vagrant will detect when it is being run within the WSL and adjust how it locates and executes third party executables. For example, when using the VirtualBox provider Vagrant will interact with VirtualBox installed on the Windows system, not within the WSL.

* The docker daemon cannot be run inside the Windows Subsystem for Linux. However, the daemon can be run on Windows and accessed by Vagrant while running in the WSL.

## Ubuntu WSL peculiarities

* Technically, you're not running Linux. It may look like Linux and squeak like Tux, the Linux penguin; but it's not Linux. That's because the Ubuntu userspace is running not on a Linux kernel, but WSL. WSL provides the API hooks to look like Linux to Ubuntu and Linux applications, but it's not the same thing. This will become important as we go along

[Source](https://www.zdnet.com/article/how-to-run-run-the-native-ubuntu-desktop-on-windows-10/)

* Executing

```bash
uname -r
```

Output on **Hyper-V Ubuntu 20.04**

```bash
5.4.0-26-generic
```

Output on **WSL Ubuntu 20.04**

```bash
4.19.128-microsoft-standard
```

[Microsoft-standard kernel releases](https://docs.microsoft.com/ru-ru/windows/wsl/kernel-release-notes)
