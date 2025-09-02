# Install Linux on a VM

The point of this exercise is to install a Linux-based OS on a Virtual Machine
(VM) dedicated to the assignments for this class.  Using a VM is necessary for
the following reasons.  First, the virtual network framework we will be using
makes configuration changes to your network interfaces, which you don't want to
affect other applications on a working system.  Second, running the framework
requires root (administrator) privileges, which you typically don't want to use
to run applications on a more general system.

VirtualBox and UTM/Qemu are the only VM platforms that have been tested.
Another VM platform might work but should be considered experimental.  Also, it
would require adapting the instructions below to your chosen platform.

Generally you might select almost any distribution of Linux.  However, for this
class I am asking that you use Debian because the framework has been tested in
this environment.

 - [VirtualBox 7.x for Windows, Linux, and MacOS (amd64 only)](#virtualbox-7x-for-windows-linux-and-macos-amd64-only)
 - [UTM/Qemu on MacOS (required for Apple Silicon (M1/M2/M3))](#utmqemu-on-macos-required-for-apple-silicon-m1m2m3)


# VirtualBox 7.x for Windows, Linux, and MacOS (amd64 only)

Please note that for Mac systems using Apple Silicon (M1/M2/M3), you should use the
UTM/Qemu instructions.

These instructions are for VirtualBox 7.x.  For versions other than 7.x, you
might need to adapt these instructions.

 1. Download and install
    [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

 2. Download the "netinst" (net install) image with amd64 architecture from
    [Debian](https://www.debian.org/releases/stable/debian-installer/).

 3. Start VirtualBox, and click "New" to create a new VM.

 4. At the "Virtual machine Name and Operating System" dialog, enter the
    following:

    a. Name: Give the VM a meaningful name (e.g., "cs460").

    b. Folder: Use the default folder value.

    c. ISO Image: Select the install image (`.iso` file) you downloaded in
       step 2.

    d. Check the box labeled "Skip Unattended Installation".

    Click "Next".

 5. At the "Hardware" dialog, enter the following:

    a. Base Memory: Give the machine 2GB (2048 MB) of RAM.

    b. Processors: Select 2 if you think your machine has at least four cores;
       select 1 otherwise.

    c. Enable EFI: Leave this default

    Click "Next".

 6. At the "Virtual Hard Disk" dialog, enter the following:

    a. "Create a virtual Hard Disk Now": 20 GB

    b. "Preallocate Full Size" - leave unchecked

    Click "Next".

 7. At the "Summary" dialog, click "Finish".

 8. In the VM, go through the Debian installation process.  If there are
    options for which you aren't sure what you enter, just use the default,
    noting the following exceptions:

    a. At the "Partition Disks" dialog, you will be asked if you want to write
       changes to disk.  You will need to change the answer from the default
       ("No") to "Yes".

    b. At the "Software selection" dialog, make sure the following boxes are
       checked:

       - "Debian desktop environment"
       - "LXDE"
       - "Standard System Utilities"

       Uncheck any other other options ("GNOME", "SSH server", etc.).  This is
       intended to be a lightweight system that provides the essentials for this
       class.  LXDE provides a lightweight desktop environment that demands less
       of your host system.

    c. If the "GRUB boot loader" dialog appears, select your hard disk to allow
       the installer to install GRUB to the hard drive.  It should start with
       "/dev" (probably "/dev/sda").

    Your VM should reboot when installation has finished.

 9. Log in to the VM after it reboots.  Use the regular account (i.e., non-root)
    username and password that you created during installation.

 10. Within the LXDE desktop environment in the VM, open a terminal
     (`LXTerminal`) and do the following to add your user to the `sudo` group:

     a. Run the following from the command line to temporarily become `root`
        (system administrator):

        ```
        su -
        ```

        When prompted, enter the password for the `root` user.

     b. At the `root` (`#`) prompt, run the following to add your user to the
        `sudo` group:

        (Replace "$USER" with the username of your regular, non-root user.

        ```
        usermod -a -G sudo $USER
        ```

     c. Log out of LXDE and log back in to make the group membership changes
        take effect.

     d. Test your `sudo` access by running the following:

        ```
        sudo ls -l /root/
        ```

        (This just lists the directory contents of `/root`, which is the
        `root` user's home directory.  A non-privileged user would not be able
        to see the contents, but by running `ls` with `sudo`, you should see
        the empty directory.)

     As a member of the `sudo` group, you will be able to run commands that
     require administrator privileges on a command-by-command basis using `sudo`,
     rather than being logged in as the `root` user, which is typically
     discouraged.

 11. Do the following to to install the VirtualBox Guest additions:

     a. On the host system, select "Devices" from the VirtualBox menu, then select
        "Insert Guest Additions CD Image...".

     b. Within the LXDE desktop environment in the VM, open a terminal
        (`LXTerminal`).

     c. From the open terminal, run the following to mount the "inserted"
        virtual CD containing the guest additions.

        ```
        mount /media/cdrom
        ```

     d. From the open terminal, run the following commands to build and install
        the VirtualBox Guest Additions for your VM:

        ```
        sudo apt install linux-headers-amd64 build-essential
        sudo sh /media/cdrom/VBoxLinuxAdditions.run
        ```

     e. Reboot your VM to have the changes take effect.

     f. Log in to the VM after it reboots.  Use the regular account (i.e.,
        non-root) username and password that you created during installation.

    This will allow you to do things like set up a shared drive between host
    and VM and use a shared clipboard, which will be accomplished in a
    subsequent step.

 12. On the host machine, do the following to set up a shared folder:

     a. From VirtualBox "Devices" menu, select "Shared Folders" then "Shared
        Folders Settings...".

     b. Click the button to add a shared folder.

     c. Choose which folder from the _host_ system you would like to share
        (e.g., `/Users/$HOSTUSER/shared`, where your actual username on the
        host OS replaces `$HOSTUSER`).

     d. Choose the location where the shared folder from the host will be
        mounted within the OS on the VM (e.g., `/home/$GUESTUSER/shared`,
        where your actual username on the VM replaces `$GUESTUSER`).

     e. Select both "Auto-mount" and "Make permanent".

     For more information, see the
     [official documentation](https://www.virtualbox.org/manual/ch04.html#sharedfolders).

 13. Within the VM, do the following to add your user to the `vboxsf`
     (VirtualBox shared folders) group:

     a. Run the following to add your user to the `vboxsf` group:

        ```
        sudo usermod -a -G vboxsf $USER
        ```

     b. Log out of LXDE and log back in to make the group membership changes
        take effect.

     As a member of the `vboxsf` group, you will be able to access the folder
     `/Users/$HOSTUSER/shared` (or whichever folder you selected) on the host
     from `/home/$GUESTUSER/shared` (or whichever mount point you selected) in
     the VM.

 14. On the host machine, do the following to enable copy and paste between
     your host and your VM.

     From the VirtualBox "Devices" menu, select "Shared Clipboard" then
     "Bidirectional".

 15. In the VM, open a terminal, and run the following to remove some
     unnecessary packages from your VM:

     ```
     sudo apt purge libreoffice-{impress,math,writer,draw,base-core,core,help-common,core-nogui} xscreensaver
     sudo apt autoremove
     ```

     This uninstalls LibreOffice, XScreenSaver, and any packages that are no
     longer needed because those two are removed.

 16. In the VM, disable the screen locker by doing the following:

     a. Select "Preferences" then "Desktop Session Settings" from the menu.

     b. Uncheck the box titled "Screen Locker," and click "OK".

     c. Log out of LXDE and log back in to make the changes take effect.

 18. In the VM, open a terminal, and run the following to install a few
     packages that will be useful for you in this class:

     ```
     sudo apt install git tmux vim build-essential make
     sudo apt install wireshark tcpdump iptables
     sudo apt install python3-{scapy,pip} virtualenv
     ```

     At the prompt "Should non-superusers be able to capture packets?" (for
     `wireshark`), select "No".  We will handle that with the next step.

     Next run the following to give `tcpdump`, `wireshark`, and `dumpcap`
     targeted capabilities, so an unprivileged user can run them to observe
     network packets without elevating to `root`:

     ```
     sudo setcap cap_net_raw=eip /usr/bin/tcpdump
     sudo setcap cap_net_raw=eip /usr/bin/wireshark
     sudo setcap cap_net_raw=eip /usr/bin/dumpcap
     ```

 19. Install whatever other tools and utilities that you think will improve your
     development environment.  Please note that if you have configured shared folders
     as described above, you can use whatever development environment you have already
     installed on your host to manipulate files in `/home/$USER/shared` or some
     subfolder thereof.  Thus, you do not have to develop within the VM itself if you
     do not want to.


# UTM/Qemu on MacOS (required for Apple Silicon (M1/M2/M3))

 1. Install [Homebrew](https://brew.sh/).

 2. Install qemu and utm by running the following from an open Terminal:

    ```bash
    brew install utm qemu
    ```

 3. Download the "netinst" (net install) image from
    [Debian](https://www.debian.org/releases/stable/debian-installer/).
    For M1/M2/M3 hardware, use the arm64 architecture.  For anything else, use
    amd64.

 4. Start UTM, then do the following:

    a. Click the "+" button to create a new virtual machine.

    b. Start: Click "Virtualize"

    c. Operating System: Click "Linux".

    d. Linux: Under "Boot ISO Image", click "Browse", then select the install
       image (`.iso` file) you downloaded.  Then click "Continue".

    e. Hardware: Specify at least 2048 MB RAM, then click "Continue".

    f. Storage: Specify at least 20GB, then click "Continue".

    g. Shared Directory: Select a folder from the _host_ system that you would
       like to share (e.g., `/Users/$HOSTUSER/shared`, where your actual
       username on the host OS replaces `$HOSTUSER`).  Then click "Continue".

    h. Summary: Give your VM a meaningful name.  Then click "Play".

 5. Start your VM by clicking the "play" icon next to its name.  It will boot
    from the installation image (`.iso` file) you downloaded.

 6. Follow steps 8 through 10 from the
    [VirtualBox instructions](#virtualbox-7x-for-windows-linux-and-macos-amd64-only),
    noting this important addition:

    At the very last dialog before the system reboots ("Finish the
    installation") "remove" the install CD by doing the following within the
    host system:

    a. Click the "Drive Image Options" button.

    b. Select "CD/DVD (ISO) Image".

    c. Click "Eject".

    Then select "Continue" to finish the installation and reboot.

 7. Within the VM, mount the shared directory by running the following from
    within a terminal:

    a. Create a mount point on the VM:

       ```bash
       sudo mkdir /media/shared
       ```

    b. Mount the shared volume as type `9p`:

       ```bash
       sudo mount -t 9p -o trans=virtio,version=9p2000.L share /media/shared/
       ```

    c. Change the permissions (from only the VM perspective) on files and
       directories in the shared directory, so your user (in the VM) can
       access the files.

       (Note: You can leave `$USER` as-is because the shell will automatically
       replace it with your username before the command is run.  You can see
       this by running `echo $USER`.)

       ```bash
       sudo chown -R $USER /media/shared/
       ```

    d. Test your new mount by listing directory contents:

       ```bash
       ls -l /media/shared
       ```

    e. Use the following command to ensure that the shared volume is
       mounted automatically at boot:

       ```
       sudo -e /etc/fstab
       ```

       (`sudo -e` safely opens a file that needs root privileges to edit.)

       Add the following line to the end of the file:

       ```
       share	/media/shared	9p	trans=virtio,version=9p2000.L,rw,_netdev,nofail	0	0
       ```

       Then close and save the file.

    f. Optionally create a symbolic link to the share mount from your home
       directory:

       ```bash
       ln -s /media/shared/ ~/shared
       ls -l ~/shared
       ls -l ~/shared/
       ```

       (Note that the last two commands are slightly different.  The first
       shows the symbolic link itself -- and what it points to.  The second
       shows the contents of the directory referred to by the symbolic link.)

 8. Follow steps 15 through 19 from the
     [VirtualBox instructions](#virtualbox-7x-for-windows-linux-and-macos-amd64-only).
