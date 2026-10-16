[[SYSTEM_DESIGN_NOTES]]

 **load balancer** is a hardware device or software application that distributes incoming network traffic across multiple backend servers to prevent any single server from becoming overloaded


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
     it notices these things: 
     source IP Addr, port, 
     destination IP Addr, port
     protocol: TCP/UDP
     
    ![[Pasted image 20261016165632.png]] 
2. L7 LB -- -- this works on **Layer 7: Transport**  of OSI models

