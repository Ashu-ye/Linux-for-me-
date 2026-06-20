# File System
<img width="1239" height="758" alt="image" src="https://github.com/user-attachments/assets/c5b2d3ba-abaa-43e5-8441-9189ef5b05ba" />

### Root directory 
```cd /```
### Home directory
```cd ~```
### list command
```ls```
**or**

```ls -l```
### list command to see hidden file 
```ls -all```

<img width="1098" height="847" alt="image" src="https://github.com/user-attachments/assets/bbab12a6-58f6-497b-81c3-10435eccd20b" />

### clear screen
```clear```

### user name
```whoami```
### give sudoers permission for single cmd
```sudo your_cmd```

What is absolute path and relative path

**Absolute Path:-**
  An absolute path specifies a location starting from the absolute root of the file system 

  **Full Blueprint:** It provides the complete address of a file or directory.
  
  **Context Independent:** It works exactly the same way no matter which folder you are currently inside.
  
  **Example:** /home/user/Documents/report.txt
  
  
**Relative Path:-**
  Relative path specifies a location relative to your current working directory
  - **Shortcuts**: It uses your current position as the reference point to find a nearby file.
  - **Special Symbols:**
    - . refers to the current directory.
    - .. refers to the parent directory (one level up).
  - **Example:** Documents/report.txt (if you are already inside /home/user)

### To get help
```cmd --help```

**Example:** aircrack-ng --help

**OR**
```cmd -h```

**Example:** nmap -h

### to get manual of any command

```man cmd```

**Example:** man nmap


### To find specific keyword 
```locate cmd```

**Example:** locate aircrack-ng

```whereis cmd```

**Example:** whereis aircrack-ng

```which cmd```

**Example:** which aircrack-ng

#### Difference Between whereis and which
=>  which shows you the single executable file that runs when you type a command, 

=> whereis finds the binary, source code, and manual pages for that command.




### To see the list of directories currently in your path
```echo $PATH```

### search for files and directories based on their names etc

```find directory_name file_name```
* Search by Name: Find a specific file anywhere inside the current directory (.)
  ```find . -name "report.txt"```
* Case-Insensitive Search: Find a file regardless of capitalization (e.g., Report.txt, REPORT.TXT).
  ```find . -iname "report.txt"```

* Search by Extension: Find all Python files within a specific folder path.
  ```find /home/user/projects -name "*.py"```
  
* Search by Type: Find only folders (directories), not files.
  ```find . -type d -name "assets"```
  
* Search by Size: Find files larger than 100 Megabytes.
  ```find /var/log -size +100M```
  
* Search by Modification Time: Find files modified within the last 24 hours.
  ```find ~ -mtime -1```
  
* Search by Empty Files: Find all empty files and folders to clean up.
  ```find . -empty```
  
* to find and automatically delete all files ending in .tmp
  ```find . -name "*.tmp" -delete```

### To view active processes running

```ps -ef```
**Note:** 

-e: Select all processes.

-f: Display full-format listing (shows UID, PID, PPID, and start time).

```ps aux```

a: Show processes for all users.

u: Display the process's user/owner and detailed resource usage.

x: Include processes not attached to a terminal (like background services).

### Start and Stop process

**To start the process**
```sudo systemctl start processname```

**To start the process**
```sudo systemctl stop processname```


