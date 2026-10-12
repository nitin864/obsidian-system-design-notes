[[SYSTEM_DESIGN_NOTES]]

So what is an API Gateway ? 

so API Gateway is smart entry point layer,  it is smart enough to automatically understand and routes  the client endpoint to their respective backend micro-service 

In simple terms, an API Gateway tells us which service the client wants to access and routes the request to that service.

![[Pasted image 20261012162609.png]]

It is layer or server that acts as a single entry point for clients to access multiple backend services, functioning as a reverse proxy in microservices architectures.  It sits between client applications and backend systems to **manage traffic, enforce security policies, and handle authentication**

Facility that API Provides: 

1. API Composition

![[Pasted image 20261012163317.png]]

  2. Authentication OAuth 2.0 Flow

     so this api gateway only give resource access to only authorized clients
 

![[Pasted image 20261012163758.png]]

What is done here is, Instead of having Authentication mechanism in each service here we created a centralized Auth Server for each service 


3. Service Discovery

  ### What is Service Discovery?

In simple words, **Service Discovery helps one service find the address of another service** in a distributed system.

For example, imagine an application has three services: User Service, Order Service, and Payment Service. When the Order Service needs to communicate with the Payment Service, it must know where the Payment Service is running.

Instead of manually storing the IP address and port of every service, Service Discovery helps locate the required service dynamically.

**How does it work?**

1. A service registers itself with a service registry.
    
2. When another service needs it, the registry helps find its address.
    
3. The requesting service uses that address to communicate with it.
    

**Example:** An API Gateway receives a request to place an order. It identifies that the request should go to the Order Service, while Service Discovery helps it find the Order Service's current address.

**In short:** API Gateway decides _which service should handle the request_, while Service Discovery helps find _where that service is running_.

here's whole flow diagram: 

![[Pasted image 20261012164749.png]]

**LOAD BALANCER**

so load balancer is  a networking device that act as a network cop, it distribute the incoming traffic to across multiple backend servers to prevent any single server overloading..

lets's understand this with an example:

suppose in an eCommerce system there is 3 Payment EC instances. Suppose EC1, EC2,EC3 and EC1 is working or we can say that it's ready to take traffic, but EC2 is shutdown similarly EC3 is taking heavy-load or overloaded , so there is a load balancer that automatically detects the available EC instances and redirect that traffic to that particular insatnces or in this case it redirects traffic to EC1.

;-> easiy naah..!


