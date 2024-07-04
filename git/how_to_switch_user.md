# How to switch the user if I have 2 github accounts?  
  First create a file named '.bash_profile' in your home folder so that the ssh private key will be loaded when you open a git bash.
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
  Then modify the file '.ssh/config' as below, the field **IdentityFile** tells us which user to use.
  ```bash
    ForwardAgent yes
     
    Host *
        # UseKeychain is a macOS feature, so ignore it on other OS
        IgnoreUnknown UseKeychain
     
    Host github.com
        HostName ssh.github.com
        User git
        Port 443
        AddKeysToAgent yes
    	
        # for user1
        # IdentityFile ~/.ssh/id_user1
    	
        # for user2
        IdentityFile ~/.ssh/id_user2
  ```
  To verify, run the following command
  ```bash
  ssh -T git@github.com
  ```
  and you will get something as below
  ```bash
  Hi user2! You've successfully authenticated, but GitHub does not provide shell access.
  ```
  If you want to swtich to user1, just comment the 'IdentityFile ~/.ssh/id_user2' and un-comment 'IdentityFile ~/.ssh/id_user1' in the file '.ssh/config'.