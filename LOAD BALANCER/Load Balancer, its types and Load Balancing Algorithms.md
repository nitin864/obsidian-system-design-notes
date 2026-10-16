[[SYSTEM_DESIGN_NOTES]]

 **load balancer** is a hardware device or software application that distributes incoming network traffic across multiple backend servers to prevent any single server from becoming overloaded

![Pasted image 20261016164118.png](Pasted%20image%2020261016164118.png)

Why do we need it ?

Suppose your application initially runs on a single server.

- 100 users visit your website — everything works fine.
    
- 10,000 users arrive — the server starts struggling.
    
- 100,000 users arrive — it may run out of CPU, memory, or available connections.
    

Instead of upgrading one server endlessly, you can add more servers and distribute traffic between them.

A load balancer helps with:

- Scalability: Add more servers when traffic increases.
    
- High availability: Stop routing traffic to unhealthy servers.
    
- Performance: Avoid overwhelming a single server.
    
- Fault tolerance: Keep the application available when an individual server fails.
    
- Maintenance: Take a server out of service while updating it, provided sufficient capacity remains.


![Pasted image 20261016165632.png](Pasted%20image%2020261016165632.png) 

OSI models:

The **OSI (Open Systems Interconnection) model** is a conceptual framework created by the **ISO** in 1984 to standardize how different computer systems communicate over a network using seven distinct layers

- **Layer 1: Physical** – Transmits raw bit streams over physical media like cables or radio waves. 
    
- **Layer 2: Data Link** – Provides node-to-node delivery, framing, and error detection using MAC addresses. 
    
- **Layer 3: Network** – Handles logical addressing, routing, and packet forwarding between different networks. 
    
- **Layer 4: Transport** – Ensures reliable end-to-end communication, segmentation, and flow control (e.g., TCP, UDP). 
    
- **Layer 5: Session** – Establishes, manages, and terminates connections between applications. 
    
- **Layer 6: Presentation** – Translates, encrypts, and compresses data for the application layer. 
    
- **Layer 7: Application** – Provides network services directly to end-user software like web browsers or email clients.

so there are two types of load balancer :
1. **L4 LB -- this works on Layer 4: Transport  of OSI models**
     works  with tcp/udp level
     dosen't understand header, http, cookies, http path 
     can't read : /api/v1/order
     **it address these things:** 
       source IP Addr, port, 
       destination IP Addr, port
       protocol: TCP/UDP
     
    ![Pasted image 20261016165632.png](Pasted%20image%2020261016165632.png)
    so LB4 dosen't care the request '/api/v1/chat/createRoom' it just creates the connection to any free inctances and obv there is already a logic implemented inside that connected ChatService

   2. **L7 LB -- -- this works on Layer 7: Transport  of OSI models**
    - URL path: `/api/users` or `/api/orders`
    
     - Hostname: `api.example.com` or `shop.example.com`
    
      - HTTP method: `GET`, `POST`, `PUT`, `DELETE`
    
       - HTTP headers and, where supported and configured, other request       attributes  
     
     ![Pasted image 20261016172052.png](Pasted%20image%2020261016172052.png)


![Pasted image 20261016172530.png](Pasted%20image%2020261016172530.png)


There are 2 types of algorithms in Load balancer : STATIC AND DYNAMIC

STATIC: 

1. Round Robin 
2. Weighted Round Robin
3. IP Hashing

DYNAMIC:

1. Least Connection
2. Weighted Least Connection
3. Least Response time

# Part 1: Static Load Balancing

## 2. Round Robin

### What is it?

Round Robin is the simplest load balancing algorithm. It distributes incoming requests across servers one by one in a repeating order.

### How does it work?

Suppose we have three servers: A, B, and C.

When the first customer visits the website, the request goes to Server A. The next request goes to Server B, and the third goes to Server C. After that, the cycle starts again from Server A.

For example:

- Request 1 → Server A
    
- Request 2 → Server B
    
- Request 3 → Server C
    
- Request 4 → Server A
    

### What problem does it solve?

Without a load balancer, one server might receive all the traffic. Round Robin spreads requests across the available servers, making traffic distribution simple and predictable.

### What is the problem with it?

Now imagine Server A is a powerful machine, but Server B is much weaker. Round Robin still gives both of them the same number of requests.

It also doesn't care whether a request takes 10 milliseconds or 10 seconds to finish.

So, even though the requests are distributed equally, the actual workload might not be equal.

### How do we improve it?

We realize that every server doesn't have the same capacity. Instead of treating all servers equally, we should give powerful servers more work.

That's where Weighted Round Robin comes in.

---

## 3. Weighted Round Robin

### What is it?

Weighted Round Robin is an improved version of Round Robin. It gives more requests to powerful servers and fewer requests to weaker ones.

### How does it work?

Suppose our servers have the following weights:

- Server A: Weight 3
    
- Server B: Weight 2
    
- Server C: Weight 1
    

The weights tell the load balancer how to distribute requests among the servers.

Out of every six requests, the approximate distribution would be:

- Server A receives 3 requests.
    
- Server B receives 2 requests.
    
- Server C receives 1 request.
    

This means Server A receives around 50% of the traffic, Server B around 33%, and Server C around 17%.

### What problem does it solve?

Unlike normal Round Robin, it doesn't assume that every server has the same capacity. A powerful server can handle a larger share of the incoming traffic.

### What is the problem with it?

The weights are configured beforehand. The load balancer doesn't necessarily know how busy a server is right now.

Imagine Server A is normally the most powerful server, but it is currently processing several heavy requests. Server B is almost idle.

Weighted Round Robin may still send more requests to Server A because its configured weight is higher.

### How do we improve it?

We now want to make routing decisions based on something about the client, rather than only distributing requests according to server capacity.

For example, we may want requests from the same client to reach the same server consistently.

This leads us to IP Hashing.

---

## 4. IP Hashing

### What is it?

IP Hashing uses a client's IP address to determine which server should receive its requests.

The idea is simple: requests from the same IP address will generally map to the same server while the server pool and hashing configuration remain unchanged.

### How does it work?

Imagine a customer visits our website.

The load balancer takes the customer's IP address, applies a hash or mapping function, and uses the result to select a backend server.

For example:

- Client A → Server B
    
- Client B → Server A
    
- Client C → Server C
    

If Client A visits the website again, the request will generally go to Server B under the same configuration.

### What problem does it solve?

Round Robin may send the same client's requests to different servers. IP Hashing provides more predictable routing and can help applications that rely on session affinity.

### What is the problem with it?

Imagine thousands of customers are using the same public IP address because they are connected through the same office network or mobile carrier.

Their requests may end up concentrated on one server, creating an uneven distribution.

Also, if a server is removed or the client's IP address changes, the client may be routed to a different server.

IP Hashing doesn't guarantee that a user's session will survive every server failure.

### What do we learn from this?

We have tried distributing requests in sequence, accounting for server capacity, and mapping clients to servers. But none of these approaches necessarily tells us how busy a server is at the moment.

We need an algorithm that looks at the current situation.

This brings us to Dynamic Load Balancing.

---

# Part 2: Dynamic Load Balancing

## 5. Least Connections

### What is it?

Least Connections sends new traffic to the eligible server that currently has the fewest active connections.

Instead of blindly following a fixed sequence, the load balancer looks at how many connections each server is handling.

![[Pasted image 20261016174731.png]]

### How does it work?

Suppose the servers currently have:

- Server A: 20 active connections
    
- Server B: 8 active connections
    
- Server C: 12 active connections
    

The load balancer selects Server B because it has the fewest active connections.

It checks the connection counts again when making subsequent routing decisions.

### What problem does it solve?

With Round Robin, a server might receive several long-running requests while another server finishes its work quickly.

Least Connections considers the number of connections currently being handled, making it more adaptive to certain workloads.

### What is the problem with it?

The number of connections doesn't always tell us how busy a server really is.

For example, Server A might have 10 connections that are performing expensive database operations, while Server B has 20 connections that are mostly idle.

Server A could be working much harder even though it has fewer connections.

It also doesn't automatically account for differences in server capacity.

### How do we improve it?

We need to consider two things together: how busy a server is and how powerful it is.

That is the idea behind Weighted Least Connections.

---

## 6. Weighted Least Connections

### What is it?

Weighted Least Connections considers both the current number of active connections and the relative capacity of each server.

It is useful when some servers can handle more work than others.

### How does it work?

Imagine we have three servers:

- Server A: Weight 4, with 40 active connections
    
- Server B: Weight 2, with 10 active connections
    
- Server C: Weight 1, with 8 active connections
    

For a simple illustration, we can compare connections relative to weight:

- Server A: 40 / 4 = 10
    
- Server B: 10 / 2 = 5
    
- Server C: 8 / 1 = 8
    

Under this simplified model, Server B has the lowest normalized connection count, so it would be selected.

Actual load balancers may use different formulas, so this calculation illustrates the concept rather than a universal implementation.

### What problem does it solve?

Least Connections treats connection counts as the main signal. Weighted Least Connections adds information about server capacity.

A powerful server can handle more connections than a weaker one, so we shouldn't always judge them by raw connection counts alone.

### What is the problem with it?

It still relies on connection counts as an estimate of workload.

A server with fewer connections may still be responding slowly because its database is overloaded or its requests require heavy computation.

We need a way to consider how quickly servers are actually responding.

This leads us to Least Response Time.

---

## 7. Least Response Time

### What is it?

Least Response Time is a dynamic load balancing approach that favors servers with better measured response times.

Depending on the implementation, it may also consider active connections, outstanding requests, or other performance signals.

![Pasted image 20261016175147.png](Pasted%20image%2020261016175147.png)

### How does it work?

Suppose the load balancer observes these average response times:

- Server A: 500 milliseconds
    
- Server B: 120 milliseconds
    
- Server C: 250 milliseconds
    

If response time is the deciding factor, Server B is preferred because it has been responding the fastest.

The load balancer uses its measurements to help select a backend for incoming traffic.

### What problem does it solve?

Least Connections assumes that the number of active connections gives us a useful picture of the server's workload.

But a server with fewer connections might still be slow, while a server with more connections might be responding quickly.

Least Response Time focuses on observed performance, which can be more closely related to the user's experience.

### What is the problem with it?

Response times can change very quickly.

A server that responded in 120 milliseconds a moment ago might suddenly become slow because of a database issue or a spike in traffic.

Measurements can also be affected by network latency, request complexity, and temporary slowdowns.

If the load balancer keeps sending traffic to whichever server looks fastest, it may overload that server and cause its response time to increase.

### How can we improve it?

A more advanced load balancing strategy can combine several signals, such as response time, active requests, server capacity, and health checks.

However, more signals also mean more complexity and a greater need for accurate monitoring.

The important lesson is that response time is useful, but it is not a perfect measure of a server's future performance.

---

# Part 3: Connecting the Concepts

## 8. The Complete Story

Let's quickly follow how our e-commerce website evolves.

**First, we use Round Robin.** It distributes requests equally, but doesn't account for different server capacities.

**Then, we introduce Weighted Round Robin.** Powerful servers receive more traffic, but fixed weights don't reflect changing workloads.

**Next, we try IP Hashing.** It helps keep a client's requests mapped to the same server, but clients sharing an IP can cause uneven traffic.

**Then, we move to Least Connections.** It considers active connections, but connection count doesn't always reflect the amount of work being done.

**After that, we introduce Weighted Least Connections.** It accounts for both connections and server capacity, but still doesn't directly measure how quickly requests are being served.

**Finally, we consider Least Response Time.** It uses observed response times to favor faster-responding servers, but those measurements can fluctuate and become outdated.

Each algorithm addresses a particular limitation, but introduces its own trade-offs.

The goal isn't to find one algorithm that is always best. The goal is to choose an algorithm that fits the application's workload and requirements.

---

# Part 4: Extra Concepts Worth Knowing

## 9. What happens when a server fails?

A load balancer usually performs health checks to determine whether backend servers are available.

For example, it may periodically send a request to a health endpoint. If a server repeatedly fails its health checks, the load balancer can stop sending normal traffic to it.

When the server recovers and passes the required checks, it can become eligible for traffic again.

This is important because even the best load balancing algorithm cannot distribute traffic successfully to a server that is unavailable.

## 10. Can a load balancer itself fail?

Yes. If an application depends on one load balancer and that load balancer fails, users may lose access to the entire application.

Production systems therefore commonly deploy redundant load balancers or use managed load balancing services with high-availability mechanisms.

A load balancer improves the availability of backend servers, but it must also be designed for availability.

## 11. What is the difference between a load balancing algorithm and a health check?

A load balancing algorithm decides **which eligible server should receive traffic**.

A health check helps determine **which servers are eligible to receive traffic**.

For example, Least Connections might identify Server B as the least busy server. But if Server B has failed its health checks, it should not be selected.

Both mechanisms work together.