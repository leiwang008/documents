



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

 + [git rebase to resolve conflict](https://www.youtube.com/watch?v=2n0_UsMf7Pg)  
  `git rebase master` to rebase with code in the master branch  
  Then modify the conflicted file, for example a file named "foo"  
  `git add foo` to add the modified file "foo"  
  `git rebase --continue` to finish the rebase  
  Finally the conflicts have been resolved and we can push our commit.  
  `git push origin`  

+ [resolve confict video](https://www.youtube.com/watch?v=__cR7uPBOIk)  
  `git fetch origin` get the remote code without automatic merge  
  `git merge origin/master` merge local commit with the remote commit  
  Then modify the conflicted file, for example a file named "foo"  
  `git add foo` to add the modified file "foo"  
  `git commit - "fixed merge conflicts"`
  Finally the conflicts have been resolved and we can push our commit.
  `git push origin`  

+ How to check if a script like foo.sh is executable and how to make it executable in Git?  
  `git ls-files --stage`  
  100644 6643f02815ab62179560af03520b32117a6f6e00 0       foo.sh  
  The attribute '100644' means the file is not executable, '**100755**' represents executable  

  `git update-index --chmod=+x foo.sh` will change that attribute to '100755'  
  `git ls-files --stage`  
  100755 6643f02815ab62179560af03520b32117a6f6e00 0       foo.sh  

# Python
+ where python is installed?
  In your Python interpreter, type the following commands:  
  **\>>>** `import os`
  **\>>>** `import sys`
  **\>>>** `os.path.dirname(sys.executable)`

+ Run test with [pytest](https://docs.pytest.org/en/6.2.x/contents.html)
  1. [Run with multiple tests](https://docs.pytest.org/en/6.2.x/getting-started.html#run-multiple-tests)
  2. [How pytest discovery tests in different folders? Choosing a test layout.](https://docs.pytest.org/en/6.2.x/goodpractices.html#test-discovery)  
      pytest implements the following standard test discovery:  
        - If no arguments are specified then collection starts from testpaths (if configured) or the current directory. Alternatively, command line arguments can be used in any combination of directories, file names or node ids.  
        - Recurse into directories, unless they match norecursedirs.  
        - In those directories, search for test_*.py or *_test.py files, imported by their test package name.  
        - From those files, collect test items:  
            + test prefixed test functions or methods outside of class  
            + test prefixed test functions or methods inside Test prefixed test classes (without an __init__ method)  

      For examples of how to customize your test discovery Changing standard (Python) test discovery.  
    3. [How to change standard pytest discovery?](https://docs.pytest.org/en/6.2.x/example/pythoncollection.html)

+ [What is python modules?](https://docs.python.org/3/tutorial/modules.html)

+ [What is setup.py?](https://docs.python.org/3/distutils/setupscript.html)
+ [How to install modules?](https://docs.python.org/3/installing/index.html#installing-index)


# Jenkins
+ [Jenkins pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
+ [Jenkins pipeline](https://www.jenkins.io/doc/book/pipeline/)

# Bash
+ [What are $0 $# $@ $* $? ](https://segmentfault.com/a/1190000021435389)
+ [Condition test in Shell ](https://www.cnblogs.com/guanyf/p/7553940.html)
+ [Bash shell manual](https://www.gnu.org/software/bash/manual/bash.html)