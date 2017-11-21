## 11/21/17: A pipe by any other name...

* Named pipes are also known as FIFOs; their names let them be identified across different programs
* `$ mkfifo <PIPE NAME>`
	* Shell command to create a named pipe, listed as `prw-r--r--` via `ls -l`
	* Use `cat <PIPE NAME>` to read from and `cat > <PIPE NAME>` to write to the pipe
	* Multiple terminals can write to the same pipe without any problem
	* If multiple terminals are reading from the same pipe, only one will receive a message
		* There is no order to which terminal will receive it
	* Even after removing a pipe, connections between terminals still exist; it's just an unnamed pipe now
* `mkfifo(<NAME>, <PERMISSIONS>)` - `<sys/types.h> <sys/stat.h>`
	* Makes a FIFO in C, returning 0 on success and -1 on failure
	* The FIFO acts like a regular file, which can be opened, read, written, and closed
	* FIFOs will block on open (unnamed pipes block on read) until both ends of the pipe have a connection
* Use semicolons in bash to run multiple commands on one line
* `cd` cannot be execed

---

## 11/17/17: Ceci n'est pas une pipe

**Pipe**
* Conduit between 2 separate processes on the same computer
* Consists of a read end and a write end, unidirectional
* Acts like a file (opening, closing, added to the file table)
* You can transfer any data you want through a pipe using read/write
* Unnamed pipes don't have external identifiers, whereas named pipes do
* `pipe(int pipefd[2])` - `<unistd.h>`
	* Creates an unnamed pipe, returning 0 if successful and -1 otherwise
	* `pipefd` contains the descriptors for the read and write ends respectively
	* Example:
```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
	
int main() {
	int READ = 0, WRITE = 1, f;
	int fds[2];
	pipe(fds);
	f = fork();
	if (!f) {
		close(fds[READ]);
		char s[10] = "hello!";
		write(fds[WRITE], s, sizeof(s));
	}
	else {
		close(fds[WRITE]);
		char s[10];
		read(fds[READ], s, sizeof(s));
		printf("Parent received: \"%s\"\n", s);
	}
	return 0;
```

---

## 11/15/17: Playing Favorites

```C
int x = 302;
char *p = &x;

Little-endian representation of x
| 46 |   1 |  0 |  0 | -> | 00101110 | 00000001 | ... |

Big-endian representation of x
|  0 |   0 |  1 | 46 | -> | ... | 00000001 | 00101110 |
```
* Depending on your operating system, the order of the bytes may be reversed
* The _bits_ in each byte, however, are never reversed
* The most significant (biggest) digit is stored first in big-endian, and vice versa for little-endian
* `WEXITSTATUS(int status)`
	* Not a function, but a macro (hence the caps) that looks at the bytes of `status` and finds what the child returned (use `WIFEXITED(int status)` to check if the child returned something in the first place)
* `waitpid(pid, status, options)` - `<unistd.h>`
	* Waits for a specific child specified by `pid`, or any child if -1
	* `options` can set other behavior for wait, does nothing if 0

---

## 11/14/17: Wait for it

* Order in which forked processes are run is unpredictable
* `getppid()` returns the parent PID, which all processes have
	* Parent PID of the original process is the Terminal window or wherever you ran the file from
	* Sometimes returns 1 (the init process) because the parent process has ended when `getppid()` was called
	* These are called orphan processes
* A thread can run on its own, but shares memory space with whatever process created it (only works while process is running)
* `wait(int *status)` - `<unistd.h>`
	* Stops a parent process from running until any child gives it a signal (usually the child exiting)
	* Returns the PID of the child that exited, or -1 (errno)
	* `status` stores information about how the process exited, both the `exit()` or return value and signal number if it exited abnormally

---

## 11/13/17: What the fork?

**Managing Sub-processes**
* `fork()` - `<unistd.h>`
	* Creates a separate process based on the original one
	* The new process is called a child, while the original one is called a parent
	* The child is a duplicate of the parent process; all parts of the parent process are copied, including stack, heap, and the file table
	* Only the PID differentiates the two
	* Example:
```C
printf("Pre-fork\nPID: %d\n", getpid());
fork();
printf("Post-fork\nPID: %d\n", getpid());
// Second print statement will be printed TWICE, once by the parent and once by the child
```
	* DO NOT USE `while (1) fork();`
	* Returns 0 to the child and the child's PID to the parent (or -1 for errno)

---

## 11/09/17: Time to make an executive decision

**The `exec` Family - `<unistd.h>`**
* C functions that run other programs from within
* Replaces current process with the new program, so PID does not change
* `execlp(<PROGRAM NAME>, <ARG1>, <ARG2>, ..., NULL)`
	* PROGRAM NAME and ARG1 are identical
	* Example: `execlp("ls", "ls", "-a", NULL)`
	* All code after an exec statement isn't run
* `execvp(<PROGRAM NAME>, <ARRAY OF STRING ARGS>)`
	* Last argument in the array must be NULL
	* Example:
```C
args[0] = "ls";
args[1] = "-a";
args[2] = "-l";
args[3] = NULL;

execvp(args[0], args);
```

---

## 11/08/17: Sending mixed signals

* `getpid()` and `sleep(<TIME>)` are pretty self-explanatory C commands

**Signals**

* Limited way of sending information to a process
* `kill`
	* Command line utility to send a signal to a process
	* `$ kill <PID>` sends signal 15 (SIGTERM) to the specified PID
	* `$ kill -<SIGNAL> <PID>` sends a specific signal to the specified PID
* `killall [-<SIGNAL>] <PROCESS>`
	* Sends SIGTERM or SIGNAL to all processes with the specified name
* CTRL + C sends SIGINT and interrupts the process

**Signal Handling in C**

* `kill(<PID>, <SIGNAL>)` - `<signal.h>`
	* Returns 0 on success or -1 (errno) on failure
* sighandler
	* In order to intercept signals in C, you have to create a signal handling function
	* Some signals cannot be caught, e.g., SIGKILL
	* `static void sighandler(int signo)`
		* Static means the function can only be called within the file it is defined (not headers)
	* Usually, just write a single sighandler for all signals you expect to intercept
* `signal(<SIGNAL>, sighandler)` - `<signal.h>`
	* After you create the function, you attach the signals to it using the original function

---

## 11/06/17: Are your processes running? Then you should go out and catch them!

**More Command Line Arguments**

* `fgets(<DESTINATION>, <BYTES>, <FILE POINTER>)` - `<stdio.h>`
	* Reads in from a file stream and stores it in a string
	* Reads at most \<BYTES> - 1 characters from the pointer (adds a null at the end), including newlines
	* File pointer is type `FILE *`, more complex than a file descriptor
		* `stdin` is a file pointer
	* Stops at newlines, end of file, or byte limit
	* What if you wanted to read in numbers? Just use...
* `sscanf(<SOURCE STRING>, <FORMAT STRING>, <VAR 1>, <VAR 2>, ...)` - `<stdio.h>`
	* Scans a string and extracts values based on a format string

**Processes**

* Every running program is a process
* Programs can create subprocesses, but these are the same as regular processes
* A processor can handle 1 process per cycle or per core
* "Multitasking" only appears to happen due to the processor switching between all the active processes quickly
* PID
	* Unique identifiers for processes
	* PID 1 is the init process, always running
	* Each entry in the `/proc` directory is a current PID
	* The `ps` command in Terminal lists what processes are currently running
	* `ps -a` lists all processes across all accounts
	* `ps -ax` lists processes not attached to Terminals

---

## 11/03/17: Input? fgets about it!

* In [Work 08](https://github.com/iwang2/08_stat/blob/master/stat.c), we used `st_mode` in `struct stat`, which returned a 6-digit octal as opposed to the 3-digit one we were expecting
* To remove the first three digits, use the bitwise & operator:

```
Mode
_ _ _ _  _ _ _  _ _ _  _ _ _
         r w x  r w x  r w x
     &0b 1 1 1  1 1 1  1 1 1
```

**Command Line Arguments**

* `int main(int argc, char *argv[])`
	* `argc` - number of command line arguments
	* `argv` - array of command line arguments
	* Program name is considered the first command line argument, e.g., `./a.out`
* `scanf(<FORMAT STRING>, <VAR 1>, <VAR 2>, ...)` - `<stdio.h>`
	* Reads in data from stdin using the format string to determine types and puts the data in each variable
	* Example:
```C
int x; float f; double d;
scanf("%d %f %lf", &x, &f, *d);
```

---
