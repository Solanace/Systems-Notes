## 01/11/18: Cisco in an Hour™

#### Layer Models of Networking
Due to the complexity of network communications, the topic is often conceptualized into distinct layers so people can work on specific components rather than everything at once. The bottom layer is the most concrete, with each subsequent layer becoming more abstract (relying less on the physical connections and more on code). There are various competing models, including the OSI (Open Systems Interconnections) and TCP/IP Models.

* TCP/IP Model Layers
	1. Application
	2. Transport
	3. Internet
	4. Link
* Link Layer
	* Point-to-point transmission between devices on the same (local) network
	* Combines physically connecting computers with basic addressing and transmission protocols
	* Physical connection
		* How to transmit bits between two computers

#### Brief History of Physical Connections
* Thicknet
	* A single coaxial cable that runs along the network
	* "Vampire taps" cut into the cable and connect to a computer
* Thinnet
	* A single coaxial cable that runs along the network
	* T-Connectors connect computers to the main cable
* Token Ring
	* Each computer is connected in a ring to each other
	* Only one computer has command of network resources at a time, AKA having the token
	* The network sends a "token" through the ring, which contains the identity of the computer allowed to use the network. All other computers must wait to use the network.
* Ethernet
	* Multiple computers connect to a single hub or switch
		* Hub - Broadcasts the data to all the computers
		* Switch - Sends data to a specific computer

---

## 01/08/18: Stop. Collaborate, and Listen

* `accept(socket descriptor, address, address length)` - `<sys/socket.h>`
	* Accept the next client in the queue of a soccket in the listen state
	* Used for stream sockets
	* Performs server side of the 3-way handshake
	* Creates a new socket for communicating with the client without modifying the listening socket
	* Returns a descriptor to the new socket
	* Blocks until a connection is made
	* `socket descriptor` - Descriptor for the listening socket
	* `address` - Pointer to a `struct sockaddr_storage` that contains information about the new socket after `accept()`
	* `address length` - Pointer to a variable that will contain the size of the new sock
	* Example:
	```C
	// create socket
	int sd = socket(AF_INET, SOCK_STREAM, 0);
	// use getaddrinfo and bind

	listen(sd, 10);

	int client_socket;
	socklen_t sock_size;

	struct sockaddr_storage client_address;
	client_socket = accept(sd, (struct sockaddr *)&client_address, &sock_size);
	```
* `connect(socket descriptor, adddres, address length)` - `<sys/socket.h>`, `<sys/types.h>`
	* Connect to a socket currently in the listening state
	* Used for stream sockets
	* Performs the client side of the 3-way handshake
	* Binds the socket to an address and port
	* Blocks until a connection is made (or fails)
	* `socket descriptor` - Descriptor for the socket
	* `address` - Pointer to a `struct sock_addr` representing the address
	* `address length` - Size of the address in bytes
	* Example:
	```C
	// create socket
	int sd = socket(AF_INET, SOCK_STREAM, 0);
	
	struct addrinfo *hints, *results;
	// use getaddrinfo()
	
	connect(sd, results->ai_addr, results->ai_addrlen);
	```

---

## 01/05/18: Stop. Collaborate, and Listen

#### To Use a Socket
1. Create the socket
2. Bind it to an address and port (server)
3. Listen and accept (server) or connect (client)
4. Send/receive data

* `socket(domain, type, protocol)` - `<sys/socket.h>`
	* Creates a socket
	* Returns a socket descriptor (int a la file desciptor)
	* `domain` - type of address, use either `AF_INET` or `AF_INET6`
	* `type` - `SOCK_STREAM` or `SOCK_DGRAM`
	* `protocol` - Combination of domain and type settings; if set to 0, the OS will set the correct protocol
	* Example: `int sd = socket(AF_INET, SOCK_STREAM, 0);`
* `struct addrinfo`
	* Used in system library calls to represent network addresses
	* Contains information like the IP address, port, and protocol
	* Variables
		* `ai_family`
			* `AF_INET` - IPv4
			* `AF_INET6` - IPv6
			* `AF_UNSPEC` - IPv4 or IPv6
		* `ai_socktype`
			* `SOCK_STREAM`
			* `SOCK_DGRAM`
		* `ai_flags`
			* `AI_PASSIVE` - Automatically set to any incoming IP address
		* `ai_addr` - Pointer to a `struct sockaddr` containing the IP address
		* `ai_addrlen` - Size of the address in bytes
* `getaddrinfo(node, service, hints, results)` - `<sys/types.h>`, `<sys/socket.h>`, `<netdb.h>`
	* Used to lookup information about a network address and gets one or more matching `struct addrinfo` entries
	* `node` - String containing an IP address or hostname to lookup
		* If NULL, uses the local machine's IP address
	* `service` - String with a port number or service name (if the service is in `/etc/services`)
	* `hints` - Pointer to a `struct addrinfo` used to provide settings for the lookup (type of address, etc.)
	* `results` - Pointer to a `struct addrinfo` that will be a linked list containing entries for each matching address
		* `getaddrinfo` will allocate memory for the linked list
	* Example:
	```C
	struct addrinfo *hints, *results;
	hints = (struct addrinfo *)calloc(1, sizeof(struct addrinfo)); // heap isn't necessary, but it must be 0-set
	hints->ai_family = AF_INET;
	hints->ai_socktype = SOCK_STREAM; // TCP socket
	hints->ai_flags = AI_PASSIVE; // only needed on server
	getaddrinfo(NULL, "80", hints, &results); // server sets node to NULL
	// client: getaddrinfo("149.89.150.100", "9845", hints, &results);
	// do stuff
	free(hints);
	freeaddrinfo(results);
	```
* `bind(socket_descriptor, address, address_length)` - `<sys/socket.h>`
	* Binds the socket to an address and port (server only)
	* Returns 0 upon success and -1 upon failure
	* `socket_descriptor` - Return value of `socket`
	* `address` - Pointer to a `struct sockaddr` representing the address
	* `address_length` - Size of the address, in bytes
	* `address` and `address_length` can be retrieved from `getaddrinfo`
	* Example:
	```C
	// run the previous two examples
	bind(sd, results->ai_addr, results->ai_addrlen);
	```
* `listen(socket_descriptor, backlog)` - `<sys/socket.h>`
	* Sets a socket to passively wait for a connection (server only)
	* Needed for stream sockets
	* Doesn't block
	* `socket_descriptor` - Return value of `socket`
	* `backlog` - Number of connections that can be queued up
		* Depending on protocol, this might not do much

---

## 01/03/18: Socket to Me

#### Network Port
* Allows a single computer to run multiple services
* A socket combines an IP address and port
* Each computer has 2^16 (65,536) ports
* Some ports are reserved for specific services:
	* 80: http
	* 22: ssh
	* 443: ssl

#### Network Connection Types
* Stream Sockets
	* Reliable 2-way commnication
	* Must be connected on both ends
	* Data is received in the order it is sent (harder than it sounds)
	* Most use the Transmisson Control Protacol (TCP)
* Datagram Sockets
	* Connectionless; an established connection is not required
	* Data sent may be received out of order, if at all
	* Significantly faster; ironically, streaming services use datagram sockets
	* Uses the User Datagram Protocol (UDP)

---

## 01/02/18: Socket to Me

#### Socket
* A connection between 2 programs over a _network_
* A socket corresponds to an IP (internet protocol) Address/Port pair

#### To Use a Socket
1. Create the socket
2. Bind it to an address and port
3. Listen for/initiate a connection
4. Send/receive data

#### IP Address
* _All_ devices connected to the Internet have an IP address
* IP addresses come in two flavors: IPv4 and IPv6
* Addresses are allocated in blocks to make routing easier
* IPv4 uses 4 byte addresses of the following form:
	* `[0-255].[0-255].[0-255].[0-255]`
	* Each group is called an _octet_ (2^8)
	* There are at most 2^32, or ~4.3 billion IPv$ addresses
* IPv6 uses 16 byte addresses of the following form:
	* `[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]:[0-ffff]`
	* Each group is called a _hextet_ (2^16)
	* Leading 0s are ignored and any number of consecutive 0 hextets can be replaced with `::`
		* `0000 : 0000 : 0000 : 0000 : 004f : 13c2 : 0009 : a2d2 -> :: 4f : 13c2 : 9 : a2d2`
		* If there are multiple "strings" of consecutive 0 hextets, only one can undergo this replacement
	* IPv4 addresses can be represented as 5 0-hextets, 1 ffff hextet, and the IPv4 address
		* `149.89.150.100 -> :: ffff : 149.89.150.100`
	* There are at most 2^128 IPv6 addresses

---

## 12/18/17: Always tip your servers.

#### Forking server/client design pattern
* Server creates WKP (well-known pipe), client creates PP (private pipe)
* When two processes read from the same pipe, a random one actually gets the data

#### Basic server/client design pattern (now with forking!)
2. Handshake
	1. Client connects to server and sends the private FIFO name. Client waits for a response from the server.
	2. Server receives client's message and forks off a subserver.
	3. Subserver connects to client FIFO, sending an initial acknowledgement message.
	4. Client receives subserver's message, removes its PP.
3. Operation
	1. Server removes WKP and closes any connections to client.
	2. Server recreates WKP and waits for a new connection.
	3. Subserver and client send information back and forth.
```
SS < - - - S - - - > SS
|^       (WKP)       |^  
||                   ||
||                   ||
v|                   v|
C                     C
```

#### Downsides to this pattern
* No communication between processes outside of clients and individual subservers

---

## 12/11/17: Creating a handshake agreement.

#### DN
Consider a program that uses pipes in order to communicate between 2 separate executable files.

One is a "server" that is always running. The other is a "client".

Design a process by which both files can connect to each other and verify that each can send and receieve data. Try to keep it as simple as possible.

1. Client sends pre-arranged message, server checks message
2. Server sends pre-arranged response, client checks response
3. Client sends pre-arranged message to let the server know it received the response

#### Handshakes
* A procedure to ensure that a connection has been established between 2 programs
* Both ends of the connection must verify that they can send and receive data to and from each other
* 3-way handshake (see DN)
* Pipes must be FIFOs because they are connecting two entirely different programs

#### Basic server/client design pattern
1. Setup
	1. Server creates a well-known FIFO and waits for a connection.
	2. Client creates a "private" FIFO.
2. Handshake
	1. Client connects to server and sends the private FIFO name. Client waits for a response.
	2. Server receives client's message and removes the well-known pipe.
	3. Server connects to client's FIFO, sending an initial acknowledgement message.
	4. Client receives server's message and removes its private pipe.
	5. Client sends response to server.
3. Operation
	1. Server and client send information back and forth.
	2. ???
	3. Profit!
4. Reset
	1. Client exits, server closes any connections to the client.
	2. Server recreates the well-known pipe and wait for another client.

---

## 12/07/17: What's a semaphore? - To control resources!

* `semctl(descriptor, index, operation, data)`
	* `data` is declared as such:
	```C
	union semun {
		int val;               // used for SETVAL
		struct semid_ds *buf;  // used for IPC_STAT and IPC_SET
		unsigned short *array; // used for SETALL
		struct seminfo *__buf;
	};
	```
	* A `union` is a C structure designed to hold only one value at a time from a group of potential values
		* Contrast to a `struct`, which can hold multiple values simultaneously
* `semop(descriptor, operation, amount)`
	* Perform an atomic semaphore operation
	* You can Up/Down a semaphore by any integer value, not just 1
	* `operation` - A pointer to a `struct sembuf`:
	```C
	struct sembuf {
		short sem_op;
		short sem_num;
		short sem_flag;
		other stuff
	};
	```
	* `sem_num` - The index of the semaphore you want to work on
	* `sem_op` - Negative for `Down(S)`, positive for `Up(S)`, 0 for blocking until the semaphore reaches 0
	* `sem_flag`
		* `SEM_UNDO` - Allows the OS to undo the given operation, useful for if the program doesn't release it
		* `IPC_NOWAIT` - Instead of waiting for the semaphore to become available, return an error
	* `amount` - The amount of semaphores you want to operate on in the semaphore set

---

## 12/05/17: How do we flag down a resource?

#### DN
How would you control access to a shared resource like a file, pipe, or shared memory, such that you could ensure no read/write conflicts occurred?

#### Semaphores
* Created by Edsger Dijkstra
* IPC construct used to control access to shared resources
* More commonly used as a counter representing how many processes can access a resource simultaneously
	* A semaphore with a value of 3 can have 3 more active "users" (every new process accessing it decreases its value by 1)
	* A semaphore with a value of 0 is unavailable
* Semaphores are _atomic_, not split up into multiple processor instructions

#### Semaphore Operations
* Non-atomic (can be interrupted)
	* Create a semaphore
	* Set an initial value
	* Remove a semaphore
* Atomic (cannot be interrupted)
	* `Up(S) / V(S)`
		* Release the semaphore to signal you are done with its associated resource
		* Pseudocode: `S ++`
	* `Down(S) / P(S)`
		* Attempt to take the semaphore
		* If the semaphore is 0, wait for it to be available
		* Psuedocode: `while (!S) {block} S --;`

#### Semaphores in C
* `<sys/types.h>`, `<sys/ipc.h>`, `<sys/sem.h>`
* `semget(key, amount, flags)`
	* Create/Get access to a semaphore
	* Different from `Up(S)` or `Down(S)`, as it does not modify the semaphore
	* Returns a semaphore descriptor or -1 (errno)
	* `key` - Unique semaphore identifier (use `ftok`)
	* `amount` - Number of semaphores to create or get (semaphores are stored in sets of 1 or more)
	* `flags` - Includes permissions for the semaphore, combined with bitwise or
		* `IPC_CREAT` - Create the semaphore and set its value to 0
		* `IPC_EXCL` - Fail if the sempahore already exists and `IPC_CREAT` is on

---

## 12/04/17: Memes

#### DN
Why is the aim Memes?

Memes are sort of like shared memory, just between people and not processes

#### Shared Memory Continued
* `shmdt(pointer)`
	* Detach a variable from a shared memory segment
	* Returns 0 upon success and -1 upon failure
	* `pointer` - Address used to access the segment (return value of `shmat`)
* `shmctl(descriptor, commands, buffer)`
	* Performs various operations on the shared memory segment
	* A segment has metadata that can be stored in `struct shmid_ds`
		* Stored metadata includes last access date, size, PID of creator, PID of last modification
	* `descriptor` - Return value of `shmget`
	* `commands`
		* `IPC_RMID` - Remove a shared memory segment
		* `IPC_STAT` - Populate the `buffer` (`struct shmid_dis *`) with segment metadata
		* `IPC_SET` - Set some of the segment metadata from `buffer`
* `ipcs -m`
	* Terminal command that lists all of the current shared memory segments with metadata

---

## 12/01/17: Sharing is caring!

#### Shared Memory
* `<sys/shm.h>`, `<sys/ipc.h>`, `<sys/types.h>`
* A segment of memory that can be accessed by multiple processes
* Instead of the parent and child each storing a copy of a variable, they instead store pointers to the same thing
* Processes need to be given keys to access the shared memory
* _Not_ released when a program exits
* 5 shared memory operations
	* Create the segment (happens once)
	* Access the segment (once per process)
	* Attach the segment to a variable (once per process)
	* Detach the segment from a variable (once per process)
	* Remove the segment (happens once)
* `shmget(key, size, flags)`
	* Creates/Accesses a shared memory segment
	* Returns a shared memory descriptor (à la file descriptor) or -1 if it fails (errno)
	* `key` - Unique integer identifier for the shared memory segment (à la file name)
	* `size` - Number of bytes requested
	* `flags` - Are we creating a segment? Accessing? 3-digit octals, combined with | (bitwise or)
		* `IPC_CREAT` - Creates the segment. If segment is new, sets value to all 0s
		* `IPC_EXCL` - Fail if the segment already exists and `IPC_CREAT` is on
* `shmat(descriptor, address, flags)`
	* Attaches a shared memory segment to a variable
	* Returns a pointer to the segment, or -1 (errno)
	* `descriptor` - Return value of `shmget`
	* `address` - If 0, the OS will provide the appropriate address
	* `flags` - Usually 0, but `SHM_RDONLY` is useful and makes the memory read-only

---

## 11/28/17: C, the ultimate hipster, using # decades before it was cool

* \# provides preprocessor instructions, handled by gcc first
* `#include <LIBRARY> | "LIBRARY"` links libraries to your code
* `#define <NAME> <VALUE>` replaces all occurances of NAME with VALUE
	* Example: `define TRUE 1`
* Macros are like typeless functions: `#define SQUARE(x) x * x`
	* `int y = square(9) -> int y = 9 * 9`
	* `#define MIN(x, y) x < y ? x : y`
* Conditional statement
	```C
	#ifndef <IDENTIFIER>
	<CODE>
	#endif
	```
	* If the identifier has been defined ignore all code until the last endif

---

## 11/27/17: Redirection; how does it ... SQUIRREL

* Changing the usual input/output behavior of a program
* `ps > ps_file` creates a new file and puts the output of `ps` into that file, rather than the terminal
* In general, `<COMMAND> > <FILE NAME>` redirects stdout to a file and overwrites its contents
	* `>>` also redirects stdout to a file, but appends it to the end
* `2>` redirects stderr to a file and overwrites it (`2>>`  appends)
* `&>` redirects stdout and stderr (`&>>` appends)
* `<` redirects stdin from a file
* `|` (pipe) redirects stdout from one command to stdin of the next
	* `ls | wc` takes the output of ls and feeds it into wc

#### Redirection in C Programs
* `dup(fd)` - `<unistd.h>`
	* Duplicates an existing entry in the file table (opens it again)
	* Returns a new file descriptor for the duplicate entry
* `dup2(fd1, fd2)` - `<unistd.h>`
	* Redirects fd2 to fd1
	* Duplicates the behavior for fd1 at fd2
	* fd2 loses its original reference; the file gets closed
	* Use it in conjunction with `dup`

---

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

#### Pipe
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

#### Managing Sub-processes
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

#### The `exec` Family - `<unistd.h>`
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

#### Signals
* Limited way of sending information to a process
* `kill`
	* Command line utility to send a signal to a process
	* `$ kill <PID>` sends signal 15 (SIGTERM) to the specified PID
	* `$ kill -<SIGNAL> <PID>` sends a specific signal to the specified PID
* `killall [-<SIGNAL>] <PROCESS>`
	* Sends SIGTERM or SIGNAL to all processes with the specified name
* CTRL + C sends SIGINT and interrupts the process

#### Signal Handling in C
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

#### More Command Line Arguments
* `fgets(<DESTINATION>, <BYTES>, <FILE POINTER>)` - `<stdio.h>`
	* Reads in from a file stream and stores it in a string
	* Reads at most \<BYTES> - 1 characters from the pointer (adds a null at the end), including newlines
	* File pointer is type `FILE *`, more complex than a file descriptor
		* `stdin` is a file pointer
	* Stops at newlines, end of file, or byte limit
	* What if you wanted to read in numbers? Just use...
* `sscanf(<SOURCE STRING>, <FORMAT STRING>, <VAR 1>, <VAR 2>, ...)` - `<stdio.h>`
	* Scans a string and extracts values based on a format string

#### Processes
* Every running program is a process
* Programs can create subprocesses, but these are the same as regular processes
* A processor can handle 1 process per cycle or per core
* "Multitasking" only appears to happen due to the processor switching between all the active processes quickly

#### PID
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

#### Command Line Arguments
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
