# Week 1 Applied Class Material

> In this unit, hands-on labs and assignments will be conducted on a virtual environment. You are required to follow the guidelines given in the Lab Environment Setup document to setup your lab environment before starting this lab. The goal of this lab is to get yourself familiar with the lab environment.


--- 

## Basic Linux Commands and Capabilities

### 1. Overview of Linux Commands
The objective of this lab is to introduce the students to the basic Linux commands that will be used in FIT5037 unit. Tasks in this lab can be tried on any Unix/Linux OS. But we encourage you to do all tasks in any Ubuntu docker container in SecureCorp GNS3 project.

### 2. Tasks
Once you start all nodes in the project open the terminal of any node (Internal-Attacker, Internal-Client etc.) and run the below commands. All nodes in SecureCorp project runs a minimized version of Ubuntu. Below commands will help to install few packages required for this lab.

```bash
apt update
apt install man-db
```

#### 2.1 General Unix Commands
Familiarize yourself with the usage of the following commands and complete the task with the guidance of your tutor.
| Command | Description |
| --- | --- |
| `man <cmd>` | an interface to the on-line references manuals |
| `whatis <cmd>` | displays manual page descriptions|
| `<cmd>` --help | print a help message and exit |
| `grep` | print lines matching a pattern |
| `cat`| concatenate files and print on the standard output |
| `<cmd> > file` | send the output of the command to that file |

_**Questions:**_
1. Which text-based command provides information on the use of other Linux commands and utilities? Identify at least two.
> Answer: `man` and `whatis`

2. View the man page of the command grep. Export the output to a file and see the content of the exported file.
```bash
man grep > how_to_use_grep
cat how_to_use_grep
```
> Answer: The output will be the manual page of the `grep` command, which includes its usage, options, and examples.

3. Identify the methods of running a command that you ran a short while ago without re-typing it. Use it to re-run the cat command from previous question.
```bash
history
!<line_number_of_the_command_from_history_list>
```
> Answer: The methods include using the `history` command to view the command history and the `!` (bang) operator followed by the line number to re-run a specific command.

4. Which key-stroke invokes auto-completion of commands and filenames?
> Answer: The `Tab` key is used for auto-completion of commands and filenames in the terminal.

#### 2.2 Basic Networking Commands
Use man pages to understand the usage of the below commands and complete the task with the guidance of your tutor.

| Command | Description |
| --- | --- |
| ifconfig | configure a network interface |
| ping | send ICMP ECHO_REQUEST to network hosts |
| netstat | print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships |
| hostname | show or set the system’s host name |
| route | show / manipulate the IP routing table |

1. Check the IP address of the node. You can one of the following commands.
```bash
ip address
ifconfig
```
2. Check the Hostname of the node.
```bash
hostname
```
3. Check the local routes in the node and the current network connections in the node.
```bash
netstat
route
```
4. Check the connectivity with the internet by pinging the Google DNS server.
```bash
ping 8.8.8.8
```

#### 2.3 File and Directory Manipulation

**2.3.1 Root Directory and its Sub-directories**
While we are poking around the Linux file system, take a look at the files in the root directory. You should see directories with names like `/bin`, `/home`, `/lib`, and `/usr`.

Every file and directory in the file system has a path. Unix paths are delimited by forward slashes (/). e.g. `/home/username/FIT5037/`

---

**Special directories:**
- Root Directory ( `/` ): The Top-Most directory.
- Bin Directory: Executable programs that comprise the GNU/Linux utilities.
- Home Directory ( `~` ): Current User’s directory.
- Current Directory ( `.` ): The directory you’re in.
- Parent Directory ( `..` ): The directory above.
- Absolute paths: start with ( `/` ). i.e the root directory.
- Relative paths: start from your current directory.

---

**File and Directory Manipulation Commands**
| Command | Description |
| --- | --- |
| `cat <file name>` | concatenate files and print on the standard output |
| `mkdir <directory name>` | make directories |
| `rmdir <directory name>` | remove empty directories |
| `rm <file name>` | remove files or directories |
| `cp <source> <destination>` | copy files and directories |
| `mv <source> <destination>` | move or rename files and directories |
| `find <path> -name <filename>` | search for files in a directory hierarchy |
| `locate <filename>` | find files by name |
| `pwd` | print location of current/working directory |
| `ls` | list directory contents |
| `cd <directory>` | change working directory |
| `chmod <permissions> <file>` | change file mode bits |
| `tar` | The GNU version of the tar archiving utility |

**2.3.2 File and Directory Manipulation**
1. Find your current working directory.
```bash
pwd
```

2. Change your working directory to /home. 
```bash
cd home
```
When working with files in these Docker containers, always change the working directory to /home to
ensure your files are not deleted when restarting the node.
3. Create a new directory with the name ‘scripts’
```bash
mkdir scripts
```
4. Create a new file in the scripts directory with the name rsa.py and copy the content from the textbook-
RSA.py file in Moodle. Save the file.
```bash
nano scripts/rsa.py
```
Press `ctrl + X` to save and exit
5. Rename the rsa.py file as textbook-rsa.py
```bash
mv scripts/rsa.py scripts/textbook-rsa.py
```
6. List the permissions of the file. Give everyone read and executable permissions to the file. Only the
owner (root) should have write permissions
```bash
ls -l scripts
chmod 755 scripts
```
7. Run the textbook-rsa.py file using the relative path as well as the absolute path of the file.
```bash
python3 scripts/textbook-rsa.py
```
```bash
python3 /home/scripts/textbook-rsa.py
```
8. Find all ‘print’ statements in the textbook-rsa.py file without openning the file.
```bash
grep ‘print’ scripts/textbook-rsa.py
```
9. Copy the textbook-rsa.py file to /home
```bash
cp scripts/textbook-rsa.py ./textbook-rsa.py
```
10. Delete the /home/scripts/texbook-rsa.py file.
```bash
rm scripts/texbook-rsa.py
```
11. Remove the scripts directory.
```bash
rmdir scripts
```
```bash
ls -l scripts
chmod 755 scripts
```
7. Run the `textbook-rsa.py` file using the relative path as well as the absolute path of the file.
```bash
python3 scripts/textbook-rsa.py
python3 /home/scripts/textbook-rsa.py
```
8. Find all ‘print’ statements in the textbook-rsa.py file without openning the file.
```bash
grep ‘print’ scripts/textbook-rsa.py
```bash
grep ‘print’ scripts/textbook-rsa.py
```
9. Copy the textbook-rsa.py file to /home
```bash
cp scripts/textbook-rsa.py /home
cp scripts/textbook-rsa.py ./textbook-rsa.py
```
10. Delete the /home/scripts/textbook-rsa.py file.
```bash
rm scripts/textbook-rsa.py
```
11. Remove the scripts directory.
```bash
rmdir scripts
```

---

#### 2.4 Linux Shell
You can write programs to do all the things you want in the Unix shell. A Shell script is a bunch of commands saved in one executable file with extension `.sh`.

1. What is your default Linux shell?
```bash
echo $SHELL
```
2. What is the PATH variable, what does it do, and how do you inspect its value?
```bash
echo $PATH
```
3. Create a simple shell script script.sh for listing all files and sub directories in the /etc directory.
```bash
nano script.sh
ls -l /etc
Press CTRL+X to save and exit
```
4. Make your script executable and run the script.
```bash
chmod +x script.sh
./script.sh
```
5. Why it is necessary to use the prefix `./` explicitly to run an executable file?
> When you run a command in the shell, the system searches for the executable file in the directories listed in the PATH environment variable. If the current directory (.) is not in the PATH, you need to specify it explicitly with `./` to run the script.