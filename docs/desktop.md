## Install Debian
   Install Debian with the netinstall iso image from <https://www.debian.org/> without any desktop environment.

## Installation
1. After installation, login and switch to root using su
    ```bash
    su
    ```

2. Update apt
    ```bash
    apt update
    ```

3. Install desktop and packages
    ```bash
    apt install \
        gnome-core \
        git \
        ssh
    ```

4. Reboot
    ```bash
    shutdown -r now
    ```

## Post install

### Fix network
1. After the restart, login, open the terminal.
    ```bash
    su
    ```

    ```bash
    mv /etc/network/interfaces /etc/network/interfaces.bak
    ```

    After the restart, the wired or wireless network should work.


    Change wifi powersave setting from 3 to 2 in `etc/NetworkManager/conf.d/default-wifi-powersave-on.conf` to fix wifi issue

    ``` txt
    [connection]
    wifi.powersave = 2
    ```

### Sudo
1. Install sudo
    ```bash
    su
    ```

    ```bash
    apt update && apt install sudo
    ```
2. Add user to sudoers group
    ```bash
    sudo visudo
    ```
3. Add the following line at the end of the file to grant coen user sudo privileges:
    ```bash
    coen  ALL=(ALL) NOPASSWD: ALL
    ```
    This will allow coen to run any command as root without a password prompt (remove NOPASSWD if you want the password prompt to appear).

### Ansible playbook
Run the Ansible playbook setup the desktop desired state, see [Infrastructure as Code](infrastructure-as-code.md#playbooks) page.