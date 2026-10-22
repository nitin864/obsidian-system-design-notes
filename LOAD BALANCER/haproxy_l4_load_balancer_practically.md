# HAProxy Layer 4 Load Balancer on Arch Linux

## Goal

Build a small local demonstration of a **Layer 4 (L4) TCP load balancer** using HAProxy and two Node.js/Express backend servers.

The intended traffic flow is:

```text
Client / Browser
      |
      | HTTP request over TCP
      v
HAProxy :8080
      |
      +----> Node.js backend 1 :3000
      |
      +----> Node.js backend 2 :4000
```

HAProxy balances TCP connections. The backend applications happen to speak HTTP, but HAProxy is configured in `mode tcp`, so it does not route based on HTTP paths or headers.

---

## 1. Install HAProxy

On Arch Linux, install HAProxy with `pacman`:

```bash
sudo pacman -S haproxy
```

Check the installed version:

```bash
haproxy -v
```

---

## 2. Create the Node.js/Express application

Install Node.js and npm if they are not already installed. Then, in your project directory, initialize npm and install Express:

```bash
npm init -y
npm install express
```

The example uses two files: `app.js` defines the Express application, while `index.js` starts the HTTP server.

### `app.js`

```javascript
const express = require('express');

const app = express();

app.get('/*', (req, res) => {
  res.send(`served on port ${process.env.PORT || 3000}`);
});

module.exports = app;
```

**Express 5 note:** If this route pattern causes a path-to-regexp error, change `app.get('/*', ...)` to `app.get('/{*splat}', ...)`. This pattern also matches the root route `/`.

### `index.js`

```javascript
const app = require('./app');

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

Keep `module.exports = app` in `app.js`, not at the bottom of `index.js`. `index.js` imports the app and starts the listener.

---

## 3. Run two backend servers

Open two terminals in the project directory.

### Terminal 1 — backend on port 3000

```bash
node index.js
```

Expected output:

```text
Server is running on port 3000
```

### Terminal 2 — backend on port 4000

```bash
PORT=4000 node index.js
```

Expected output:

```text
Server is running on port 4000
```

The `PORT=4000` assignment applies to that command only. It does not change the port used by the other Node.js process.

Test the backends directly:

```bash
curl http://localhost:3000
curl http://localhost:4000
```

Expected responses:

```text
served on port 3000
served on port 4000
```

Keep both backend processes running while testing HAProxy.

---

## 4. Back up the HAProxy configuration

The main configuration file used by the Arch Linux system service is:

```text
/etc/haproxy/haproxy.cfg
```

Back it up before editing:

```bash
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

Edit the configuration:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Save in Nano with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.

> The command `haproxy -f tcp.cfg` looks for a file named `tcp.cfg` in the current directory. It failed with “No such file or directory” because that file was not present in `~`. Use the full path to the system configuration file, or create `tcp.cfg` explicitly if you want a separate file.

---

## 5. HAProxy Layer 4 configuration

Use the following configuration in `/etc/haproxy/haproxy.cfg` for this local demo:

```haproxy
global
    log stdout format raw local0
    maxconn 2000

defaults
    log global
    mode tcp
    option tcplog
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend tcp_frontend
    bind *:8080
    mode tcp
    default_backend node_backends

backend node_backends
    mode tcp
    balance roundrobin
    server node1 127.0.0.1:3000 check
    server node2 127.0.0.1:4000 check
```

### What each section means

- **`global`**: Sets process-wide HAProxy options.
- **`defaults`**: Defines settings inherited by later sections.
- **`mode tcp`**: Uses Layer 4 TCP forwarding rather than HTTP-aware routing.
- **`timeout connect 5s`**: Allows up to five seconds to establish a backend connection.
- **`timeout client 30s` / `timeout server 30s`**: Sets inactivity timeouts for client and server connections.
- **`frontend tcp_frontend`**: Defines the incoming listener.
- **`bind *:8080`**: Listens on port 8080 on the machine's available interfaces.
- **`default_backend node_backends`**: Sends accepted connections to the backend pool.
- **`balance roundrobin`**: Selects eligible backends in rotation for new connections.
- **`server ... check`**: Enables active TCP health checks for each backend.

### Layer 4 versus Layer 7

| Layer 4 (this demo) | Layer 7 (HTTP mode) |
|---|---|
| Operates on TCP connections | Understands HTTP requests |
| Does not route by URL path or HTTP headers | Can route by path, host, headers, etc. |
| Can proxy non-HTTP TCP protocols | Requires application-layer protocol support for HTTP routing |

Even though the browser sends HTTP, HAProxy is configured to proxy the underlying TCP connection. The Express application generates the HTTP response.

---

## 6. Validate the configuration

Before starting or reloading HAProxy, validate the configuration:

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

A successful validation reports that the configuration is valid. Fix any reported errors before continuing.

---

## 7. Start HAProxy with systemd

Enable HAProxy to start at boot and start it now:

```bash
sudo systemctl enable --now haproxy
```

Check its status:

```bash
systemctl status haproxy
```

Follow the service logs when troubleshooting:

```bash
sudo journalctl -u haproxy -f
```

To see recent logs without following them:

```bash
sudo journalctl -u haproxy -n 50 --no-pager
```

If HAProxy was already running and you changed its configuration, validate it first and then restart the service:

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```

---

## 8. Test through the load balancer

With both backends and HAProxy running, send requests through the frontend:

```bash
curl http://localhost:8080
curl http://localhost:8080
curl http://localhost:8080
curl http://localhost:8080
```

Expected responses should include both:

```text
served on port 3000
served on port 4000
```

The exact order can vary. A browser may reuse a TCP connection, so repeated browser refreshes are not a perfect way to observe round-robin selection. Separate `curl` invocations usually create separate connections and are better for this simple demonstration.

You can also test the frontend in a browser at:

http://localhost:8080

---

## 9. Demonstrate health checks and backend failure

1. Confirm both Node.js servers are running.
2. Stop the backend on port 4000 by pressing `Ctrl+C` in its terminal.
3. Wait briefly for HAProxy's health check to mark it unavailable.
4. Run several requests:

   ```bash
   curl http://localhost:8080
   ```

5. Requests should continue to reach the healthy backend on port 3000.
6. Restart the second backend:

   ```bash
   PORT=4000 node index.js
   ```

7. After the health check succeeds, HAProxy should make that backend eligible again.

This demonstrates basic backend health checking and continued service when one backend is unavailable. It does not make the application highly available if HAProxy itself or the machine fails.

---

## 10. Troubleshooting

### Error: `Cannot open configuration file/directory tcp.cfg`

Cause: HAProxy could not find `tcp.cfg` in the current directory.

Use the system configuration file:

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

### HAProxy service fails to start

Inspect the service logs:

```bash
sudo journalctl -u haproxy -n 50 --no-pager
```

Then validate the configuration:

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

### Port 8080 is already in use

Check which process is listening:

```bash
sudo ss -ltnp | grep ':8080'
```

Stop or reconfigure the conflicting process before starting HAProxy.

### A backend is unavailable

Check listeners:

```bash
ss -ltnp | grep -E ':(3000|4000|8080)\b'
```

Test each backend directly:

```bash
curl http://127.0.0.1:3000
curl http://127.0.0.1:4000
```

Ensure the Node.js servers are still running and that each uses the expected port.

### Express 5 rejects `app.get('/*', ...)`

Change the route to:

```javascript
app.get('/{*splat}', (req, res) => {
  res.send(`served on port ${process.env.PORT || 3000}`);
});
```

Then restart both backend processes.

---

## 11. Commands quick reference

| Task | Command |
|---|---|
| Install HAProxy | `sudo pacman -S haproxy` |
| Check version | `haproxy -v` |
| Start backend 1 | `node index.js` |
| Start backend 2 | `PORT=4000 node index.js` |
| Edit config | `sudo nano /etc/haproxy/haproxy.cfg` |
| Validate config | `sudo haproxy -c -f /etc/haproxy/haproxy.cfg` |
| Start and enable service | `sudo systemctl enable --now haproxy` |
| Check service | `systemctl status haproxy` |
| Restart after config change | `sudo systemctl restart haproxy` |
| Follow logs | `sudo journalctl -u haproxy -f` |
| Test backend 1 | `curl http://localhost:3000` |
| Test backend 2 | `curl http://localhost:4000` |
| Test load balancer | `curl http://localhost:8080` |

---

## Final result

The local demo uses three ports:

- **3000:** First Express backend
- **4000:** Second Express backend
- **8080:** HAProxy TCP frontend

A client connects to port 8080, and HAProxy distributes new TCP connections between the healthy backend servers using round-robin balancing.

> **Scope note:** This is a local learning setup. Before exposing it to a network or production traffic, review access controls, logging, TLS requirements, timeouts, health checks, and service hardening.


![Screenshot From 2026-10-22 20-23-31.png](Screenshot%20From%202026-10-22%2020-23-31.png)![Screenshot From 2026-10-22 20-23-22.png](Screenshot%20From%202026-10-22%2020-23-22.png)

haproxy configuration file: 


global
    log stdout format raw local0
    maxconn 2000

defaults
    log global
    mode tcp
    option tcplog
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend tcp_frontend
    bind *:8080
    mode tcp
    default_backend node_backends

backend node_backends
    mode tcp
    balance roundrobin
    server node1 127.0.0.1:3000 check
    server node2 127.0.0.1:4000 check
