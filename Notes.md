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
	`status` stores information about how the process exited

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
