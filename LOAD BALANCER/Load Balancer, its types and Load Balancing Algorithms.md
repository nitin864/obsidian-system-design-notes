[[SYSTEM_DESIGN_NOTES]]

 **load balancer** is a hardware device or software application that distributes incoming network traffic across multiple backend servers to prevent any single server from becoming overloaded

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


![[Pasted image 20261016164118.png]]

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
     
    ![[Pasted image 20261016165632.png]] 
    so LB4 dosen't care the request '/api/v1/chat/createRoom' it just creates the connection to any free inctances and obv there is already a logic implemented inside that connected ChatService

   2. **L7 LB -- -- this works on Layer 7: Transport  of OSI models**
    - URL path: `/api/users` or `/api/orders`
    
     - Hostname: `api.example.com` or `shop.example.com`
    
      - HTTP method: `GET`, `POST`, `PUT`, `DELETE`
    
       - HTTP headers and, where supported and configured, other request       attributes  
     
     ![[Pasted image 20261016172052.png]]


![[Pasted image 20261016172530.png]]


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

