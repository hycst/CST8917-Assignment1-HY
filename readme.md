#####  Assignment 1: Serverless Computing - Critical Analysis
#####  Student: Hesheng Yang

##### Course: CST8917 - Serverless Applications
#####  Assignment: Assignment 1: Serverless Computing - Critical Analysis

##### Part 1: Paper Summary
###### This is paper which published in 2019, six yeas ago, by Joseph Hellerstein and his colleges from Berkley.   The paper think that first-generation serverless computing, (especially Functions-as-a-Service (FaaS)), is a key mile store for cloud computing, but it has critical limit for development of cloud computing.

###### The “one step forward” meaning autoscaling.   It provides several advantages, developers can upload functions, run code on demand, save the workload of server management, and, save cost by pay only for actual execution. This is a major improvement, compared to manually virtual machines and predicting peak capacity.  

###### However, the “two steps back” are serious of current FaaS platforms’ disadvantage, it is too weak for data-intensive computing and distributed systems.


###### The paper talked about several limitations of severless comouting.

###### Firstly,  there is lifetime limition for functions,  at the times when the paper was written.  For example, AWS Lambda had a just 15-minute execution limit.  It means,  it is very hard for long-running jobs, so, the developers had to split work into smaller pieces and store progress externally. 

###### Second, FaaS platforms brought side effect of communication bottlenecks, and I/O bottlenecks as well. Functions are separated and short lived, and, they had to communicate with each other through cloud storage services, for example, object stores, queues, or databases.  So the communication among processes are very slow, compare to direct network communication.

###### Thirdly, the paper point the “data shipping” architecture has big problem.  Traditional data systems often try to move code closer to data, but, for FaaS, it is designed to moves data to code. 

Because functions run separately from storage area, and cannot reliably keep local state, so then applications have to read and write data through network again and again. This architect brought application latency, more bandwidth use, and more cost. 

###### Fourthly, at that time, FaaS platforms have limited access for hardware.  Serverless computing offered CPU and memory only, but did not give access to GPUs or other hardware.  This is a problem for workloads of machine learning and analytics.

###### The paper also explains that FaaS does not match enough for the requirement for the workloads of distributed and stateful tasks. Distributed systems needs instant fine-grained communication, consistency protocols, leader election, shared state coordination and membership tracking. Because functions are not directly assigned with direct access address, so, they cannot easily communicate like normal distributed processes. On other hands, developers have to rely on slow speed storage storage, so then, the coordination is inefficient.

###### The paper introduce several improvements for next step. They suggest for flexible code and data storage, so the application can move computation closer to data once if needs. They also suggest support for heterogeneous hardware such as GPUs and accelerators.

###### Another suggestion is long-running agent, and with direct access address, that can help to keep identification even over time, as long they are managed by the cloud.  The paper also recommend better design models for disorderly, asynchronous cloud execution, general intermediate representations, service-class objectives, and powerful security models. 

###### Generally, the paper is not to intend to reject serverless computing. On the contrast, it suggest that serverless computing have to upgrade to beyond simple stateless functions, in order to support the application to process scale computing and big data.



##### Part 2: Azure Durable Functions Deep Dive (5 topic explanations)

#####  Orchestration Model

Currently,  Azure Durable Functions has extended previous basic Azure Functions, it has new workflow orchestration model. 

A typical Durable Functions application will have client functions, orchestrator functions, and activity functions. 

The client function will start orchestration instance, the orchestrator function controls the workflow logic, and, activity functions is to run the actual work. 

The change from basic FaaS, because the application is more than collection of isolated event-triggered functions.   Instead, the orchestrator will call activities in sequence, wait for results, use timers, handle any external events, and coordinate complex workflows. 

This directly fixed one pint in the paper of Hellerstein et al. (2019): basic FaaS lacks a natural way to compose functions into reliable stateful applications. Durable Functions improves function composition, but it still runs on top of serverless infrastructure and storage-backed coordination.



##### State Management

Currently, Durable Functions can manage workflow state automatically.  It records orchestration log, checkpoints progress, based on these, it can replay the orchestrator function, in order to rebuild local state.

So, developers can write workflow logic, as  write code, provides to platform to make progress from backend. In case a function app restarts,  or, an orchestration waits for a long time, the workflow can continue from its recorded stop point, rather than starting over. 

This update fixed the the paper’s criticism that FaaS functions are stateless and short time runtime. 

However, the solution is not the exacty what the paper suggested, giving each function durable local memory.  Durable Functions runtime store status still usually on storage provider. 


##### Execution Timeouts

In current Cloud Computing,  Durable Functions is to fixing the timeout limitations, with allowing orchestrations to run for long periods. 

The orchestrator does not continuously execute like a regular long-running function.  On the contrast, it just check specific breakpoints,  stop while waiting, and only resumes in case activity results, timers, or external events is up. 

This allows workflows to last longer time, rather than let one function invocation running the whole period. 

This is an signficiant change over the initialy FaaS limitations. However, activity functions are still typelly Azure Functions, and have limit of hosting plan.   Such as, in Premium or Dedicated plan, it has more flexibility, but in the Consumption plan, regular function execution has timeout limits. 

Therefore, Durable Functions overcomes the timeout limits for workflow coordination, but does not over for each indidual unit of actual work.

Therefore, Durable Functions fixed application-level workflow state, but it does not completely fixed the paper’s  concern, which is about efficient in-memory state.


#####  Communication Between Functions

In Durable Functions, orchestrator functions communicate with each other via Functions runtime, for the information of functions activities. 

The orchestrator is responsible for schedules activity calls, holds for results, and records the results in orchestration logs. 

This architecture makes communication more efficiently, more reliable.  

From the developer’s point of view, calling an activity is similar as calling a function. 

This improves the basic FaaS problem where developers must build their own workflow state management. 

However, it only fixed part of the criticism from Hellerstein et al. (2019).   Durable Functions still uses storage in backend, and runtime-managed messages to keep works coordinately.  It still not direct, low-latency, point-to-point networking among functions.


