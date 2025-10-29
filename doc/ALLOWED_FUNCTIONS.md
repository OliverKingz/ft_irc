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

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

**Allowed equivalents:** `select()`, `kqueue()`, `epoll()`

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
