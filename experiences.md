
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
