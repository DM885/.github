
By using a programmatic RESTful API the user should be able to:
* create its own profile that can be access with a username and password (Done)
* Create, read, update and delete a .mzn instance  (Done)
* Create, read, update and delete a .dzn instance  (Done)
* list the name of the solvers supported and their configuration options  (Done)
  (e.g., free search, return all solutions, ...)
* trigger the execution of a solver giving the id of the mzn and dzn instances
  (only mzn if dzn is not needed), selecting the solver to use and its options,
  the timeout, maximal amount of memory that can be used, vCPUs to use   (Done)
* trigger the execution in parallel of more solvers terminating the other solvers
  when the quickest solver terminates  (Not Done)
* monitor the termination state of the solver execution  (Done)
* given a computaton request, retrieve its result if terminated, what solver
  manage to solve it first and the time it took to solve it  (Done)
* cancel the execution of a computation request (terminate the solver if
  running, delete the result otherwise)  (Done)
* bulk execution of various instances to be solved with a set of solvers in
  parallel (Not done)
* GUI support (optional for group with less than 6 people)  (Done)

The administrator of the framework should be able to:
* monitor and log the platform using a dashboard  (Done)
* kill all solver executions started by a user  (Done)
* set resources quota to users (e.g., no more than 6 vCPUs in total)  (Done)
* delete a user and all its material  (Done)
* deploy the system and add new computing nodes in an easy way  (Done)
* add or remove a solver. It is possible to assume that the solver to add
  satisfy the submission rules of the MiniZinc challenge (note also that you have to handle
  the case when a users asks to use a removed solver)  (Done)

The developer of the systems have to:
* Use continuous integration and deployment  (Done)
* Infrastructure as a Code with an automatic DevOps pipeline  (Done)
* scalable, supporting multiple users exploiting if needed more resources in the
  cloud (note: vcpus allocated to a run depending on the parameter "-p")  (Done)
* have tests to test the system (unit test, integration, ...)  (Done)
* security (proper credential management and common standard security practices
  enforced)  (Done)
* provide user stories to explain how the system is intended to be use  (Done)
* provide minimal documentation to deploy and run the system  (Done)
* fairness: if the resources do not allow to run all the solvers at the same time
  the jobs should be delayed and executed fairly (e.g. FIFO).
  User should therefore not wait  indefinitely to run their jobs (optional).  (Done)
