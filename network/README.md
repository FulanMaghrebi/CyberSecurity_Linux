# Networking

This README deals with important Linux commands used to operate and scan the network.

---

## netstat

`netstat` displays information about network connections, listening ports, routing tables, and network interfaces.

```bash
netstat
```

On Ubuntu/Debian, `netstat` is part of the `net-tools` package:

```bash
sudo apt install net-tools
```

Modern Linux systems often use `ss` instead of `netstat`.

### Show all connections

```bash
netstat -a
```

`-a` → shows all sockets, including listening and non-listening sockets.

### Show TCP connections

```bash
netstat -t
```

`-t` → shows TCP connections.

Show all TCP sockets:

```bash
netstat -at
```

### Show UDP connections

```bash
netstat -u
```

`-u` → shows UDP sockets.

Show all UDP sockets:

```bash
netstat -au
```

### Show listening ports

```bash
netstat -l
```

`-l` → shows only listening sockets.

TCP listening ports:

```bash
netstat -lt
```

UDP listening ports:

```bash
netstat -lu
```

TCP and UDP listening ports:

```bash
netstat -ltu
```

### Show numerical addresses and ports

```bash
netstat -n
```

`-n` → shows numerical IP addresses and port numbers instead of resolving hostnames and service names.

### Show processes using ports

```bash
sudo netstat -p
```

`-p` → shows the PID and program associated with a socket.

### Show listening services

One of the most useful commands:

```bash
sudo netstat -tulpn
```

Options:

```text
-t → TCP
-u → UDP
-l → listening sockets
-p → PID / program
-n → numerical IP addresses and ports
```

### Show established connections

```bash
netstat -tn | grep ESTABLISHED
```

### Show routing table

```bash
netstat -r
```

Show numerical addresses:

```bash
netstat -rn
```

### Show network interfaces

```bash
netstat -i
```

`-i` → shows statistics about network interfaces.

### Show network statistics

```bash
netstat -s
```

TCP statistics:

```bash
netstat -st
```

UDP statistics:

```bash
netstat -su
```

### Continuous output

```bash
netstat -c
```

`-c` → continuously refreshes the output.

Stop with:

```text
Ctrl + C
```

### Verbose mode

```bash
netstat -v
```

`-v` → enables verbose output.

Combined example:

```bash
sudo netstat -tulpnv
```

### Search for a specific port

Search for port 22:

```bash
sudo netstat -tulpn | grep :22
```

Search for port 80:

```bash
sudo netstat -tulpn | grep :80
```

Search for port 443:

```bash
sudo netstat -tulpn | grep :443
```

### Useful combinations

```bash
netstat -atn
netstat -aun
sudo netstat -tlpn
sudo netstat -tulpn
netstat -tn | grep ESTABLISHED
netstat -rn
netstat -i
netstat -s
```

Modern equivalent using `ss`:

```bash
sudo ss -tulpn
``` 
#netstat Quick Overview

| Option | Purpose |
| ------ | ------- |
| `-a` | Show all sockets |
| `-t` | Show TCP sockets |
| `-u` | Show UDP sockets |
| `-l` | Show listening sockets |
| `-n` | Show numerical IP addresses and ports |
| `-p` | Show PID and program |
| `-r` | Show routing table |
| `-i` | Show network interfaces |
| `-s` | Show protocol statistics |
| `-c` | Continuously refresh output |
| `-v` | Verbose output |
