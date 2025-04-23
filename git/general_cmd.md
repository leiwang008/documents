# How to get a clean version?
+ git reset
  `git reset --hard origin/main`  
  `git reset --hard 9fceb02`  
  `git reset --soft origin/main`  

+ `git branch --list` list the branches in the local repository

+ `git status` to see the current status of the git repository

+ `git restore <filename>` to cancel the modification of a file, and the file's content will be the same as the current commit.

+ git commit  
  `git commit --amend -m "new message"` 

+ `git push ` to push code

+ git diff  
  `git diff` show the difference between files in the working tree and the those in the repository  
  `git diff --cached` show the difference between files in the stage area and the those in the repository  
  `git diff --cached --name-only` show the files in the stage area  


# How to make tag?
+ git tag  
  `git tag -a v1.0.0 -m "message for this tag"` make a tag with message in local repository  
  `git show v1.0.0` show the information of tag v1.0.0  
  `git tag -a v1.2 9fceb02 -m "blablabla"` make a tag for a special commit *9fceb02* with message in local repository  
  `git push origin v1.0.0` push tag v1.0.0 to remote repository  
  `git push origin --tags` push all tags to remote repository  

  `git tag -d <tagname>` Delete tag '\<tagname\>'  in the local repository  
  `git push origin --delete <tagname>` Delete tag '\<tagname\>'  in the remote repository  


# How to resolve the conflict?
 + [git rebase to resolve conflict](https://www.youtube.com/watch?v=2n0_UsMf7Pg)  
  `git rebase master` to rebase with code in the master branch  
  Then modify the conflicted file, for example a file named "foo"  
  `git add foo` to add the modified file "foo"  
  `git rebase --continue` to finish the rebase  
  Finally the conflicts have been resolved and we can push our commit.  
  `git push origin`  

+ [resolve conflict video](https://www.youtube.com/watch?v=__cR7uPBOIk)  
  `git fetch origin` get the remote code without automatic merge  
  `git merge origin/master` merge local commit with the remote commit  
  Then modify the conflicted file, for example a file named "foo"  
  `git add foo` to add the modified file "foo"  
  `git commit - "fixed merge conflicts"`
  Finally the conflicts have been resolved and we can push our commit.
  `git push origin`  


# How to check if a script like foo.sh is executable and how to make it executable in Git?  
  `git ls-files --stage`  
  100644 6643f02815ab62179560af03520b32117a6f6e00 0       foo.sh  
  The attribute '100644' means the file is not executable, '**100755**' represents executable  

  `git update-index --chmod=+x foo.sh` will change that attribute to '100755'  
  `git ls-files --stage`  
  100755 6643f02815ab62179560af03520b32117a6f6e00 0       foo.sh  
  
# How to move file to unstaged?
`git restore --staged kafka/image.png` will move file **kafka/image.png** from staged area to unstaged area.

# How to load your ssh private key to 'ssh agent' when you open the git bash?  
After [adding your ssh public key to github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account), you still need to load your private key into the 'ssh agent' in your local machine so that the ssh connection will be established successfully.   
Create a file named '.bash_profile' in your home folder so that the ssh private key will be loaded when you open a git bash.
  ```bash
    #!/bin/bash
  
    if ps --process $SSH_AGENT_PID > /dev/null
    then
    	echo "ssh-agent is already running"
    	# Do something knowing the pid exists, i.e. the process with $PID is running
    else
    	echo "starting ssh agent ..."
    	eval `ssh-agent -s`
    fi
    
    if ps --process $SSH_AGENT_PID > /dev/null
    then
    	echo "ssh agent process SSH_AGENT_PID=$SSH_AGENT_PID is running ... "
    	echo "add ssh keys to the agent ... "
    	ssh-add ~/.ssh/id_user1
    	ssh-add ~/.ssh/id_user2
    else
    	echo "no ssh agent, problem ..."
    fi
  ```

# how to save the local files?
```bash
git stash
git stash push -m "save my local work"
```

# how to list all stashed work?
```bash
git stash list
```

```shell
stash@{0}: On backend-api-lif: unique spec vehicle type
stash@{1}: WIP on backend-api-lif: 8f7d33b feat(api): add constraints for vehicle_type_id of edge table
stash@{2}: On backend-api-lif: start end node id constraint
```


# how to apply my local work?
```bash
# apply the latest work
git stash apply

# apply a certain stash
git stash apply stash@{2}

```

# how to amend a commit?
```bash
# after you make the commit, you can still modify more files

# add them
git add .

# amend the commit
git commit --amend

# an editor will open, modify the commit message and save the file and close it
git push
```


# how to modify a historical commit message?
```bash
# change the previous 5th commit message
git rebase -i HEAD~5
# Change 'pick' to 'reword' for the fifth commit in the editor
# Save and close the editor
# Modify the commit message in the new editor
# Save and close the editor
git push --force
```

# How to reset to a certain commit on remote repository?
There are 2 ways to do so.  
1. rollback by commit-hash
+ reset to a previous commit   
  `git reset --hard <commit-hash>`
+ push the commit to remote repository hardly  
  `git push -f origin master`

2. rollback to last commit.
or you can do this
+ undo the last commit locally
  `git reset HEAD^`
+ force push the local commit
  `git push origin +HEAD`


# How to make vscode your git editor
```bash
# --wait is important — it tells Git to wait until you close the VS Code window before continuing
git config --global core.editor "code --wait"
# check what your editor is currently set to
git config --global core.editor
```