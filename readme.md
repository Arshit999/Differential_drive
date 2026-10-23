<p align="center">
  <img src="https://img.shields.io/badge/IMPORTANT-DOWNLOAD%20UBUNTU%2024.04%20ONLY-red?style=for-the-badge" alt="IMPORTANT: Download Ubuntu 24.04 only">
</p>

<h1 align="center">🚨 DOWNLOAD UBUNTU 24.04 ONLY 🚨</h1>

STRICT REQUIREMENT: Install Ubuntu 24.04 LTS Desktop using the download links in this README. Other Ubuntu release versions are not supported for this workshop. Do not download Ubuntu Server.

The download filenames include a point-release number (for example, 24.04.5). This is still part of the Ubuntu 24.04 LTS release family. Use the exact links below.

Ubuntu 24.04 LTS Desktop Setup Guide
This guide prepares your laptop for the robotics workshop and ROS 2 Jazzy activities. Follow the section for your computer's operating system.
Choose the correct download
Your computer	Use this image
Windows laptop with an Intel or AMD processor	AMD64 Ubuntu Desktop ISO
Apple Silicon Mac (M1, M2, M3, M4, or later)	ARM64 Ubuntu Desktop ISO
Intel-based Mac	Do not use the ARM64 image. Contact the organiser for guidance.


Both links below download Ubuntu Desktop, which includes the graphical user interface (GUI). Do not download the Server image.
- DOWNLOAD UBUNTU 24.04 LTS DESKTOP — AMD64 (Windows Intel/AMD laptops)
- DOWNLOAD UBUNTU 24.04 LTS DESKTOP — ARM64 (Apple Silicon Macs)
The ISO files are several gigabytes in size. Download them over a reliable internet connection and keep the file in an easy-to-find folder, such as Downloads. Do not extract the ISO file.
Part A — Windows: Install Ubuntu 24.04 using VirtualBox
These steps are for Windows laptops with an Intel or AMD processor. Ubuntu will run in a virtual machine, so you can use Ubuntu without replacing Windows or changing your Windows partitions.
Step 1 — Download the Ubuntu 24.04 AMD64 Desktop ISO
Click the link below and allow the download to finish:

Download Ubuntu 24.04 LTS Desktop — AMD64 (direct ISO download)

Check that the filename ends with desktop-amd64.iso. Do not download a file containing live-server in its name; the Server image does not include the standard desktop GUI by default.
Step 2 — Download and install VirtualBox
1. Open the official VirtualBox downloads page.
2. Under VirtualBox platform packages, click Windows hosts.
3. Open the downloaded installer and follow the setup prompts. Approve the driver/network prompts if Windows asks you to do so.
4. Restart Windows if the installer requests it, then open Oracle VirtualBox from the Start menu.
Step 3 — Create a new Ubuntu virtual machine
1. Open VirtualBox and click New.
2. Set the name to Ubuntu 24.04.
3. For ISO Image, browse to the Ubuntu ISO you downloaded in Step 1.
4. If the wizard offers Skip Unattended Installation, select it to use the standard Ubuntu installer and create your own username and password during installation.
5. Select Linux / Ubuntu 64-bit if VirtualBox asks for the operating system type.
6. Allocate resources. Recommended starting settings are:
   - RAM: 6–8 GB, provided your laptop has enough memory. Keep sufficient RAM for Windows.
   - Processors: 4 CPU cores, if available without allocating nearly all of your laptop's cores.
   - Virtual disk: 40 GB, preferably dynamically allocated.
7. Review the settings and finish creating the virtual machine.
If VirtualBox does not offer a 64-bit Ubuntu option or reports that hardware virtualisation is unavailable, check that virtualisation is enabled in your computer's BIOS/UEFI settings, or ask your device administrator for assistance.
Step 4 — Start the Ubuntu installer
1. Select the Ubuntu 24.04 virtual machine and click Start.
2. When the Ubuntu installer appears, select Install Ubuntu.
3. Choose your language and keyboard layout, then continue through the installer.
4. Select the standard/default installation options unless the workshop organiser has instructed otherwise.
5. When prompted, create your Ubuntu username and password. Keep these credentials available; you will need the password for commands using sudo.
6. If the installer asks where to install Ubuntu, make sure you are installing inside the VirtualBox virtual machine. The “virtual disk” is the disk created for this VM; it is not your normal Windows drive.
7. Continue with the installation and wait for it to finish.
Step 5 — Restart and open the Ubuntu GUI
1. Restart the virtual machine when the installer asks you to.
2. If it boots into the installer again, power off the VM and remove the ISO from Settings → Storage (or Devices → Optical Drives, depending on your VirtualBox version), then start the VM again.
3. Log in with the Ubuntu username and password you created.
The Ubuntu Desktop GUI should now appear. No separate GUI-enabling command is required because you installed the Desktop edition.
Step 6 — Update Ubuntu
Inside the Ubuntu virtual machine, open Terminal. You can use the keyboard shortcut Ctrl + Alt + T, then run:
sudo apt update && sudo apt upgrade -y
Enter your Ubuntu password when prompted. The password characters may not appear while you type; this is normal. If prompted to confirm a change, read the prompt before proceeding.

Optionally refresh installed Snap packages with:
sudo snap refresh
When the updates finish, restart Ubuntu if requested. Keep the VM installed for the workshop.
Part B — Mac: Install Ubuntu 24.04 using UTM
These instructions are intended for Apple Silicon Macs (M1, M2, M3, M4, and later). Use the ARM64 Desktop ISO. The provided video demonstrates the UTM setup process.
Step 1 — Download the Ubuntu 24.04 ARM64 Desktop ISO
DOWNLOAD UBUNTU 24.04 LTS DESKTOP — ARM64 (direct ISO download)

Wait for the download to finish. Do not extract the ISO.
Step 2 — Download UTM
Download UTM from its official website:

Download UTM for Mac

Install and open UTM. If macOS asks you to confirm opening the app, follow the standard macOS security prompts for the app you downloaded from the official website.
Step 3 — Follow the UTM installation video
Open the following video and follow its instructions to create and install the Ubuntu virtual machine:

Watch: Install UTM on Mac and set up Ubuntu 24.04

When selecting the Ubuntu installation image, choose the ARM64 Desktop ISO downloaded in Step 1. On an Apple Silicon Mac, use UTM's Virtualize option for Linux when the setup provides that choice; do not select Emulate for this ARM64 guest unless the video specifically explains a reason to do so.

Allocate enough memory and virtual storage for Ubuntu while leaving resources available for macOS. Follow the video for the remaining installer screens.
Step 4 — Open Ubuntu and update it
Once Ubuntu Desktop is installed, start the VM and log in. The graphical desktop is included automatically; no separate GUI-enabling command is needed.

Open Terminal inside Ubuntu and run:
sudo apt update && sudo apt upgrade -y
If prompted, enter the Ubuntu password created during installation. Restart the VM if requested.
Intel Mac users: The ARM64 image and Apple Silicon UTM steps above are not intended for Intel Macs. Please contact the workshop organiser before installing.

Before the workshop
- [ ] Ubuntu 24.04 LTS Desktop is installed and opens to the graphical desktop.
- [ ] Ubuntu has completed its initial updates.
- [ ] You remember your Ubuntu username and password.
- [ ] You can open Terminal and run a command.
- [ ] Your laptop has its charger available for the session.
Troubleshooting and official references
- Official Ubuntu 24.04 releases
- Official Ubuntu Desktop installation documentation
- Official Ubuntu guide: Run Ubuntu Desktop in VirtualBox
- Official VirtualBox downloads
- Official UTM website
Reminder: DOWNLOAD UBUNTU 24.04 LTS DESKTOP ONLY. Use the correct architecture for your computer and do not download Ubuntu Server.
