# [[SYSTEM_DESIGN_NOTES]]

# Proxy vs Reverse Proxy

> Topic: Proxy vs Reverse Proxy | How Proxy differs from VPN and Load Balancer

## First understand the word "Proxy"

In simple terms, a **proxy is an intermediate server** between the client and the destination server.

The client does not directly talk to the destination. It first sends the request to the proxy, and then the proxy forwards that request to the destination on behalf of the client.

So the basic flow is:

```text
Client  --->  Proxy  --->  Destination Server
Client  <---  Proxy  <---  Destination Server
```

The important thing is **whose behalf the proxy is acting on**:

- A **forward proxy** acts on behalf of the client.
- A **reverse proxy** acts on behalf of the backend server.

This is the main difference between a normal proxy and a reverse proxy.

![](Pasted%20image%2020261021155656.png)

---

## What is a Forward Proxy?

A **forward proxy** is a server that sits in front of the clients and sends requests to the internet or some other external server for them.

Here the proxy is controlled or configured by the **client side**.

```text
                    Internet
                       ▲
                       │
Client  ─────────►  Forward Proxy  ─────────►  Website
```

For example, suppose I am using a computer inside an organization. Instead of allowing my computer to access websites directly, the organization can configure all outgoing traffic to pass through a proxy server.

Now the proxy can decide:

- which websites I am allowed to visit
- which websites should be blocked
- whether the response can be served from cache
- how much bandwidth a user is consuming
- whether the request should be logged

In this case, the website sees the **proxy's IP address** instead of the actual client's IP address.

### Why do we use a Forward Proxy?

1. **Privacy** - the destination website does not directly see the client's IP address.
2. **Access control** - an organization can allow or block certain websites.
3. **Monitoring** - administrators can monitor outgoing traffic.
4. **Caching** - commonly requested responses can be stored and returned quickly.
5. **Content filtering** - requests can be checked before leaving the network.
6. **Bypassing network restrictions** - depending on the setup, the client may access resources through another network.

But one important thing is:

> A proxy does not automatically mean that the traffic is encrypted.

If we send HTTP traffic through a proxy, the proxy may be able to read that traffic. HTTPS protects the traffic between the client and the destination, but we still need to understand where TLS is terminated and who operates the proxy.

---

## What is a Reverse Proxy?

A **reverse proxy** is a server that sits in front of one or more backend servers and receives requests on their behalf.

Here the reverse proxy is controlled by the **server side**.

The client thinks it is talking to the actual application, but the request first reaches the reverse proxy. The reverse proxy then decides which backend service should process that request.

```text
Client  ─────►  Reverse Proxy  ─────►  Backend Server 1
                    │
                    ├──────────────►  Backend Server 2
                    │
                    └──────────────►  Backend Server 3
```

For example, when I open `example.com`, the request may first reach Nginx, HAProxy, Cloudflare, an AWS load balancer, or some other reverse proxy. That component then forwards my request to the correct application server.

The client does not need to know:

- the private IP address of the backend server
- how many backend instances are running
- which service is responsible for the request
- whether the request is being served from cache

So the reverse proxy becomes a **single public entry point** for the backend system.

### Why do we use a Reverse Proxy?

#### 1. Hide backend servers

The backend servers can stay inside a private network. Only the reverse proxy is exposed to the internet.

This reduces direct exposure of the application servers.

#### 2. Routing

The reverse proxy can route requests based on the hostname or URL path.

```text
api.example.com/users  ─────►  User Service
api.example.com/orders ─────►  Order Service
api.example.com/payments ───►  Payment Service
```

This is why an API Gateway is also commonly implemented using reverse-proxy behaviour.

#### 3. Load balancing

If we have multiple instances of the same service, the reverse proxy can distribute traffic between them.

```text
                         ┌──► App Instance 1
Client ─► Reverse Proxy ─┼──► App Instance 2
                         └──► App Instance 3
```

#### 4. TLS termination

The reverse proxy can handle the HTTPS connection from the client. It decrypts the request and forwards it to the backend using HTTP or HTTPS depending on the security requirement.

```text
Client ── HTTPS ──► Reverse Proxy ── HTTP/HTTPS ──► Backend
```

This removes the responsibility of managing certificates from every individual backend instance.

#### 5. Caching and compression

The reverse proxy can cache static or frequently requested responses. It can also compress responses before sending them to the client, reducing bandwidth usage.

#### 6. Security and traffic control

It can provide:

- rate limiting
- authentication checks
- request filtering
- Web Application Firewall (WAF) rules
- IP allowlists and denylists
- request and response logging

---

## Forward Proxy vs Reverse Proxy

| Point | Forward Proxy | Reverse Proxy |
| --- | --- | --- |
| Works on behalf of | Client | Backend server |
| Sits in front of | Clients | Servers |
| Main direction | Outgoing client traffic | Incoming server traffic |
| Client knows about it? | Usually yes, it is configured in the client/network | Usually no, it looks like the actual application |
| Hides | Client IP from the destination | Backend server details from the client |
| Common use cases | Filtering, monitoring, privacy, outbound access | Routing, TLS, caching, security, load balancing |
| Example | Corporate proxy | Nginx, HAProxy, Cloudflare, AWS ALB |

The word **proxy** is not enough to understand the design. We should always ask: **is it representing the client or the server?**

---

## Proxy vs VPN

Many people think that a proxy and VPN are the same because both can hide the client's public IP address. But their working level and coverage are different.

### What does a VPN do?

A **VPN (Virtual Private Network)** creates an encrypted tunnel between the user's device/network and the VPN server.

```text
Your Device  ═════ Encrypted VPN Tunnel ═════►  VPN Server  ───► Internet
```

After connecting to a VPN, normally the network traffic from the device is routed through that tunnel. This includes traffic from multiple applications, not only one browser or one configured request.

### Main differences

#### 1. Level at which they work

- A proxy usually works at the application or protocol level, such as HTTP or SOCKS.
- A VPN works at the network level and routes traffic through a virtual network interface.

#### 2. Coverage

- A proxy may affect only the browser or application configured to use it.
- A VPN can affect almost all network traffic from the device, depending on the VPN configuration.

#### 3. Encryption

- A VPN normally provides an encrypted tunnel between the device and the VPN server.
- A proxy by itself does not guarantee encryption.

HTTPS can still encrypt traffic when a proxy is being used, but that encryption comes from HTTPS, not automatically from the proxy.

#### 4. Performance

- A proxy can be lighter because it may handle only selected application traffic.
- A VPN adds encryption and tunnel overhead, but provides broader protection.

#### 5. Typical use cases

**Proxy:**

- organization-level filtering
- browser-specific routing
- caching
- scraping or controlled outbound requests
- application-level access control

**VPN:**

- secure connection over an untrusted network
- connecting remote employees to a private company network
- protecting traffic on public Wi-Fi
- routing device traffic through another network

### Simple way to remember

> A proxy is generally an intermediary for selected application requests, while a VPN creates a secure network path for the device or network.

Neither one should be treated as a magic privacy solution. The proxy or VPN provider can still observe some metadata or traffic depending on encryption and configuration.

---

## Reverse Proxy vs Load Balancer

A **load balancer** distributes traffic across multiple backend servers.

A **reverse proxy** is a broader server-side intermediary that receives client requests and can perform routing, security, caching, TLS termination, and load balancing.

So load balancing can be one responsibility of a reverse proxy, but a reverse proxy is not limited to load balancing.

```text
Client ─► Reverse Proxy
             │
             ├── TLS termination
             ├── Authentication
             ├── Rate limiting
             ├── Caching
             ├── Routing
             └── Load balancing ─► Multiple backend instances
```

### Example

Suppose we have three Payment Service instances:

```text
                         ┌──► Payment Instance 1
Client ─► Reverse Proxy ──┼──► Payment Instance 2
                         └──► Payment Instance 3
```

The reverse proxy receives `/payment` requests. The load-balancing part checks which instances are healthy and selects one instance using an algorithm such as:

- Round Robin
- Weighted Round Robin
- Least Connections
- IP Hashing
- Least Response Time

If Payment Instance 2 is down, the reverse proxy/load balancer performs health checks and stops sending traffic to that instance.

### Difference in one line

- **Reverse proxy:** protects and manages access to backend servers.
- **Load balancer:** distributes traffic among backend servers.

In real systems, the same product can provide both features, which is why the terms sometimes look interchangeable.

---

## Complete request flow in a production system

```text
User
  │
  ▼
DNS
  │
  ▼
CDN / WAF / Reverse Proxy
  │
  ├── TLS termination
  ├── Authentication and rate limiting
  ├── Cache check
  ├── URL-based routing
  └── Load balancing
          │
          ├──► API Service Instance 1
          ├──► API Service Instance 2
          └──► API Service Instance 3
```

The flow can be understood like this:

1. The user requests a domain name.
2. DNS points the request towards the public entry point.
3. The reverse proxy receives the request.
4. It checks security rules, authentication, and rate limits.
5. If a cached response exists, it may return it directly.
6. Otherwise, it identifies the correct backend service.
7. The load-balancing logic selects a healthy instance.
8. The backend processes the request and sends the response back through the reverse proxy.

This is the reason reverse proxies are very common in scalable system design.

---

## Important points to remember

1. A forward proxy is **client-facing** and represents the client.
2. A reverse proxy is **server-facing** and represents the backend.
3. A reverse proxy hides backend infrastructure and provides one public entry point.
4. A reverse proxy can perform load balancing, but load balancing is only one of its responsibilities.
5. A VPN creates an encrypted network tunnel; a proxy does not automatically provide encryption.
6. HTTPS encryption and VPN encryption are different things.
7. Health checks are required before sending traffic to backend instances.
8. In production, the reverse proxy itself should be highly available. Otherwise it can become a single point of failure.

## Final understanding

If I am trying to protect or control **outgoing requests from clients**, I think about a **forward proxy**.

If I am trying to protect, route, secure, or scale **incoming requests to backend servers**, I think about a **reverse proxy**.

If I want an encrypted network connection for the device or private network, I think about a **VPN**.

If my main job is to distribute traffic among multiple healthy backend instances, I think about a **load balancer**.

```text
Forward Proxy  -> represents the client
Reverse Proxy  -> represents the server
VPN            -> creates an encrypted network tunnel
Load Balancer  -> distributes traffic across servers
```
