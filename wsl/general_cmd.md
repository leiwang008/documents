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