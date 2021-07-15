



# Git

+ git reset
  `git reset --hard origin/main`  
  `git reset --hard 9fceb02`  
  `git reset --soft origin/main`  

+ git diff  
  `git diff` show the difference between files in the working tree and the those in the repository  
  `git diff --cached` show the difference between files in the stage area and the those in the repository  
  `git diff --cached --name-only` show the files in the stage area  

+ git commit  
  `git commit --amend -m "new message"` 

+ git tag  
  `git tag -a v1.0.0 -m "message for this tag"` make a tag with message in local repository  
  `git show v1.0.0` show the information of tag v1.0.0  
  `git tag -a v1.2 9fceb02 -m "blablabla"` make a tag for a special commit *9fceb02* with message in local repository  
  `git push origin v1.0.0` push tag v1.0.0 to remote repository  
  `git push origin --tags` push all tags to remote repository  

  `git tag -d <tagname>` Delete tag '\<tagname\>'  in the local repository  
  `git push origin --delete <tagname>` Delete tag '\<tagname\>'  in the remote repository  
  