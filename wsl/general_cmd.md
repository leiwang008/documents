# How to know what version has been installed?
```shell
wsl --list
```

```shell
Windows Subsystem for Linux Distributions:
Ubuntu-18.04 (Default)
docker-desktop
docker-desktop-data
```

# how to list the detail status of each distribution?
```shell
wsl --list --verbose
```

Normally you will see them stopped
```shell
  NAME              STATE           VERSION
* Ubuntu-24.04      Stopped         2
  docker-desktop    Stopped         2
```

But if you open a **wsl** terminal, you will see it running. If you run **Docker Desktop** and you will also see them running as **Docker Desktop** will use them.
```shell
  NAME              STATE           VERSION
* Ubuntu            Running         2
  docker-desktop    Running         2
```

# how to remove a distribution?
```
wsl --unregister Ubuntu-18.04
```

# List more available images?
```shell
wsl -l -o
```

```shell
The following is a list of valid distributions that can be installed.
Install using 'wsl --install -d <Distro>'.

NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
Ubuntu-24.04                    Ubuntu 24.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_7                 Oracle Linux 8.7
OracleLinux_9_1                 Oracle Linux 9.1
openSUSE-Leap-15.6              openSUSE Leap 15.6
SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
openSUSE-Tumbleweed             openSUSE Tumbleweed
```

# how to install a distribution?
```shell
wsl --install -d Ubuntu-24.04
```

# how to set the default wal?
```shell
wsl --setdefault Ubuntu-24.04
```

```shell
Windows Subsystem for Linux Distributions:
Ubuntu-24.04 (Default)
docker-desktop
docker-desktop-data
```

# open a wsl terminal
`wsl`

# shutdown a wsl
`wsl --shutdown`

# how to know the current distribution's version?
```shell
more /etc/os-release
```

```shell
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```


# How to upgrade the ubuntu inside wsl?
```shell
# open the wsl terminal
wsl
# inside the terminal

# Update package lists:
sudo apt update

# Upgrade installed packages:
sudo apt dist-upgrade

# Perform the release upgrade:
sudo do-release-upgrade

# If the upgrade doesn't start, you might need to edit the /etc/update-manager/release-upgrades file and change the prompt from lts or never to normal1. After completing these steps, your Ubuntu distribution inside WSL should be upgraded to the latest version available.

```