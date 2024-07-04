# How to avoid the **C:/Program Files/Git** when running .sh script in **Git Bash**?

When you run a .sh script in the Git Bash, there is an annoying thing is that the **C:/Program Files/Git** will be added in front of the leading slash /


java.io.FileNotFoundException: **C:\Program Files\Git**\kafka\_2.13-3.7.0\ssl\_certs\_bak\ca-cert (The system cannot find the path specified)


Found the solution, run the following in the Git Bash

```bash
export MSYS_NO_PATHCONV=1
```

References:

https://stackoverflow.com/questions/39632924/git-bash-is-adding-its-current-path-to-one-of-the-parameter

https://stackoverflow.com/questions/7250130/how-to-stop-mingw-and-msys-from-mangling-path-names-given-at-the-command-line/34386471#34386471

# an other problem to fix