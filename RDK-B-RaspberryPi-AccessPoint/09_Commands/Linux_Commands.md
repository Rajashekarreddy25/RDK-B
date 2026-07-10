# Linux_Commands

## Introduction

Linux commands were extensively used throughout the development of the RDK-B Raspberry Pi Access Point project. They were essential for navigating the filesystem, editing configuration files, verifying generated images, managing services, troubleshooting networking, and validating the final system.

This document provides a complete reference to the Linux commands used during the project along with their purpose and practical examples.

---

# Linux Command Categories

```text
Linux Commands

        │

        ├────────► File Management

        ├────────► Directory Navigation

        ├────────► File Editing

        ├────────► File Searching

        ├────────► Mount Operations

        ├────────► Service Management

        ├────────► File Permissions

        ├────────► Process Monitoring

        └────────► System Information
```

---

# 1. pwd

Displays the current working directory.

```bash
pwd
```

Example Output:

```text
/home/scl/rdkb-pi/poky/build
```

Purpose:

- Verify current location.
- Avoid running commands in the wrong directory.

---

# 2. ls

Lists directory contents.

```bash
ls
```

Long format:

```bash
ls -l
```

Recursive listing:

```bash
ls -R
```

Purpose:

- View files.
- Verify generated images.
- Inspect directories.

---

# 3. cd

Change directory.

```bash
cd ~/rdkb-pi
```

Navigate to build directory:

```bash
cd poky/build
```

Navigate to deploy directory:

```bash
cd tmp/deploy/images/raspberrypi4-64
```

Purpose:

- Move between project directories.

---

# 4. mkdir

Create a directory.

```bash
mkdir verify-root
```

Create parent directories automatically:

```bash
mkdir -p /mnt/verify-root
```

Purpose:

- Create mount points.
- Organize project files.

---

# 5. cp

Copy files.

```bash
cp source destination
```

Project Example:

```bash
cp core-image-rdkb-ap-*.wic.bz2 verify.wic.bz2
```

Purpose:

- Preserve original image.
- Create verification copy.

---

# 6. mv

Move or rename files.

Rename:

```bash
mv old.txt new.txt
```

Move:

```bash
mv file.txt backup/
```

Purpose:

- Rename files.
- Organize project directories.

---

# 7. rm

Delete files.

```bash
rm filename
```

Delete recursively:

```bash
rm -rf directory
```

Purpose:

- Remove temporary files.
- Clean workspace.

---

# 8. nano

Edit text files.

```bash
nano filename
```

Project Examples:

```bash
nano conf/local.conf
```

```bash
nano core-image-rdkb-ap.bb
```

```bash
nano rdkb-network.sh
```

Purpose:

- Modify configuration files.
- Edit recipes.
- Update scripts.

---

# 9. cat

Display file contents.

```bash
cat filename
```

Project Example:

```bash
cat /mnt/verify-root/usr/bin/rdkb-network.sh
```

Purpose:

- Verify installed files.
- Inspect configurations.

---

# 10. less

View large files.

```bash
less filename
```

Purpose:

- Read logs.
- Inspect long configuration files.

Exit:

```text
q
```

---

# 11. grep

Search text.

```bash
grep "iptables" filename
```

Project Example:

```bash
grep -n "iptables" core-image-rdkb-ap.bb
```

Purpose:

- Search recipes.
- Locate configuration entries.

---

# 12. find

Locate files.

```bash
find directory -name filename
```

Project Examples:

```bash
find /mnt/verify-root -name iptables
```

```bash
find /mnt/verify-root -name nft
```

Purpose:

- Verify installed packages.
- Search generated images.

---

# 13. chmod

Change permissions.

```bash
chmod +x filename
```

Example:

```bash
chmod +x rdkb-network.sh
```

Purpose:

- Make scripts executable.

---

# 14. chown

Change file ownership.

```bash
sudo chown user:user filename
```

Purpose:

- Correct file ownership.

---

# 15. mount

Mount a filesystem.

Project Example:

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Purpose:

- Mount generated image.
- Verify root filesystem.

---

# 16. umount

Unmount filesystem.

```bash
sudo umount /mnt/verify-root
```

Purpose:

- Safely remove mounted image.

---

# 17. losetup

Attach loop device.

Project Example:

```bash
sudo losetup -Pf --show verify.wic
```

Example Output:

```text
/dev/loop28
```

Purpose:

- Access partitions inside image files.

---

# 18. sync

Flush filesystem buffers.

```bash
sync
```

Purpose:

- Ensure all data is written before removing storage.

---

# 19. df

Display filesystem usage.

```bash
df -h
```

Purpose:

- Check available disk space.

---

# 20. du

Display directory size.

```bash
du -sh directory
```

Purpose:

- Estimate build storage usage.

---

# 21. uname

Display system information.

```bash
uname -a
```

Purpose:

- Verify Linux kernel version.

---

# 22. hostname

Display hostname.

```bash
hostname
```

Purpose:

- Identify current machine.

---

# 23. whoami

Display current user.

```bash
whoami
```

Purpose:

- Confirm logged-in user.

---

# 24. history

Display previously executed commands.

```bash
history
```

Purpose:

- Recall project commands.
- Reuse previous commands.

---

# 25. clear

Clear terminal screen.

```bash
clear
```

Shortcut:

```text
Ctrl + L
```

Purpose:

- Improve terminal readability.

---

# 26. echo

Print text.

```bash
echo Hello
```

Project Example:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Purpose:

- Enable IPv4 forwarding.
- Write configuration values.

---

# 27. tar

Extract archive.

```bash
tar -xvf archive.tar
```

Purpose:

- Extract downloaded packages.

---

# 28. bunzip2

Extract compressed image.

Project Example:

```bash
bunzip2 verify.wic.bz2
```

Purpose:

- Decompress Yocto image.

---

# 29. file

Identify file type.

```bash
file verify.wic
```

Purpose:

- Confirm image format.

---

# 30. stat

Display file information.

```bash
stat filename
```

Purpose:

- Verify timestamps.
- Confirm file updates.

---

# Linux Commands Used During This Project

The following commands were used frequently throughout the project:

Navigate directories:

```bash
cd ~/rdkb-pi/poky/build
```

Edit files:

```bash
nano rdkb-network.sh
```

View files:

```bash
cat hostapd.conf
```

Search files:

```bash
find /mnt/verify-root -name iptables
```

Mount image:

```bash
sudo mount /dev/loop28p2 /mnt/verify-root
```

Attach loop device:

```bash
sudo losetup -Pf --show verify.wic
```

Search recipe:

```bash
grep -n "iptables" core-image-rdkb-ap.bb
```

Create directory:

```bash
mkdir -p /mnt/verify-root
```

Extract image:

```bash
bunzip2 verify.wic.bz2
```

Copy image:

```bash
cp core-image-rdkb-ap-*.wic.bz2 verify.wic.bz2
```

---

# Best Practices

- Always verify your current directory before executing commands.
- Mount generated images in read-only mode when possible.
- Use `find` instead of manually searching large filesystems.
- Verify file contents using `cat` after making changes.
- Use `mkdir -p` to safely create nested directories.
- Unmount filesystems before disconnecting storage devices.
- Keep a copy of the original image before modifying it.

---

# Lessons Learned

Working extensively with Linux commands during this project reinforced several key practices:

- Strong command-line skills significantly improve development efficiency.
- Image verification using `losetup` and `mount` is faster than repeatedly flashing an SD card.
- Commands like `find`, `grep`, and `cat` are invaluable for debugging embedded Linux systems.
- File permissions must be handled carefully, especially for executable scripts.
- Understanding Linux filesystem structure simplifies troubleshooting and project maintenance.

---

# Conclusion

Linux command-line tools formed the foundation of the RDK-B Raspberry Pi Access Point development workflow. From editing recipes and configuration files to verifying generated images and managing the filesystem, these commands enabled efficient development, debugging, and validation. Mastering these essential Linux utilities greatly improved productivity and contributed to the successful completion of the project.