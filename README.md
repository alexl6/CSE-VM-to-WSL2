# Introduction
This is a guide on how to convert a UW CSE Linux Home VM into a WSL2 distro.

The user can expect:
- Fast bootup (WSL2 distros can boot in seconds)
- OS integration (File sharing between Windows & Linux, VS Code remote development, etc.)
- GUI appplication suport (e.g. VLC, Firefox, Eclipse, etc.)


I took CSE 351 & 391 in Winter 2022 and was able to get consisitent result on all homeworks, labs, and midterm with Attu using the WSL2 distro described here. 

---
**NOTE: WSL2 is NOT supported by CSE as of Winter 2022.**

**It is strongly recommended that you test your assignments/programs on a CSE Linux VM or Attu before submitting them.**

---

While the setup worked flawlessly for me, your milage may vary. I am not responsible for point deductions due to inconsistencies between the WSL2 distro and CSE environment (CSE Linux Home VM or Attu).

# Compatibility

## No issues found
- Emacs
- Vim
- gdb
- gcc
- objdump
- Git
- Java
- Bash shell & CLI tools (grep, xargs, sed, etc.)
- Firefox browser
- VLC

## Works, but needs additional configuration
- Licensed software that checks MAC address (Was able to work around this by binding a static MAC address to WSL2)

## Known issues
- USB device support (The workarounds didn't work for me)
- No systemd
- GUI scaling maybe inconsistent when using multiple monitors with different scaling factors


# Steps
1. Follow the instruction on CSE website to set up your VM

    [CSE Linux Home VM](https://www.cs.washington.edu/lab/software/linuxhomevm)

2. Backup the Linux VM to a tar file

    Open a new terminal window in your Linux VM, then type in the following commands
    ```
    cd /
    ```
    ```
    tar -cvpzf ~/VM_backup.tar.gz --exclude=~/VM_backup.tar.gz --one-file-system / `
    ```
    Wait for the process to finish. You should see a tar file called  `VM_backup.tar.gz` under your user directory. Move that file to Windows.

    Shutdown the CSE Linux VM.

3. Set up WSL2

    - Run PowerShell as Administrator then use the following commands
        ```
        dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
        ```

        ```
        dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
        ```

        Restart your computer before the next step.

    - Download and install the WSL2 update package [here](https://docs.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)

    - Set WSL2 as default by running the following command in PowerShell
        ```
        wsl --set-default-version 2
        ```
    
    - Search for "Turn Windows features on or off" on your computer, then make sure "Windows Subsystem for Linux" is checked

    - Install `Windows Terminal` from the MS Store (Recommended)

    Refer to [Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/install-manual) if you're stuck.
    
4. Import tar backup to WSL

    Create a new directory (aka. folder) so you can store your WSL2 distro there.

    Use the following command in PowerShell. Replace `"RockyLinux"` with the name of your Linux distro, replace `C:\Users\Alex\RockyLinuxWSL` with the folder you just created, and finally replace `C:VM_backup.tar.gz` with the path to your tar file.
    ```
    wsl --import "RockyLinux" C:\Users\Alex\RockyLinuxWSL C:\VM_backup.tar.gz
    ```

5. Configure distro for use in WSL
    To run the WSL2 distro, use the command (Replace `RockyLinux` with the name of your Linux distro in all upcoming commands in this guide):
    ```
    wsl -d "RockyLinux"
    ```

    Your WSL2 distro should start.
    
    For distros using yum (e.g. Rocky Linux), run the following command to update your distro
    ```
    yum update -y && yum install passwd sudo -y
    ```
    
    Then set default user, replace `Alex` with the username you created when setting up the CSE Linux Home VM (probably your NetID)
    ```
    echo -e "[user]\ndefault=Alex" >> /etc/wsl.conf
    ```

    Now we can restart the distro. Open a new PowerShell window, then run
    ```
    wsl --terminate RockyLinux
    ```
    
    Wait for a few seconds, then run 
    ```
    wsl -d RockyLinux
    ```

    If you get an error message about `mountFstab`, edit the `wsl.conf` file to solve this. In your WSL2 distro, run 
    ```
    sudo emacs -nw /etc/wsl.conf
    ```

    Change `mountFstab = false ` or delete the line that contains `mountFstab`. Then add the following line
    ```
    appendWindowsPath = false`
    ```

    Press `Ctrl+X` followed by `Ctrl+S` to save the file. Then exit emacs by `Ctrl+X` followed by `Ctrl+C`.

    Now the error should be gond when you restart your distro.


6. Cleanup
    
    Once everything is set up, you can delete the tar file in the original Linux VM/Windows. Remove the original Linux Home VM and the image you downloaded from CSE website if you don't plan on using them.

# Usage
 In general, avoid storing files in Windows if you're going to work on them in Linux. 
 
 https://docs.microsoft.com/en-us/windows/wsl/filesystems


## Starting up
In PowerShell, type
- `wsl` if this is the only distro on your computer
- `wsl -d RockyLinux` if there are multiple distros

You might be able to find a shortcut to it in the start menu too. YOu can also created a profile for your distro in Windows Terminal/VS code.

## Shutting down
WSL2 distros should automatically shutdown after you have closed all Linux terminal/GUI applications (this might not happen immediately).

To manually shutdown the distro, use the command `wsl --shutdown` in PowerShell.

## Compacting your distro
A WSL2 distro stores its files in a virtual hard disk, you can compact it.

## Accessing Files in Windows/Linux
In Linux, your Windows C drive is mapped to `/mnt/c/`.

In Windows, your WSL drive should show up on the left side of File Explorer (or you can type in `\\$wsl\` in the address bar). You can find your Linux home directory under `home`.


# End
This is a guide for myself in case I need to set up the system again in the future. I might make this available publically if this could be healpful for other. Please excuse my poor grammar in this guide. Writing in pretty hard when you're only half awake :(

