#### **EXERCISE 1: Linux Mint Virtual Machine**

Create a Linux Mint Virtual Machine on your computer. Check the distribution, which package manager it uses (yum, apt, apt-get). Which CLI editor is configured (Nano, Vi, Vim). What software center/software manager it uses. Which shell is configured for your user. 

**Solution:**
A) Initial setup:
1) Download and install Oracle Virtual box - refer to their site for link per distribution
2) Do a template setup of the needed Mint machine - CPU, memory, storage
3) Download Linux Mint image - https://www.linuxmint.com/download.php 
4) Load the image in the template and install Mint by following the instruction setup
B) Checks:
   Considering that Mint is based on Ubuntu, which itself is based on Debian.
1) Package manager - APT
2) Distribution info -  run "lsb_ release -a"  
3) Software manager/center - APT based 
4)  Check default shell - run "cat /etc/passwd | grep "username" " , look at the last section 

#### **EXERCISE 2: Bash Script - Install Java**

Write a bash script using Vim editor that installs the latest java version and checks whether java was installed successfully by executing a java -version command.

After installation command, it checks 3 conditions:

- 1. whether java is installed at all
- 2. whether an older Java version is installed (java version lower than 11)
- 3. whether a java version of 11 or higher was installed

It prints relevant informative messages for all 3 conditions. Installation was successful if the 3rd condition is met and you have Java version 11 or higher available.

**Solution:**
```
#!/bin/bash
#Install by default latest java
apt install -y default-jre

#Variable for storing output of command by redirecting to file
is_java_installed=$(java -version > is_java_installed.txt 2>&1) 
#Work with file content 
java_version=$(cat is_java_installed.txt | grep "openjdk version" | awk '{print substr($3,2,2)}')

if [ "$java_version" == "" ]
then 
	echo "There is no java installed. Installation failed."
elif [ "$java_version" -lt "11" ]
then
	echo "The installed java version is older than 11"
elif [ "$java_version" -ge 11 ]
then
	echo "Successfully installed java 11 or higher"
	echo "Closing ..."
fi
#remove file after check is done
rm  is_java_installed.txt
```
 Give executable rights by running:
 chmod +x  "<filename>" 

---

  

#### **EXERCISE 3: Bash Script - User Processes**

Write a bash script using Vim editor that checks all the processes running for the current user (USER env var) and prints out the processes in console. Hint: use ps aux command and grep for the user.

  

#### **EXERCISE 4: Bash Script - User Processes Sorted**

Extend the previous script to ask for a user input for sorting the processes output either by memory or CPU consumption, and print the sorted list.

  

#### **EXERCISE 5: Bash Script - Number of User Processes Sorted**

Extend the previous script to ask additionally for user input about how many processes to print. Hint: use head program to limit the number of outputs. 

**Combined solution for 3,4,5:** 
All needed commands can be explored to see their options within the manual. 
Syntax is "man "command" " .


```
#!/bin/bash
echo "Explore processes for the current user."
echo "Processes can be sorted by Memory or CPU and restricted resullt count."
echo "To sort by Memory, press "m". "
echo "To sort by CPU, press "c". "
echo "Choose sorting: "
read sort
echo "choose result outut (number): "
read count

if [  "$sort" = "m"  ]
then 
	ps aux --sort -%mem | grep $USER | head -n "$count"
elif [  "$sort" = "c"  ]
then
	ps aux --sort -%cpu | grep $USER | head -n "$count"
else
	echo "Wrong input..exiting"
fi

```

---

  

Context: We have a ready NodeJS application that needs to run on a server. The app is already configured to read in environment variables.

#### **EXERCISE 6: Bash Script - Start Node App**

Write a bash script with following logic: 

- Install NodeJS and NPM and print out which versions were installed
- Download an artifact file from the URL: [https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz](https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz). Hint: use curl or wget
    - Unzip the downloaded file
    - Set the following needed environment variables: APP_ENV=dev, DB_USER=myuser, DB_PWD=mysecret
    - Change into the unzipped package directory
    - Run the NodeJS application by executing the following commands:  npm install and node server.js

Notes:

- Make sure to run the application in background so that it doesn't block the terminal session where you execute the shell script
- If any of the variables is not set, the node app will print error message that env vars is not set and exit
- It will give you a warning about LOG_DIR variable not set. You can ignore it for now.

  

#### **EXERCISE 7: Bash Script - Node App Check Status**

Extend the script to check after running the application that the application has successfully started and prints out the application's running process and the port where it's listening. 
  

#### **EXERCISE 8: Bash Script - Node App with Log Directory**

Extend the script to accept a parameter input log_directory: a directory where application will write logs.

The script will check whether the parameter value is a directory name that doesn't exist and will create the directory, if it does exist, it sets the env var LOG_DIR to the directory's absolute path before running the application, so the application can read the LOG_DIR environment variable and write its logs there.

Note:

- Check the app.log file in the provided LOG_DIR directory.
- This is what the output of running the application must look like: [node-app-output.png](https://cdn.fs.teachablecdn.com/hryaYMxlRqtJSSa81yFl)

  

#### **EXERCISE 9: Bash Script - Node App with Service user**

You've been running the application with your user. But we need to adjust that and create own service user: myapp for the application to run. So extend the script to create the user and then run the application with the service user.

**Solution for 6,7,8,9 as final code, with all extentions:** 
``` 
#!/bin/bash

#Install needed tools for work
echo "Installing npm, nodejs, curl, net "
apt  install -y npm nodejs curl net-tools
echo "Installation finished!"

#check version of NPM and NodeJS
npm_ver= $(npm --version)
nodejs_ver= $(node --version)
echo "Installed successfully NPM version $npm_ver and NodeJS version $nodejs_ver."

#request log dir from the user
echo "Provide absolute path for log directory for the application:  "
echo "example path:  /home/user1/app1/log"
read  LOG_DIR

#check if location exists
if [  -d %LOG_DIR ]
then 
	echo "Provided directory exists."
else 
	echo "Provided directory not found. Creating..."
	 #create all needed folders until the destination, hence "-p" option"
	mkdir  -p $LOG_DIR 	 
	 echo "Log directory created!  $LOG_DIR"
fi

#set app user var
echo "Set app user var"
new_usr=myapp
echo "Create new user for application, set owner to log dir"
useradd $new_usr -m
chown $new_usr -R $LOG_DIR

#New user to run commands from now on.
# Get NodeJS archive
runuser -l $new_usr -c "curl https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz -o nodejs_archive.tgz"

#Unarchive the tar file
runuser -l $new_usr -c "tar zxvf  nodejs_archive.tgz"

#Set env vars, start app
runuser -l $new_usr -c "
	export APP_ENV=dev &&
	export DB_PWD=mysecret &&
	export DB_USER=myuser &&
	export LOG_DIR=$LOG_DIR &&
	cd package &&
	npm install &&
	node server.js"

#Display nodejs process
ps aux |  grep node 
#Display port for nodejs
netstat  -ltnp | grep :3000


```