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
1. L4 LB -- this works on **Layer 4: Transport**  of OSI models
     works  with tcp/udp level
     dosen't understand header, http, cookies, http path 
     can't read : /api/v1/order
     **it address these things:** 
       source IP Addr, port, 
       destination IP Addr, port
       protocol: TCP/UDP
     
    ![[Pasted image 20261016165632.png]] 
    so LB4 dosen't care the request '/api/v1/chat/createRoom' it just creates the connection to any free inctances and obv there is already a logic implemented inside that connected ChatService

1. L7 LB -- -- this works on **Layer 7: Transport**  of OSI models
     http,https, websockets --it understands 
     
     ![[Pasted image 20261016172052.png]]

