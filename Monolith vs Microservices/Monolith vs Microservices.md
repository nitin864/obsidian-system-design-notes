
[[SYSTEM_DESIGN_NOTES]]


 in monolith archi we put all files in single codebase, for example folder like Auth, Cart , Payments in a single codebase (tightly coupled)

it is a traditional software design model where an entire application including the user interface, business logic, and data access layers is built as a **single, unified codebase** and deployed as one indivisible unit.

Example: 

![[Pasted image 20261007163748.png]]

index.js --> entry point

**PROBLEMS IN MONOLITH ARCHI**:
  --> single point of failure
  --> for exmaple suppose in grocery system,

 there are many modules such  AUTH,CART,ORDER AND PAYMENT, if something goes wrong with payment models it directly affects all other modules such as AUTH,ORDER etc as these modules as directly dependent on each other and the system is tightly coupled...

  --> deployment bottle-neck
 
suppose we have a deployed system and we want to change something, in Payment module we want to integrate a new feature to pay via CreditCard, after integrating this feature we need to redeploy this whole system!!, that's the main issue with current monolith archi
