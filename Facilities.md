# Topic: Facilities for User/Programmers

*Lectures 5 & 6*
Resources: [File Sys Implementation: https://pages.cs.wisc.edu/~remzi/OSTEP/file-implementation.pdf]

*Previous Lecture: Abstraction*

## Learning Objectives:
- Talking to the Computer (machine)
- Permissions (File Modes)
- Controlling Running Programs
- File Abstraction and File Systems
- Tutorial 3 Notes
___
# 1. Process: Talking to the Computer:
*Steps 4, 5, and 6 are not chronologically necessary*
### a. Connect to the Computer (TERMINAL: device)
- Terminal is a device.
- conversion between human <-> machine (I/O)
### b. Log in (AUTHENTICATION)
- Prove who you claim to be.
- Important because we have *time sharing* nowadays i.e., users use the same computer at the same time. 
    #### 3 Authentication Factors:
        1. what you know (knowledge)
        2. what you have (possessions)
        3. what you are (biometrics)
### c. Send Commands (SHELL: program)
- **Shell** is a program (not a device) or a *command interpreter* to -> run other programs
- Internal/External
### d. Control Programs 

### e. Feed data to programs
- Data exchange between programs
### f. Programs work with OS Services
_________

## What is a Terminal?
- is an electronic or electromechanical hardware device that can be used for:
    - entering data into, and 
    - transcribing data from a computer or a computing system
- **Teletypewriter (TTY)** is a device where you type your input and output will be printed. This is helpful for people who are deaf, etc.
    ```terminal
    > ls /dev/tty*
    // Produces many tty keys in your terminal

    Other involved commands:
    > stty
    > who
    > echo
    > echo $TERM, etc...
    > which getty (for you to log in to the system)
    ```
- pseudo-terminals (who command) or terminal emulators

The use of *MAN PAGE* is always helpful when you forget some commands.

## Users and Access Control
- File Protection Mechanisms
- Username -> UID, user group -> GID
- Login Process: /usr/bin/login
    - More of this in Tutorial 4
- The password file
    ```terminal
    /etc/passwd -> $HOME, $SHELL, $PATH, ...
    /etc/shadow
    ```

# 2. Permissions (File Modes)
**File Modes**
- wether the owner can decide and transfer the access. *Discretionary Access Control*
- Idea: I am who I am, **so what is your decision?**

**In the form of running processes**
- UID, EUID (effective UID), FSUID (File System UID)
    - **EUID** is used in most access control checks
    ```terminal
    read, write, execute

    > setuid 
        - sets EUID to something you specify
    > setid
    > sticky bit
    ```

### Shell
The command interpreter
- The GUI is also like a shell, but in a complex form
- **Typical steps include:**
    1. Initialize the environment (e.g., env variables)
    2. Display a prompt
    3. Parse user input, and perform the task in the case of internal commands
    4. fork(), and wait in the parent process
    5. Try to find the program (external command)
    6. Manipulate fds as needed
    7. exec()to attempt execution
    8. Report errors (if any) and terminate child

- Separating exec() and fork() provides many benefits such as **in terms of flexibility and control over process creation and management.** As well as:
    - Child Process Env Customization
    - Clean Parent-Child Separation: 
    - Portability: Aligns with unix philosophy of simplicity and composability

# 3. Controlling Running Programs
#### All processes have parent process *(except init)*
- **init (PID=1)** is the ancestor of all **user** processes
- **kthreadd (PID=2)** is the ancestor of all **kernel** processes
#### The wait() system call
- blocks the parent process until a child process exits
    - a process must be waited on or it enters a zombie state after it exits (zombie state can't be killed)
- pstree: is a geat tool for monitoring system processes and debugging.
```terminal
pstree command to modify its behaviour:
-p to show process IDs (PIDs)
-u to show the usernames of the processes
-a to show command-line arguments
```
#### Signals
- is a *limited* (because it is predefined) form of IPC or Inter-Process Communication:
- asynchronous (SIGINT, interrupt: ctrl+C), an OS artifact
- No new signal can be defined; use SIGUSR1 and SIGUSR2 for custom purposes.  
    Signals are provided by the OS (Software)
    Interruptions are provided by the CPU (Hardware)

#### Signal Handling
- is when the OS interrupts the **signalled process** and **calls the handler function**
- Signal handlers are defined by the **process** (except for new signals)
    - signaction

    **See Commands**
    ```terminal 
    To send signals or see predefined signals:
        > kill -l           // send signal

    To define process (header function):
        > sigaction()       // define function, OR
        > signal()          // not recommended
    ```

**Pipeline & Redirection**
*Connecting Programs: Pipeline* is a shell feature which gives you the ability to connect the standard output of one program and standard input of another program.
- Multiple stages
    ```terminal
    Knowing that 'ls' and 'wc' are different processes
        > ls | wc -l            // both are being ran at the same time
                                // this outputs number of lines in the current directory

    Pipe commands:
    |   stdout->stdin
    |&  stdout+stderr->stdin 
    ```
*Redirection* is a shell feature which gives you the ability to redirect the standard output of one program to a file. 


# 4. File Abstraction & File Systems
**File** - is a linear array of bytes stored persistently
**File system** - ways of organizing files.
- *Implementations:* File system types are determined by purposes. Optical dics, flash memory, Union/overlay, etc. 
- *Mounting File Systems:* connect to the uniform file-system tree whereby the mount point is an existing directory.
    ```terminal
    sudo mount ../filename.vfat /mnt        // mount
    sudo unmount /mnt                       // unmount
    ```
**Pathnames** applies to all operating systems
- Paths are hierarchical, and there is root ("/")
    **Types:**
    1. Absolute pathname - e.g., /home/student/comp3000 (hierarchical)
    2. Relative pathname - e.g., tut1/abc (current working directory: CWD) per process
    
**Operations on Files**
```terminal
Create
    > int fd = open("/path/to/dir", O_CREAT | O_WRONLY | O_TRUNC, S_IRUSR | S_IWUSR)
    Example way of creating files 
    O_TRUNC: If the file exists, it truncates (clears) its contents to zero length.
        S_IRUSR | S_IWUSR: These are file permissions:
        S_IRUSR (0400) → Owner has read permission.
        S_IWUSR (0200) → Owner has write permission.

Open
    > fd        // for existing files, fd is needed for any operations

Read/Write/Close
    
Seek
    > lseek()    
```

**Address Space Layout Randomization (ASLR)**
- its purpose is to randomize the layout of the address so that the attacker cannot predict where a memory object is. 
    - apply a randomly generated offset to the bases of critical segments, e.g., heap, stack, and loaded libraries.
- *a security concept topic*

__________________________
# 5. Tutorial no. 3 Notes
#### Controlling Processes 
- (Linux) once a process is created, you can control it by sending them signals (i.e., typing key sequences in most shells in the terminal)
        Example of key sequences:
        > kill -<signal> <process ID> 
        > kill -STOP 4542                   // to stop process 4542
                // default: kill sends TERM signal

#### Standard input/output, Shell and Terminal
- The standard input (e.g., /dev/stdin), 
- output (e.g., /dev/stdout) and 
- error (/dev/stderr) 
    - are symbolic endpoints in any programs (of which the shell is an example), that can be redirected anywhere, where output goes, input comes and error is reported
- Further in the course, you will find **stdin, stdout, stderr** useful especially when combined with pipeline/redirection
