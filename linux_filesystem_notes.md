# Linux File System — Detailed Notes

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Everything is a File](#2-everything-is-a-file)
3. [Linux Directory Structure (FHS)](#3-linux-directory-structure-fhs)
4. [File Types in Linux](#4-file-types-in-linux)
5. [Inodes and Metadata](#5-inodes-and-metadata)
6. [File Permissions](#6-file-permissions)
7. [Hard Links vs Soft Links](#7-hard-links-vs-soft-links)
8. [Common File Systems](#8-common-file-systems)
9. [Mounting and Unmounting](#9-mounting-and-unmounting)
10. [Virtual File Systems (VFS)](#10-virtual-file-systems-vfs)
11. [Special File Systems](#11-special-file-systems)
12. [Disk Partitioning](#12-disk-partitioning)
13. [Useful Commands](#13-useful-commands)
14. [Summary Diagram](#14-summary-diagram)

---

## 1. Introduction

The **Linux File System** is a structured way of organizing and storing data on a storage device. Unlike Windows (which uses drive letters like `C:\`, `D:\`), Linux uses a **single unified directory tree** rooted at `/` (the root directory).

All files, directories, devices, and processes are represented within this single tree, regardless of how many physical drives are attached.

**Key Characteristics:**
- Case-sensitive (`file.txt` ≠ `File.txt`)
- Everything starts at `/` (root)
- No drive letters — devices are mounted as directories
- Designed for multi-user environments
- Supports a wide variety of file system formats

---

## 2. Everything is a File

One of Linux's core philosophies is:

> **"Everything is a file"**

This means:
| Resource | Represented As |
|----------|---------------|
| Regular text or binary data | Regular file |
| Directories | Special file containing filenames |
| Hardware devices (disk, USB) | Device files in `/dev` |
| Network sockets | Socket files |
| Running processes | Entries in `/proc` |
| Kernel parameters | Entries in `/sys` |

This abstraction allows uniform access to all system resources through standard file I/O operations (`read`, `write`, `open`, `close`).

---

## 3. Linux Directory Structure (FHS)

The **Filesystem Hierarchy Standard (FHS)** defines the standard directory layout. Here is a full breakdown:

```
/
├── bin/       → Essential user binaries
├── boot/      → Boot loader files, kernel
├── dev/       → Device files
├── etc/       → System configuration files
├── home/      → Home directories for users
├── lib/       → Shared libraries for /bin and /sbin
├── lib64/     → 64-bit shared libraries
├── media/     → Mount point for removable media
├── mnt/       → Temporary mount points
├── opt/       → Optional/third-party software
├── proc/      → Virtual filesystem for process info
├── root/      → Home directory of root user
├── run/       → Runtime data (PIDs, sockets)
├── sbin/      → System binaries (admin use)
├── srv/       → Data for services (HTTP, FTP)
├── sys/       → Virtual filesystem for kernel/device info
├── tmp/       → Temporary files (cleared on reboot)
├── usr/       → Secondary hierarchy (user programs)
│   ├── bin/   → Non-essential user commands
│   ├── lib/   → Libraries for /usr/bin
│   ├── local/ → Locally installed software
│   └── share/ → Architecture-independent data
└── var/       → Variable data (logs, caches, spool)
    ├── log/   → System log files
    ├── cache/ → Application cache data
    └── spool/ → Mail/print queue
```

### Key Directories Explained

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the entire filesystem tree |
| `/bin` | Core commands like `ls`, `cp`, `mv`, `bash` |
| `/boot` | Kernel (`vmlinuz`), initrd, GRUB config |
| `/dev` | Device nodes — `/dev/sda` (disk), `/dev/null`, `/dev/tty` |
| `/etc` | Config files — `/etc/passwd`, `/etc/fstab`, `/etc/hosts` |
| `/home` | User personal files — `/home/alice`, `/home/bob` |
| `/proc` | Live kernel and process data in virtual files |
| `/tmp` | Scratch space; world-writable, auto-cleared |
| `/usr` | Read-only user data, programs, documentation |
| `/var` | Grows over time — logs, databases, spool files |

---

## 4. File Types in Linux

Linux recognizes **7 file types**, identified by the first character in `ls -l` output:

| Symbol | Type | Example |
|--------|------|---------|
| `-` | Regular file | `/etc/hosts` |
| `d` | Directory | `/home/user/` |
| `l` | Symbolic (soft) link | `/usr/bin/python → python3` |
| `b` | Block device | `/dev/sda` |
| `c` | Character device | `/dev/tty`, `/dev/null` |
| `s` | Socket | `/run/docker.sock` |
| `p` | Named pipe (FIFO) | Used for IPC |

```bash
# Identify file type
file /dev/sda        # block special
file /etc/passwd     # ASCII text
file /run/docker.sock # socket
```

---

## 5. Inodes and Metadata

### What is an Inode?

An **inode** (index node) is a data structure that stores metadata about a file. Every file has exactly one inode, identified by an **inode number**.

### Inode Stores:
- File type and permissions
- Owner (UID) and group (GID)
- File size
- Timestamps: `atime` (access), `mtime` (modify), `ctime` (change)
- Number of hard links
- Pointers to data blocks on disk

### What Inode Does NOT Store:
- The **filename** (stored in the directory entry)
- The file's actual content (stored in data blocks)

```bash
# View inode number
ls -i /etc/passwd         # e.g., 131073 /etc/passwd

# View full inode info
stat /etc/passwd
```

### Inode Table

| Field | Description |
|-------|-------------|
| Inode number | Unique identifier within the filesystem |
| Mode | File type + permission bits |
| UID / GID | Ownership |
| Size | In bytes |
| Block count | Number of 512-byte blocks used |
| Timestamps | atime, mtime, ctime |
| Data blocks | Pointers (direct, indirect, double-indirect) |

---

## 6. File Permissions

### Permission Model

Linux uses a **9-bit permission model** (plus special bits), visible in `ls -l`:

```
-rwxr-xr--  1  alice  staff  4096  Jun 21  file.sh
│││││││││
│├─┤├─┤└─┘
│ │  │   └── Other permissions
│ │  └────── Group permissions
│ └───────── Owner permissions
└─────────── File type
```

### Permission Bits

| Symbol | Octal | Meaning |
|--------|-------|---------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

### Example

```
rwxr-xr--  =  7 5 4  (owner=7, group=5, others=4)
```

### Changing Permissions

```bash
chmod 755 script.sh        # rwxr-xr-x
chmod u+x script.sh        # add execute for owner
chmod go-w file.txt        # remove write from group and others
```

### Ownership

```bash
chown alice file.txt           # change owner
chown alice:devs file.txt      # change owner and group
chgrp devs file.txt            # change group only
```

### Special Permission Bits

| Bit | Name | Effect |
|-----|------|--------|
| `4000` | SUID | Run file as owner (e.g., `passwd`) |
| `2000` | SGID | Run as group; inherit group in dirs |
| `1000` | Sticky | Only owner can delete (e.g., `/tmp`) |

```bash
chmod u+s /usr/bin/program    # Set SUID
chmod g+s /shared/dir         # Set SGID
chmod +t /tmp                 # Set sticky bit
```

### umask

`umask` defines default permission bits **subtracted** when new files are created.

```bash
umask          # View current umask (e.g., 0022)
umask 027      # New files: 640, new dirs: 750
```

---

## 7. Hard Links vs Soft Links

### Hard Links

- A directory entry that points directly to an **inode**
- Multiple filenames → same inode → same data
- Cannot cross filesystem boundaries
- Cannot link directories

```bash
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt    # Same inode number
```

### Soft (Symbolic) Links

- A special file that contains a **path string** to another file
- Can cross filesystem boundaries
- Can link directories
- Broken if the target is deleted

```bash
ln -s /path/to/original symlink.txt
ls -l symlink.txt     # Shows → /path/to/original
```

### Comparison

| Feature | Hard Link | Soft Link |
|---------|-----------|-----------|
| Inode | Same as original | New inode |
| Cross-filesystem | ✗ No | ✓ Yes |
| Link directories | ✗ No | ✓ Yes |
| Survives deletion | ✓ Yes (data persists) | ✗ No (becomes broken) |
| Identified in `ls` | No indicator | `l` type, shows `→` |

---

## 8. Common File Systems

Linux supports many filesystem formats:

| Filesystem | Description |
|------------|-------------|
| **ext4** | Default for most Linux distros. Journaling, up to 1 EiB volumes |
| **ext3** | Older journaling FS (ext2 + journal) |
| **ext2** | Classic, no journaling |
| **XFS** | High-performance, scalable. Default on RHEL/CentOS |
| **Btrfs** | Modern CoW FS with snapshots, RAID, checksums |
| **ZFS** | Enterprise-grade, CoW, snapshots, self-healing |
| **NTFS** | Windows FS, readable/writable via `ntfs-3g` |
| **FAT32/exFAT** | Portable; used for USB drives |
| **tmpfs** | RAM-based; used for `/tmp`, `/run` |
| **NFS** | Network File System for shared storage |

### ext4 Features
- **Journaling** — recovers from crashes by replaying a journal
- **Extents** — contiguous block allocation reduces fragmentation
- **Delayed allocation** — improves write performance
- **64-bit block numbers** — up to 1 EiB volume size

---

## 9. Mounting and Unmounting

In Linux, all storage devices must be **mounted** to a directory (mount point) before they can be accessed.

### Manual Mounting

```bash
mount /dev/sdb1 /mnt/usb          # Mount USB to /mnt/usb
mount -t ext4 /dev/sda1 /mnt/disk # Specify filesystem type
mount -o ro /dev/sdb1 /mnt/usb    # Mount read-only
```

### Unmounting

```bash
umount /mnt/usb         # Unmount by mount point
umount /dev/sdb1        # Unmount by device
```

### /etc/fstab — Persistent Mounts

The file `/etc/fstab` defines filesystems to mount at boot:

```
# <device>         <mount point>  <type>  <options>        <dump>  <pass>
UUID=abc123        /              ext4    defaults          1       1
UUID=def456        /home          ext4    defaults          1       2
UUID=789ghi        /boot/efi      vfat    umask=0077        0       1
tmpfs              /tmp           tmpfs   defaults,noatime  0       0
```

```bash
mount -a        # Mount all entries in /etc/fstab
```

### Viewing Mounts

```bash
mount                  # List all mounts
df -h                  # Disk usage with mount points
lsblk                  # Block devices and mount points
findmnt                # Tree view of mounts
```

---

## 10. Virtual File Systems (VFS)

The **Virtual File System (VFS)** is a kernel abstraction layer that provides a **uniform interface** for all filesystems.

```
User Space
  │  open(), read(), write(), close()
  ▼
┌────────────────────────────────────────┐
│           VFS Layer (Kernel)           │
│   inode, dentry, superblock, file ops  │
└──────┬─────────┬──────────────┬────────┘
       │         │              │
     ext4       xfs           tmpfs   ← Filesystem Drivers
       │         │              │
    Disk      Disk          RAM
```

VFS defines four key objects:
- **Superblock** — filesystem metadata (size, type, block size)
- **Inode** — file metadata
- **Dentry** — maps filename → inode (directory entry cache)
- **File** — represents an open file in a process

---

## 11. Special File Systems

### /proc — Process Information

A **virtual filesystem** exposing kernel and process data as files:

```bash
cat /proc/cpuinfo          # CPU info
cat /proc/meminfo          # Memory stats
cat /proc/uptime           # System uptime
ls /proc/1234/             # Info about PID 1234
cat /proc/1234/status      # Process status
cat /proc/1234/maps        # Memory maps
```

### /sys — Kernel and Device Info

Exposes kernel objects, devices, and drivers:

```bash
cat /sys/class/net/eth0/speed         # NIC speed
cat /sys/block/sda/queue/rotational   # Is disk an SSD?
ls /sys/bus/usb/devices/              # USB devices
```

### /dev — Device Files

Contains device nodes:

```bash
/dev/sda      # First SATA/SCSI disk
/dev/sda1     # First partition of sda
/dev/nvme0n1  # First NVMe disk
/dev/null     # Discard all input (black hole)
/dev/zero     # Returns endless null bytes
/dev/random   # Random number generator
/dev/tty      # Current terminal
```

### tmpfs — RAM-backed Storage

```bash
mount -t tmpfs -o size=512m tmpfs /mnt/ramdisk
```

Used for `/tmp`, `/run`, and `/dev/shm` — fast but non-persistent.

---

## 12. Disk Partitioning

### Partition Tables

| Type | Description |
|------|-------------|
| **MBR** | Legacy, max 4 primary partitions, max 2 TB disk |
| **GPT** | Modern, up to 128 partitions, supports 9.4 ZB disks |

### Partitioning Tools

```bash
fdisk /dev/sda       # MBR partition tool (interactive)
gdisk /dev/sda       # GPT partition tool
parted /dev/sda      # Supports both MBR and GPT
lsblk                # View block devices and partitions
blkid                # Show UUIDs and filesystem types
```

### Creating a Filesystem

```bash
mkfs.ext4 /dev/sdb1          # Format as ext4
mkfs.xfs /dev/sdb2           # Format as XFS
mkfs.vfat /dev/sdb3          # Format as FAT32
mkswap /dev/sdb4             # Create swap space
```

### Swap Space

Swap is used when RAM is full:

```bash
swapon /dev/sda4           # Enable swap partition
swapoff /dev/sda4          # Disable swap
free -h                    # View RAM and swap usage
swapon --show              # Show active swap
```

---

## 13. Useful Commands

### Navigating & Listing

```bash
pwd                    # Print working directory
ls -la                 # List all files with details
ls -lh                 # Human-readable sizes
tree /etc              # Directory tree
cd -                   # Go to previous directory
```

### File Operations

```bash
cp -r src/ dest/       # Copy directory recursively
mv old.txt new.txt     # Move/rename
rm -rf dir/            # Remove directory recursively
touch file.txt         # Create empty file / update timestamp
```

### Disk Usage

```bash
df -h                  # Filesystem usage (human-readable)
du -sh /var/log        # Size of directory
du -ah /home | sort -rh | head -20   # Top 20 largest items
ncdu /                 # Interactive disk usage explorer
```

### Finding Files

```bash
find / -name "*.log" -type f         # Find by name
find /var -size +100M                # Find files > 100 MB
find / -perm /4000                   # Find SUID files
locate passwd                        # Fast search (uses index)
which python3                        # Find binary in PATH
```

### Viewing File Info

```bash
stat file.txt           # Full metadata
file file.txt           # File type detection
ls -i file.txt          # Show inode number
lsof /var/log/syslog    # Which processes have file open
```

### Disk and Partition Info

```bash
lsblk -f               # Block devices with filesystem info
blkid                  # UUIDs and types
fdisk -l               # Partition table listing
parted -l              # Partition details
```

---

## 14. Summary Diagram

```
                    Linux File System Tree
                           /
          ┌──────┬──────┬──┴──┬──────┬──────┐
         bin    etc   home   dev   proc   usr
          │      │      │     │      │      │
       ls,cp  config  users /dev  kernel  /usr/bin
              files   dirs  nodes  info   programs


   Physical Layout:
   ┌─────────────────────────────────────────────┐
   │  Storage Device (e.g., /dev/sda)            │
   │  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │
   │  │ Partition│ │ Partition│ │  Partition  │ │
   │  │  sda1    │ │  sda2    │ │    sda3     │ │
   │  │  (ext4)  │ │  (swap)  │ │   (xfs)     │ │
   │  │  /       │ │          │ │   /home      │ │
   │  └──────────┘ └──────────┘ └─────────────┘ │
   └─────────────────────────────────────────────┘

   Inode Structure:
   ┌──────────────────────────────────────┐
   │  Inode #1042                         │
   │  Type: Regular file                  │
   │  Permissions: rw-r--r-- (644)        │
   │  Owner: root / root                  │
   │  Size: 2048 bytes                    │
   │  atime: 2026-06-21 10:00            │
   │  mtime: 2026-06-20 18:30            │
   │  Data blocks: [block_1, block_2...]  │
   └──────────────────────────────────────┘
```

---

## Quick Reference Card

| Concept | Command / Info |
|---------|---------------|
| Root directory | `/` |
| List files with details | `ls -la` |
| File metadata | `stat <file>` |
| Inode number | `ls -i <file>` |
| Disk usage | `df -h` |
| Directory size | `du -sh <dir>` |
| Mount device | `mount /dev/sdX /mnt/point` |
| Persistent mounts | `/etc/fstab` |
| Change permissions | `chmod 755 <file>` |
| Change ownership | `chown user:group <file>` |
| Create filesystem | `mkfs.ext4 /dev/sdX` |
| Partition tool | `fdisk`, `gdisk`, `parted` |
| Find files | `find / -name "*.txt"` |
| File type | `file <name>` |
| Open files | `lsof` |

---

*Notes compiled for Linux System Administration reference.*
*Applies to: Ubuntu, Debian, RHEL, Fedora, Arch, and most Linux distributions.*
