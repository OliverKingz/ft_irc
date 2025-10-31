# Funciones Permitidas para ft_irc

## **Socket Functions**

- **socket**: Creates an endpoint for communication.
- **setsockopt**: Sets socket options.
- **getsockname**: Gets socket name.
- **getprotobyname**: Gets protocol entry by name.
- **gethostbyname**: Gets host entry by name.
- **getaddrinfo**: Gets address information.
- **freeaddrinfo**: Frees address information.
- **bind**: Binds a name to a socket.
- **connect**: Connects a socket.
- **listen**: Listens for connections on a socket.
- **accept**: Accepts a connection on a socket.

```c
int socket(int domain, int type, int protocol);
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
int getsockname(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
struct protoent *getprotobyname(const char *name);
struct hostent *gethostbyname(const char *name);
int getaddrinfo(const char *node, const char *service, const struct addrinfo *hints, struct addrinfo **res);
void freeaddrinfo(struct addrinfo *res);
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
int listen(int sockfd, int backlog);
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

## **Network Conversion Functions**

- **htons**: Converts host to network short (port).
- **htonl**: Converts host to network long (address).
- **ntohs**: Converts network to host short (port).
- **ntohl**: Converts network to host long (address).
- **inet_addr**: Converts IP string to binary.
- **inet_ntoa**: Converts binary IP to string.
- **inet_ntop**: Converts binary IP to string (modern).

```c
uint16_t htons(uint16_t hostshort);
uint32_t htonl(uint32_t hostlong);
uint16_t ntohs(uint16_t netshort);
uint32_t ntohl(uint32_t netlong);
in_addr_t inet_addr(const char *cp);
char *inet_ntoa(struct in_addr in);
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

## **Communication Functions**

- **send**: Sends data over socket.
- **recv**: Receives data from socket.

```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

## **Signal Functions**

- **signal**: Sets signal handler.
- **sigaction**: Examines and changes signal action.
- **sigemptyset**: Initializes empty signal set.
- **sigfillset**: Fills signal set with all signals.
- **sigaddset**: Adds signal to signal set.
- **sigdelset**: Removes signal from signal set.
- **sigismember**: Tests whether signal is in set.

```c
void (*signal(int signum, void (*handler)(int)))(int);
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);
int sigismember(const sigset_t *set, int signum);
```

## **File Operations**

- **close**: Closes a file descriptor.
- **lseek**: Repositions file offset.
- **fstat**: Gets file status.
- **fcntl**: Manipulates file descriptor.

```c
int close(int fd);
off_t lseek(int fd, off_t offset, int whence);
int fstat(int fd, struct stat *statbuf);
int fcntl(int fd, int cmd, ... /* arg */ );
```

## **I/O Multiplexing**

- **poll**: Waits for events on multiple file descriptors.
- **select**: Waits for events on sets of file descriptors (portable, POSIX).
- **kqueue (BSD / macOS)**: Kernel event notification interface supporting many event types (I/O, timers, signals, etc.). Scales well for large numbers of descriptors.
- **epoll (Linux)**: Linux-specific scalable event facility. Uses an interest list manipulated with epoll_ctl and notifies readiness with epoll_wait. Supports level- and edge-triggered modes and flags like EPOLLONESHOT / EPOLLET.

| Feature                   |                                                                 select() |                                            poll() |                                                                                                                         kqueue |                                                                                              epoll |
| ------------------------- | -----------------------------------------------------------------------: | ------------------------------------------------: | -----------------------------------------------------------------------------------------------------------------------------: | -------------------------------------------------------------------------------------------------: |
| Portability               |                 Portable across Unix-like systems; limited by FD_SETSIZE |                 Portable across Unix-like systems |                                                                                                               BSD / macOS only |                                                                                         Linux only |
| Scalability & performance | O(n) scan of fd_sets per call; costly for many fds; must rebuild fd_sets |                O(n) scan of pollfd array per call |                                                                        Kernel-maintained interest list — scalable for many fds |                                Kernel-maintained interest list — scalable; avoids scanning all fds |
| API semantics             |                       Modifies fd_sets in-place; reinit before each call | Array of struct pollfd; revents returned in-place | Register/unregister via kevent changelists; supports many filters (EVFILT\_\*, timers, signals) and flags (EV_ADD, EV_ONESHOT) | Register/unregister via epoll_ctl; epoll_wait returns ready events; supports EPOLLET, EPOLLONESHOT |
| Practical advice          |           Good for small/portable apps; watch FD_SETSIZE and reinit cost |               Good for portable small–medium apps |                                                                            Preferred on BSD/macOS for high-concurrency servers |                                                    Preferred on Linux for high-concurrency servers |
| Edge-triggered / caveat   |                                                           Not applicable |                                    Not applicable |                                                      Can use oneshot/flags; when using edge-like modes, drain I/O until EAGAIN |                           EPOLLET requires draining I/O until EAGAIN to avoid missed notifications |

- select — Use only for very small, highly portable programs or quick prototypes. It is simple but suffers O(n) scans and FD_SETSIZE limits, so it does not scale for many connections.
- poll — Prefer for portable small-to-medium applications when select’s FD_SETSIZE is a problem. Still O(n) per call, but more flexible and widely supported.
- kqueue — Best on BSD/macOS for high‑concurrency servers. Kernel‑maintained interest lists and rich filters make it scalable and efficient on those platforms.
- epoll — Best on Linux for high‑concurrency servers. Scalable and efficient with edge‑triggered and one‑shot modes, but requires careful draining (read/write until EAGAIN) to avoid missed events.
- Overall guidance — For cross‑platform code prefer poll (or abstract over poll/select), and pick kqueue or epoll when targeting macOS/BSD or Linux respectively for maximum scalability and performance.

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

```c
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

```c
#include <sys/poll.h>
int kqueue(void);
int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
```

```c
int epoll_create1(int flags);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

---

## **Usage Examples**

### **Basic Server Setup**

```c
// Create socket
int server_fd = socket(AF_INET, SOCK_STREAM, 0);

// Set socket options
int opt = 1;
setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

// Bind to address
struct sockaddr_in address;
address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;
address.sin_port = htons(6667);
bind(server_fd, (struct sockaddr*)&address, sizeof(address));

// Listen for connections
listen(server_fd, 10);

// Accept connection
int client_fd = accept(server_fd, NULL, NULL);
```

### **Non-blocking I/O (MacOS requirement)**

```c
fcntl(fd, F_SETFL, O_NONBLOCK);
```

### **Using poll()**

```c
struct pollfd fds[1];
fds[0].fd = server_fd;
fds[0].events = POLLIN;

int ready = poll(fds, 1, -1);
if (ready > 0 && (fds[0].revents & POLLIN)) {
    // Handle incoming connection
}
```

### **Signal Handling**

```c
void signal_handler(int sig) {
    // Cleanup and exit
}

struct sigaction sa;
sa.sa_handler = signal_handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = 0;
sigaction(SIGINT, &sa, NULL);
```
