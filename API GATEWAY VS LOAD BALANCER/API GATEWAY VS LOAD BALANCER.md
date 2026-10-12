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
 

