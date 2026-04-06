Day 1: Install Linux VM & Learn Basic Commands
Goal: Set up a Linux development environment and become comfortable with the command line.

Activities:

Download and install VirtualBox or VMware (free options available)
Install a Linux distribution (Ubuntu 22.04 LTS recommended for beginners)
Learn essential commands: ls, cd, pwd, mkdir, touch, rm, cp, mv
Practice navigating the file system and creating directories
Understand the Linux directory structure (/home, /etc, /var, etc.)
Resources:

Linux Command Line Basics - Ubuntu Official Docs
Linux Filesystem Hierarchy

Steps to Download and install VirtualBox or VMware (free options available)
Both VirtualBox and VMware Workstation Player are free options you can use to run virtual machines on your computer. Here’s how you can get started with either one:

- Go to VMware’s site: https://www.vmware.com/products/workstation-player.html (vmware.com in Bing).
- Click Download Now.
- Select the version for your operating system (Windows or Linux).
- Run the installer and follow the setup instructions.
- Launch VMware Player and create a new virtual machine.



Directory	Purpose
/ (Root)	The top-level directory. Every other directory is a "child" of this one.
/bin	Essential command binaries that need to be available for all users (e.g., ls, cp).
/boot	Contains static files of the boot loader, including the Linux kernel.
/dev	Contains file representations of hardware components (e.g., hard drives, USBs).
/etc	Configuration central. This is where system-wide settings and script files live.
/home	Personal storage for users (e.g., /home/username). This is your playground.
/lib	Shared library files required for the binaries in /bin and /sbin to run.
/media	Mount point for removable media like USB drives or CD-ROMs.
/opt	"Optional" software. Usually used for manual installations of third-party apps.
/root	The home directory for the root user (the system administrator).
/sbin	System binaries used for system administration (e.g., iptables, reboot).
/tmp	Temporary files. Many systems clear this folder upon reboot.
/usr	User programs and data. Includes its own /usr/bin for non-essential apps.
/var	Variable data. This is where logs (/var/log) and databases are kept.
