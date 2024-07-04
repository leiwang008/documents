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