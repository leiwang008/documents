# How to know what version has been installed?
```shell
wsl --list
```

# List more available images?
```shell
wsl -l -o
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