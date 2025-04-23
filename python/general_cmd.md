# How to know what version the python is?
```shell
python --version
```

# How to know where this python locates?
```shell
# where is the python
which python
# where is the pip
which pip
```

# Install packages, upgrade packages.
```shell
# upgrade pip
python -m pip install --upgrade pip

# pip install package
pip install asyncpg

```

# How to create a virtual python environment?
Normally the packages will be installed into the folder **<your_python>\Lib\site-packages**, that is the global installation and will be shared by all python program. But some times you want an isolated python environment specific to a certain project with its own dependencies, you can create virtual python environment.

```shell
# create the virtual environment
python -m venv .venv
# activate or use this virtual environment
source .venv/Scripts/activate

# verify that you are using it, you should see .venv in the path for the python
which python

# in the git bash, you should always see a (.venv) showing up

```

# how to install python and pip on wsl ubuntu?
```bash
# install python3
sudo apt install python3
sudo ln -s /usr/bin/python3 /usr/bin/python
python --version

# add the universe repository
sudo add-apt-repository universe

# install pip
sudo apt update
sudo apt install python3-pip
pip --version

# install python3 venv
sudo apt install python3.12-venv

```

# how to create virtual python environment?
```shell
# create virtual env
sudo python -m venv .venv

# activate it
. .venv/bin/activate

```

# Why python import file always goes to the old project?
Python searches for modules in the directories listed in **sys.path**
```shell
python
>>import sys
>>print(sys.path)
```
If the **old project path** is in there, Python will prioritize that over your current directory.


If you previously installed a project like:

```bash
pip install -e C:\repository\\your-project-dir
```
Then you'd uninstall it like this:

```bash
pip uninstall your-project-name
```

The **your-project-name** you pass to pip uninstall should match the package name declared in setup.py (or pyproject.toml if you're using Poetry or a modern build tool).
