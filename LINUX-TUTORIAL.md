# PForth Linux System Integration Tutorial

**Ein umfassender Leitfaden für die Integration von PForth mit dem Linux-System**

---

## Überblick

PForth unter Linux ist ein vollwertiger Linux-Prozess mit Zugang zu allen Systemfunktionen. Dieses Tutorial erklärt, wie Sie PForth nutzen können, um direkt mit dem Linux-Kernel zu kommunizieren, mit anderen Prozessen zu interagieren und die vollständige Power des Linux-Systems zu nutzen.

---

## Systemfunktionen und glibc Verfügbarkeit

### Standard C Library Integration

**Ja, die glibc ist vollständig verfügbar!** PForth läuft als normaler Linux-Prozess und hat Zugang zu allen Standard-C-Bibliotheksfunktionen:

```c
// PForth nutzt normale C-Bibliotheksfunktionen:
#include <unistd.h>     // POSIX system calls
#include <termios.h>    // Terminal control
#include <sys/poll.h>   // I/O multiplexing
#include <stdlib.h>     // malloc, free
#include <stdio.h>      // printf, fopen etc.
#include <sys/socket.h> // Network programming
#include <pthread.h>    // Threading
#include <signal.h>     // Signal handling
```

### Verfügbare Systemressourcen

PForth hat als Linux-Prozess Zugang zu:
- ✅ Alle glibc-Funktionen
- ✅ System calls via glibc wrappers
- ✅ POSIX-konforme APIs
- ✅ Linux-spezifische Funktionen
- ✅ Hardware-nahe Programmierung (mit Privilegien)
- ✅ Netzwerk-APIs
- ✅ Threading und IPC

---

## Memory Map eines PForth-Prozesses

### Typische Linux Process Memory Layout

```
Linux Process Memory Map:

0x7fff00000000  ┌─────────────────┐
                │      Stack      │  ← Return Stack, Data Stack
                │                 │  ← Lokale Variablen
                ├─────────────────┤
                │      ...        │  ← Memory Gap
                ├─────────────────┤
0x7f0000000000  │   Shared Libs   │  ← glibc, libm, libpthread
                │   (libc.so.6)   │  ← ld-linux.so
                │   (libm.so.6)   │  ← weitere .so Dateien
                ├─────────────────┤
                │      ...        │  ← Memory Gap
                ├─────────────────┤
0x600000000000  │      Heap       │  ← Dictionary Memory (malloc)
                │  (malloc'd)     │  ← Custom Allocator Pool
                │                 │  ← Dynamische Datenstrukturen
                ├─────────────────┤
0x400000000000  │   Text/Data     │  ← PForth Executable
                │    .text        │  ← C Kernel Code
                │    .rodata      │  ← Konstanten, Strings
                │    .data        │  ← Initialisierte Globals
                │    .bss         │  ← Uninitialisierte Globals
0x000000000000  └─────────────────┘
```

### PForth-spezifische Memory Segmente

```c
// Typische PForth Memory Allocation:
typedef struct {
    // Dictionary im Heap
    ucell_t *dic_HeaderBase;    // malloc'd Header Space
    ucell_t *dic_CodeBase;      // malloc'd Code Space
    
    // Stacks als Arrays
    cell_t  *DataStack;         // malloc'd oder statisch
    cell_t  *ReturnStack;       // malloc'd oder statisch
    
    // System Memory
    char    *MemoryPool;        // Custom Allocator Pool
} PForthMemory;
```

### Memory-Debugging

```bash
# Memory Map des laufenden PForth-Prozesses anzeigen
cat /proc/$(pgrep pforth)/maps

# Memory Usage
cat /proc/$(pgrep pforth)/status | grep -E "Vm|Peak"

# Heap-Analyse mit Valgrind
valgrind --tool=massif pforth

# Memory Leaks aufspüren
valgrind --leak-check=full pforth
```

---

## Kernel-Kommunikation

### Custom C Functions für System Calls

PForth kann über Custom C Functions alle System Calls nutzen:

```c
// custom_linux.c - Erweiterte Linux-Integration
#include "pf_all.h"
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/mman.h>
#include <signal.h>
#include <fcntl.h>
#include <unistd.h>

// File Operations
static cell_t ForthOpen(cell_t filename_addr, cell_t flags, cell_t mode) {
    char* filename = (char*)filename_addr;
    return open(filename, flags, mode);
}

static cell_t ForthRead(cell_t fd, cell_t buffer_addr, cell_t count) {
    return read(fd, (void*)buffer_addr, count);
}

static cell_t ForthWrite(cell_t fd, cell_t buffer_addr, cell_t count) {
    return write(fd, (void*)buffer_addr, count);
}

static cell_t ForthClose(cell_t fd) {
    return close(fd);
}

// Process Management
static cell_t ForthFork(void) { 
    return fork(); 
}

static cell_t ForthExecve(cell_t path, cell_t argv, cell_t envp) {
    return execve((char*)path, (char**)argv, (char**)envp);
}

static cell_t ForthWaitpid(cell_t pid, cell_t status_addr, cell_t options) {
    return waitpid(pid, (int*)status_addr, options);
}

static cell_t ForthGetpid(void) {
    return getpid();
}

// Memory Management
static cell_t ForthMmap(cell_t addr, cell_t len, cell_t prot, 
                       cell_t flags, cell_t fd, cell_t offset) {
    return (cell_t)mmap((void*)addr, len, prot, flags, fd, offset);
}

static cell_t ForthMunmap(cell_t addr, cell_t len) {
    return munmap((void*)addr, len);
}

// Signal Handling
static cell_t ForthKill(cell_t pid, cell_t sig) {
    return kill(pid, sig);
}

static cell_t ForthSignal(cell_t sig, cell_t handler) {
    return (cell_t)signal(sig, (void(*)(int))handler);
}

// Directory Operations
static cell_t ForthMkdir(cell_t path, cell_t mode) {
    return mkdir((char*)path, mode);
}

static cell_t ForthRmdir(cell_t path) {
    return rmdir((char*)path);
}

// Custom Function Table - WICHTIG: Name nicht ändern!
CFunc0 CustomFunctionTable[] = {
    (CFunc0) ForthOpen,     // Index 0
    (CFunc0) ForthRead,     // Index 1  
    (CFunc0) ForthWrite,    // Index 2
    (CFunc0) ForthClose,    // Index 3
    (CFunc0) ForthFork,     // Index 4
    (CFunc0) ForthExecve,   // Index 5
    (CFunc0) ForthWaitpid,  // Index 6
    (CFunc0) ForthGetpid,   // Index 7
    (CFunc0) ForthMmap,     // Index 8
    (CFunc0) ForthMunmap,   // Index 9
    (CFunc0) ForthKill,     // Index 10
    (CFunc0) ForthSignal,   // Index 11
    (CFunc0) ForthMkdir,    // Index 12
    (CFunc0) ForthRmdir,    // Index 13
    NULL  // Terminator
};

// Custom Function Registration
Err CompileCustomFunctions( void ) {
    Err err;
    
    // File Operations
    err = CreateGlueToC("LINUX.OPEN",   0, C_RETURNS_VALUE, 3); if(err) return err;
    err = CreateGlueToC("LINUX.READ",   1, C_RETURNS_VALUE, 3); if(err) return err;
    err = CreateGlueToC("LINUX.WRITE",  2, C_RETURNS_VALUE, 3); if(err) return err;
    err = CreateGlueToC("LINUX.CLOSE",  3, C_RETURNS_VALUE, 1); if(err) return err;
    
    // Process Management  
    err = CreateGlueToC("LINUX.FORK",   4, C_RETURNS_VALUE, 0); if(err) return err;
    err = CreateGlueToC("LINUX.EXECVE", 5, C_RETURNS_VALUE, 3); if(err) return err;
    err = CreateGlueToC("LINUX.WAITPID",6, C_RETURNS_VALUE, 3); if(err) return err;
    err = CreateGlueToC("LINUX.GETPID", 7, C_RETURNS_VALUE, 0); if(err) return err;
    
    // Memory Management
    err = CreateGlueToC("LINUX.MMAP",   8, C_RETURNS_VALUE, 6); if(err) return err;
    err = CreateGlueToC("LINUX.MUNMAP", 9, C_RETURNS_VALUE, 2); if(err) return err;
    
    // Signal Handling
    err = CreateGlueToC("LINUX.KILL",   10, C_RETURNS_VALUE, 2); if(err) return err;
    err = CreateGlueToC("LINUX.SIGNAL", 11, C_RETURNS_VALUE, 2); if(err) return err;
    
    // Directory Operations
    err = CreateGlueToC("LINUX.MKDIR",  12, C_RETURNS_VALUE, 2); if(err) return err;
    err = CreateGlueToC("LINUX.RMDIR",  13, C_RETURNS_VALUE, 1); if(err) return err;
    
    return 0;
}
```

### Forth Interface für System Programming

Erstellen Sie eine Forth-Bibliothek für Linux-System-Integration:

```forth
\ linux-system.fth - Linux System Integration Library
ANEW TASK-LINUX-SYSTEM.FTH

\ File Operation Constants
33188 CONSTANT O_RDONLY    \ 0
33189 CONSTANT O_WRONLY    \ 1  
33190 CONSTANT O_RDWR      \ 2
33792 CONSTANT O_CREAT     \ 64
34816 CONSTANT O_TRUNC     \ 512
35840 CONSTANT O_APPEND    \ 1024

\ File Permissions
256 CONSTANT S_IRUSR       \ User read
128 CONSTANT S_IWUSR       \ User write
64  CONSTANT S_IXUSR       \ User execute
511 CONSTANT S_IRWXU       \ User all permissions

\ Memory Protection Constants
1 CONSTANT PROT_READ
2 CONSTANT PROT_WRITE  
4 CONSTANT PROT_EXEC
0 CONSTANT PROT_NONE

\ Memory Mapping Constants
1   CONSTANT MAP_SHARED
2   CONSTANT MAP_PRIVATE
32  CONSTANT MAP_ANONYMOUS
16  CONSTANT MAP_FIXED

\ Signal Constants
2  CONSTANT SIGINT
9  CONSTANT SIGKILL
15 CONSTANT SIGTERM
17 CONSTANT SIGCHLD

\ ========================================
\ File Operations
\ ========================================

: OPEN.FILE ( filename-addr flags mode -- fd )
    LINUX.OPEN
    DUP 0< IF
        ." Error opening file, errno: " . CR
    THEN
;

: READ.FILE ( fd buffer-addr count -- bytes-read )
    LINUX.READ
    DUP 0< IF
        ." Error reading file, errno: " . CR
    THEN  
;

: WRITE.FILE ( fd buffer-addr count -- bytes-written )
    LINUX.WRITE
    DUP 0< IF
        ." Error writing file, errno: " . CR
    THEN
;

: CLOSE.FILE ( fd -- result )
    LINUX.CLOSE
    DUP 0< IF
        ." Error closing file, errno: " . CR
    THEN
;

\ Convenience words
: CREATE.FILE ( filename-addr mode -- fd )
    SWAP O_CREAT O_WRONLY OR O_TRUNC OR SWAP
    OPEN.FILE
;

: READ.TEXT.FILE ( filename-addr -- )
    O_RDONLY 0 OPEN.FILE DUP 0< IF
        DROP ." Cannot open file" CR EXIT
    THEN
    
    >R  \ Save fd on return stack
    CREATE buffer 1024 ALLOT
    
    BEGIN
        R@ buffer 1024 READ.FILE
        DUP 0>
    WHILE
        buffer SWAP TYPE
    REPEAT
    DROP
    
    R> CLOSE.FILE DROP
;

\ ========================================
\ Process Management  
\ ========================================

: FORK.PROCESS ( -- pid )
    LINUX.FORK
    DUP 0< IF
        ." Fork failed, errno: " . CR
    THEN
;

: GET.PID ( -- pid )
    LINUX.GETPID
;

: EXEC.PROGRAM ( path argv envp -- result )
    LINUX.EXECVE
    \ If we return here, exec failed
    DUP 0< IF
        ." Exec failed, errno: " . CR
    THEN
;

: WAIT.FOR.CHILD ( pid options -- exit-status )
    CREATE status-buffer 4 ALLOT
    status-buffer SWAP LINUX.WAITPID
    DUP 0< IF
        ." Wait failed, errno: " . CR
        DROP 0
    ELSE
        DROP status-buffer @
    THEN
;

: KILL.PROCESS ( pid signal -- result )
    LINUX.KILL
    DUP 0< IF
        ." Kill failed, errno: " . CR
    THEN
;

\ ========================================
\ Memory Management
\ ========================================

: MAP.MEMORY ( addr length prot flags fd offset -- mapped-addr )
    LINUX.MMAP
    DUP -1 = IF
        ." Memory mapping failed" CR
    THEN
;

: UNMAP.MEMORY ( addr length -- result )
    LINUX.MUNMAP
    DUP 0< IF
        ." Memory unmapping failed, errno: " . CR
    THEN
;

: ALLOC.ANONYMOUS.MEMORY ( size -- addr )
    0 SWAP PROT_READ PROT_WRITE OR 
    MAP_PRIVATE MAP_ANONYMOUS OR 
    -1 0 MAP.MEMORY
;

\ ========================================
\ Directory Operations  
\ ========================================

: MAKE.DIRECTORY ( path-addr mode -- result )
    LINUX.MKDIR
    DUP 0< IF
        ." mkdir failed, errno: " . CR
    THEN
;

: REMOVE.DIRECTORY ( path-addr -- result )
    LINUX.RMDIR
    DUP 0< IF
        ." rmdir failed, errno: " . CR
    THEN
;

\ ========================================
\ High-Level Examples
\ ========================================

\ Example: Create child process
: START.CHILD.PROCESS ( -- child-pid )
    FORK.PROCESS DUP 0= IF
        \ In child process
        ." Child process started with PID: " GET.PID . CR
        \ Do something in child...
        ." Child process exiting" CR
        1 EXIT  \ Exit child process
    ELSE  
        \ In parent process
        DUP 0> IF
            ." Parent: Child PID is " . CR
        ELSE
            ." Fork failed!" CR
        THEN
    THEN
;

\ Example: File copy
: COPY.FILE ( src-name dest-name -- )
    >R  \ Save dest name
    O_RDONLY 0 OPEN.FILE DUP 0< IF
        DROP R> DROP ." Cannot open source file" CR EXIT
    THEN
    
    R> S_IRUSR S_IWUSR OR CREATE.FILE DUP 0< IF
        SWAP CLOSE.FILE DROP
        DROP ." Cannot create destination file" CR EXIT  
    THEN
    
    \ src-fd dest-fd on stack
    CREATE copy-buffer 4096 ALLOT
    
    BEGIN
        OVER copy-buffer 4096 READ.FILE  \ Read from source
        DUP 0>
    WHILE
        2 PICK copy-buffer ROT WRITE.FILE DROP  \ Write to dest
    REPEAT
    DROP
    
    \ Close both files
    CLOSE.FILE DROP
    CLOSE.FILE DROP
    ." File copied successfully" CR
;

\ Example: Simple shell command
: SYSTEM.COMMAND ( command-string -- exit-status )
    FORK.PROCESS DUP 0= IF
        \ In child - execute command
        DROP S" /bin/sh" DROP 
        CREATE argv 3 CELLS ALLOT
        S" /bin/sh" DROP argv !
        S" -c" DROP argv CELL+ !
        argv 2 CELLS + !  \ command string
        0 argv 3 CELLS + !  \ NULL terminator
        
        argv 0 EXEC.PROGRAM
        ." Exec failed" CR
        127 EXIT  \ Should not reach here
    ELSE
        \ In parent - wait for child
        DUP 0> IF
            0 WAIT.FOR.CHILD
        ELSE
            ." Fork failed" CR
            -1
        THEN
    THEN
;

\ Example: Memory mapped file
: MAP.FILE ( filename -- addr size )
    O_RDONLY 0 OPEN.FILE DUP 0< IF
        DROP 0 0 ." Cannot open file" CR EXIT
    THEN
    
    \ Get file size (simplified - should use stat)
    DUP 0 2 SEEK.FILE  \ Seek to end (if SEEK.FILE exists)
    \ For now, assume 4096 bytes
    4096 CONSTANT file-size
    
    \ Map the file
    0 file-size PROT_READ MAP_SHARED 
    4 PICK 0 MAP.MEMORY
    
    \ Close file descriptor (mapping remains)
    SWAP CLOSE.FILE DROP
    
    file-size
;

\ ========================================
\ Debugging and System Information
\ ========================================

: SHOW.PID ( -- )
    ." Current PID: " GET.PID . CR
;

: SEND.SIGNAL.TO.SELF ( signal -- )
    GET.PID SWAP KILL.PROCESS DROP
;

: CREATE.TEST.FILE ( -- )
    S" /tmp/pforth-test.txt" DROP 
    S_IRUSR S_IWUSR OR CREATE.FILE DUP 0< IF
        DROP ." Cannot create test file" CR EXIT
    THEN
    
    S" Hello from PForth on Linux!" DROP DUP STRLEN
    2 PICK SWAP WRITE.FILE DROP
    CLOSE.FILE DROP
    ." Test file created in /tmp/pforth-test.txt" CR
;

\ Usage examples:
\ S" /etc/passwd" DROP READ.TEXT.FILE
\ S" /tmp/test" DROP S_IRWXU MAKE.DIRECTORY DROP
\ S" ls -la" DROP SYSTEM.COMMAND .
\ CREATE.TEST.FILE
```

---

## Inter-Process Communication (IPC)

### Pipes und FIFOs

```c
// Pipe-Funktionen hinzufügen
static cell_t ForthPipe(cell_t pipefd_addr) {
    int* pipefd = (int*)pipefd_addr;
    return pipe(pipefd);
}

static cell_t ForthMkfifo(cell_t path, cell_t mode) {
    return mkfifo((char*)path, mode);
}
```

```forth
\ Named Pipes (FIFOs)
: CREATE.FIFO ( path-addr mode -- result )
    LINUX.MKFIFO
;

: PIPE.EXAMPLE ( -- )
    \ Create pipe
    CREATE pipe-fds 2 CELLS ALLOT
    pipe-fds LINUX.PIPE 0< IF
        ." Pipe creation failed" CR EXIT
    THEN
    
    FORK.PROCESS DUP 0= IF
        \ Child process - writer
        pipe-fds @ CLOSE.FILE DROP  \ Close read end
        pipe-fds CELL+ @            \ Get write end
        S" Hello from child!" DROP DUP STRLEN
        ROT SWAP WRITE.FILE DROP
        CLOSE.FILE DROP
        0 EXIT
    ELSE
        \ Parent process - reader  
        pipe-fds CELL+ @ CLOSE.FILE DROP  \ Close write end
        pipe-fds @                        \ Get read end
        CREATE buffer 100 ALLOT
        buffer 100 READ.FILE
        ." Received: " buffer SWAP TYPE CR
        CLOSE.FILE DROP
        0 WAIT.FOR.CHILD DROP
    THEN
;
```

### Unix Domain Sockets

```c
#include <sys/socket.h>
#include <sys/un.h>

static cell_t ForthSocket(cell_t domain, cell_t type, cell_t protocol) {
    return socket(domain, type, protocol);
}

static cell_t ForthBind(cell_t sockfd, cell_t addr, cell_t addrlen) {
    return bind(sockfd, (struct sockaddr*)addr, addrlen);
}

static cell_t ForthListen(cell_t sockfd, cell_t backlog) {
    return listen(sockfd, backlog);
}

static cell_t ForthAccept(cell_t sockfd, cell_t addr, cell_t addrlen) {
    return accept(sockfd, (struct sockaddr*)addr, (socklen_t*)addrlen);
}

static cell_t ForthConnect(cell_t sockfd, cell_t addr, cell_t addrlen) {
    return connect(sockfd, (struct sockaddr*)addr, addrlen);
}
```

### Shared Memory

```c
#include <sys/ipc.h>
#include <sys/shm.h>

static cell_t ForthShmget(cell_t key, cell_t size, cell_t shmflg) {
    return shmget(key, size, shmflg);
}

static cell_t ForthShmat(cell_t shmid, cell_t shmaddr, cell_t shmflg) {
    return (cell_t)shmat(shmid, (void*)shmaddr, shmflg);
}

static cell_t ForthShmdt(cell_t shmaddr) {
    return shmdt((void*)shmaddr);
}
```

---

## Network Programming

### TCP Sockets

```c
#include <netinet/in.h>
#include <arpa/inet.h>

static cell_t ForthInetAddr(cell_t cp) {
    return inet_addr((char*)cp);
}

static cell_t ForthHtons(cell_t hostshort) {
    return htons(hostshort);
}

static cell_t ForthNtohs(cell_t netshort) {
    return ntohs(netshort);
}
```

```forth
\ Network Constants
1 CONSTANT AF_UNIX
2 CONSTANT AF_INET
1 CONSTANT SOCK_STREAM
2 CONSTANT SOCK_DGRAM

\ TCP Client Example
: TCP.CLIENT ( ip-string port -- socket )
    CREATE sockaddr 16 ALLOT
    
    AF_INET SOCK_STREAM 0 LINUX.SOCKET DUP 0< IF
        ." Socket creation failed" CR
        DROP 0 EXIT
    THEN
    
    \ Setup sockaddr_in structure
    AF_INET sockaddr W!                    \ sin_family
    LINUX.HTONS sockaddr 2 + W!           \ sin_port
    LINUX.INET.ADDR sockaddr 4 + !        \ sin_addr
    
    \ Connect
    DUP sockaddr 16 LINUX.CONNECT 0< IF
        ." Connection failed" CR
        CLOSE.FILE DROP 0
    THEN
;

\ TCP Server Example  
: TCP.SERVER ( port -- )
    CREATE sockaddr 16 ALLOT
    
    \ Create socket
    AF_INET SOCK_STREAM 0 LINUX.SOCKET DUP 0< IF
        ." Socket creation failed" CR
        DROP EXIT
    THEN
    
    \ Setup sockaddr_in
    AF_INET sockaddr W!                    \ sin_family
    LINUX.HTONS sockaddr 2 + W!           \ sin_port  
    0 sockaddr 4 + !                      \ INADDR_ANY
    
    \ Bind
    DUP sockaddr 16 LINUX.BIND 0< IF
        ." Bind failed" CR
        CLOSE.FILE DROP EXIT
    THEN
    
    \ Listen
    DUP 5 LINUX.LISTEN 0< IF
        ." Listen failed" CR
        CLOSE.FILE DROP EXIT
    THEN
    
    ." Server listening on port " OVER . CR
    
    \ Accept connections
    BEGIN
        DUP 0 0 LINUX.ACCEPT DUP 0>
    WHILE
        ." New connection accepted" CR
        \ Handle client...
        CLOSE.FILE DROP
    REPEAT
    DROP CLOSE.FILE DROP
;
```

---

## Threading mit pthreads

```c
#include <pthread.h>

static cell_t ForthPthreadCreate(cell_t thread, cell_t attr, 
                                cell_t start_routine, cell_t arg) {
    return pthread_create((pthread_t*)thread, (pthread_attr_t*)attr,
                         (void*(*)(void*))start_routine, (void*)arg);
}

static cell_t ForthPthreadJoin(cell_t thread, cell_t retval) {
    return pthread_join(*(pthread_t*)thread, (void**)retval);
}

static cell_t ForthPthreadMutexInit(cell_t mutex, cell_t attr) {
    return pthread_mutex_init((pthread_mutex_t*)mutex, 
                             (pthread_mutexattr_t*)attr);
}

static cell_t ForthPthreadMutexLock(cell_t mutex) {
    return pthread_mutex_lock((pthread_mutex_t*)mutex);
}

static cell_t ForthPthreadMutexUnlock(cell_t mutex) {
    return pthread_mutex_unlock((pthread_mutex_t*)mutex);
}
```

---

## Debugging und Monitoring

### Verfügbare Linux-Tools

```bash
# Memory Map des laufenden PForth-Prozesses
cat /proc/$(pgrep pforth)/maps

# System Calls in Echtzeit verfolgen
strace -f -e trace=all pforth

# Library Dependencies anzeigen  
ldd pforth

# Performance Profiling
perf record -g pforth
perf report

# Memory Leak Detection
valgrind --leak-check=full --show-leak-kinds=all pforth

# CPU Usage monitoring
top -p $(pgrep pforth)

# File Descriptor monitoring
lsof -p $(pgrep pforth)

# Network Connections
netstat -p | grep pforth
```

### Forth-basierte Debugging-Tools

```forth
\ System Information
: SHOW.MEMORY.INFO ( -- )
    ." Process ID: " GET.PID . CR
    ." Memory map: /proc/" GET.PID . ." /maps" CR
    ." Status: /proc/" GET.PID . ." /status" CR
;

\ Signal Handler für Debugging
: SETUP.DEBUG.SIGNALS ( -- )
    \ SIGUSR1 für Stack-Dump
    10 ' .S LINUX.SIGNAL DROP
    
    \ SIGUSR2 für Dictionary-Info
    12 ' WORDS LINUX.SIGNAL DROP
    
    ." Debug signals installed:" CR
    ." kill -USR1 " GET.PID . ."  -> Stack dump" CR  
    ." kill -USR2 " GET.PID . ."  -> Word list" CR
;

\ Performance Measurement
: BENCHMARK.SYSCALLS ( count -- time-ms )
    TIMER-START
    0 DO
        GET.PID DROP
    LOOP
    TIMER-STOP
;
```

---

## Build-Integration

### Erweiterte Makefile-Konfiguration

```makefile
# Makefile für erweiterte Linux-Integration
EXTRA_CFLAGS = -DPF_USER_CUSTOM=\"custom_linux.c\" \
               -DPF_SUPPORT_FP \
               -D_GNU_SOURCE

EXTRA_LIBS = -lpthread -lrt -lm

# Custom Linux Build Target
pforth-linux: $(PFOBJS) custom_linux.o
	$(CC) -o $@ $(PFOBJS) custom_linux.o $(EXTRA_LIBS)

custom_linux.o: custom_linux.c
	$(CC) $(CFLAGS) $(EXTRA_CFLAGS) -c $< -o $@

# Installation mit System-Integration
install-system: pforth-linux
	sudo cp pforth-linux /usr/local/bin/pforth
	sudo cp linux-system.fth /usr/local/share/pforth/
	sudo chmod +x /usr/local/bin/pforth
```

### CMake-Konfiguration

```cmake
# CMakeLists.txt für Linux-Integration
cmake_minimum_required(VERSION 3.6)
project(PForth-Linux)

# Linux-spezifische Optionen
option(PF_LINUX_INTEGRATION "Enable Linux system integration" ON)

if(PF_LINUX_INTEGRATION)
    add_definitions(-DPF_USER_CUSTOM="custom_linux.c")
    add_definitions(-D_GNU_SOURCE)
    
    # Threading support
    find_package(Threads REQUIRED)
    target_link_libraries(pforth Threads::Threads)
    
    # Real-time extensions
    target_link_libraries(pforth rt)
endif()
```

---

## Praktische Anwendungsbeispiele

### 1. System-Monitor in Forth

```forth
\ system-monitor.fth
: MONITOR.SYSTEM ( interval-seconds -- )
    BEGIN
        CLEAR.SCREEN
        ." === PForth System Monitor ===" CR CR
        
        ." PID: " GET.PID . CR
        ." Time: " TIME&DATE . . . . . . CR
        
        \ CPU und Memory Info aus /proc lesen
        S" /proc/meminfo" DROP READ.SYSTEM.FILE
        
        DUP SLEEP.SECONDS
        ?TERMINAL
    UNTIL
    DROP
;

\ Usage: 5 MONITOR.SYSTEM
```

### 2. Network Server

```forth
\ http-server.fth  
: SIMPLE.HTTP.SERVER ( port -- )
    TCP.SERVER
    \ Implementation für HTTP-Request handling
;

\ Usage: 8080 SIMPLE.HTTP.SERVER
```

### 3. Inter-Process Communication

```forth
\ ipc-example.fth
: MASTER.PROCESS ( -- )
    \ Create named pipe
    S" /tmp/pforth-pipe" DROP 0o666 CREATE.FIFO DROP
    
    \ Fork worker processes
    5 0 DO
        FORK.PROCESS 0= IF
            \ Worker process
            I WORKER.PROCESS
            0 EXIT
        THEN
    LOOP
    
    \ Master coordination logic
    COORDINATE.WORKERS
;

: WORKER.PROCESS ( worker-id -- )
    ." Worker " . ." started" CR
    \ Worker implementation
;
```

---

## Sicherheitsüberlegungen

### Memory Protection

```forth
\ Sichere Memory-Operationen
: SAFE.@ ( addr -- value | 0 )
    ['] @ CATCH IF
        ." Invalid memory access at " . CR
        0
    THEN
;

: VALIDATE.POINTER ( addr -- addr | error )
    DUP 0= IF
        ." Null pointer error" CR
        ABORT
    THEN
    
    \ Weitere Validierung...
;
```

### Input Validation

```forth
: SAFE.SYSTEM.COMMAND ( command-string -- )
    \ Validate command string
    DUP C@ 0= IF
        DROP ." Empty command" CR EXIT
    THEN
    
    \ Check for dangerous characters
    DUP S" ;" SEARCH IF
        ." Dangerous command detected" CR
        2DROP DROP EXIT
    THEN
    2DROP
    
    SYSTEM.COMMAND DROP
;
```

---

## Zusammenfassung

PForth unter Linux bietet **vollständige System-Integration**:

### ✅ Verfügbare Features:
- **Vollständiger glibc-Zugang** über Custom C Functions
- **Alle Linux System Calls** nutzbar
- **Inter-Process Communication** via Pipes, Sockets, Shared Memory
- **Network Programming** mit TCP/UDP Sockets
- **Threading** mit pthreads
- **Signal Handling** für Event-driven Programming
- **Memory Mapping** für Performance-kritische Anwendungen
- **File System Operations** mit vollständiger Kontrolle

### 🎯 Anwendungsgebiete:
- **System Administration** Tools
- **Network Services** und Server
- **Embedded System** Prototyping
- **Real-time Applications**
- **Cross-Process Communication**
- **Hardware-nahe Programmierung**

### 🚀 Vorteile:
- **Direkte Kernel-Kommunikation** ohne Wrapper
- **Minimaler Overhead** durch C-Integration
- **Vollständige Linux-Kompatibilität**
- **Einfache Erweiterbarkeit** durch Custom Functions
- **Debugging-freundlich** mit Standard Linux-Tools

PForth ist somit nicht nur ein Forth-Interpreter, sondern eine **vollwertige Linux-Entwicklungsplattform** für System-nahe Programmierung!