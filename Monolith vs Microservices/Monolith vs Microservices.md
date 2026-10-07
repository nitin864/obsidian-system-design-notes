
[[SYSTEM_DESIGN_NOTES]]

**PART: 1 (DIFFERENCE)**

 in monolith archi we put all files in single codebase, for examp le folder like Auth, Cart , 
 Payments in a single codebase (tightly coupled)
 
![Pasted image 20261007172420](./Pasted%20image%2020261007172420.png)

it is a traditional software design model where an entire application including the user interface, business logic, and data access layers is built as a **single, unified codebase** and deployed as one indivisible unit.

Example: 

![Pasted image 20261007163748](./Pasted%20image%2020261007163748.png)

index.js --> entry point

**PROBLEMS IN MONOLITH ARCHI**:
  1--> single point of failure
  --> for exmaple suppose in grocery system,

 there are many modules such  AUTH,CART,ORDER AND PAYMENT, if something goes wrong with payment models it directly affects all other modules such as AUTH,ORDER etc as these modules as directly dependent on each other and the system is tightly coupled...

  2--> deployment bottle-neck
 
suppose we have a deployed system and we want to change something, in Payment module we want to integrate a new feature to pay via CreditCard, after integrating this feature we need to redeploy this whole system!!, that's the main issue with current monolith archi

 3--> Individual Scaling

 this is another a big issue SCALING PROBLEM, suppose your PAYMENT module is hitting 20M+ requests and it need to be scaled before whole system crashes, and in other module like AUTH and other modules not hitting that much request but due to which we need to scale our whole system instead of scaling PAYMENT module, this increase the costing and Infra!!!

This whole issue is solved by **MICROSERVICES ACRITECHTURE** 

It is a software design approach that structures an application as a collection of **loosely coupled**, **independently deployable** services organized around **business capabilities**

in this archi we divide/break-down each module in different different services rather running whole system on a single machine/server, and these services are loosely coupled. we can build,run,test,deploy them individually. This solves Monolith archi problem

![Pasted image 20261007174942](./Pasted%20image%2020261007174942.png)

Example Architeccture:

![Pasted image 20261007175109](./Pasted%20image%2020261007175109.png)


**one more thing about database, in monolith archi there is a common DB in use but in microservices archi there is different DB for each different services**. we select DB based on service functionality

**So how this micro-service solves the problem:**

suppose Payment module crashed, this does not affect the Auth, Order etc other modules because its a loosely coupled design and eacch module is running on a different servers/machines   

Also, if we want to implement a **credit-card payment feature** in the Payment module, we don't need to redeploy the entire system. We can simply redeploy the **Payment module** after integrating the feature. This minimizes downtime, prevents the entire system from going down, and ensures that other modules and users remain unaffected. 

In the **scaling** aspect, if the Auth module receives significantly more requests than the other modules, we only need to scale the **Auth module** by adding more server instances. There is no need to scale the entire system, which makes resource utilization more efficient and **reduces infrastructure costs**.

**PART2: HOW TO CONVERT MONOLITH TO MICRO ARCHI**

1. It's not a 1 day activity
2. we can't migrate whole system in 1 flow
3. we are not going to transfer 100% of traffic in a single flow

Taking example of Amazon

1. understand your monolith
2. Identify high impact area (in which module the traffic is high)
3. Build API Contract & setup communication
4. take a singe module and work on it
5. Monitoring and Observebility
6. Scaling and Optimization

**Canary deployment** is a way to release a new software version to a small number of users first. The team checks if everything works properly and there are no major bugs. If everything is fine, the new version is gradually released to all users. This reduces the risk of a faulty update affecting everyone.

STRANGLER DESIGN PATTERN

It is a software strategy used to slowly replace a large, old **monolithic application** with modern **microservices** bit by bit, instead of rewriting everything at once==

![Pasted image 20261007191915](./Pasted%20image%2020261007191915.png)

let's suppose after redirecting requests by controller, if our newly developed micro-service is currently not able to process those 10% requests than the controller automatically reduces the traffic to 0% and routes that requests to monolith archi and the whole flow looks like this: 

![Pasted image 20261007192338|700](./Pasted%20image%2020261007192338.png)

Maintain Transaction:

we use SAGA design pattern

It is a strategy for managing **distributed transactions** across multiple microservices that each have their own database.  Instead of relying on complex, blocking protocols like **Two-Phase Commit (2PC)**, the Saga breaks a large business transaction into a sequence of smaller **local transactions**.

another thing is Data Consistency

suppose we are migrating from monolith to micro and, but there is an issue the DB Consistency 