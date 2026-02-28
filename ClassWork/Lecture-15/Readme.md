<h2 align='center'> Docker Swarm </h2>


<hr>

<h4 align='center'> Introduction  </h4>

<hr>

#### 1. Why Do We Need Docker Swarm ?

**Problem Without Swarm**

If you run containers using:

```bash
docker run nginx
```

**Problems:**

* Runs on single machine only
* No automatic recovery if container crashes
* No scaling across multiple servers
* No built-in load balancing
* Manual management

**Real World Requirements**

| Requirement        | Example Scenario                             |
| ------------------ | -------------------------------------------- |
| High availability  | Web app must not go down                     |
| Auto healing       | If container crashes → restart automatically |
| Horizontal scaling | Increase instances during high traffic       |
| Rolling updates    | Update app without downtime                  |
| Load balancing     | Distribute traffic across replicas           |
| Multi-node cluster | Use multiple physical/VM servers             |

Docker Swarm solves all of this.



##### 2. Feature Highlights of Docker Swarm

* Built into Docker Engine (no extra installation)
* Declarative service model
* Automatic load balancing
* Rolling updates
* Service scaling
* Self-healing containers
* Routing mesh
* Secure by default (TLS between nodes)



#### 3. Swarm Mode Key Concepts

**3.1 Swarm**
A **cluster of Docker hosts** working together.


**3.2 Nodes**
A machine participating in the swarm.

_Types:_

* **Manager Node** → Controls cluster
* **Worker Node** → Runs containers


**3.3 Service**
A definition of desired state.

_Example:_
> “Run 3 nginx containers and keep them running.”

Swarm ensures desired state is maintained.


**3.4 Task**

A single running container inside a service.

If service replicas = 3 → 3 tasks created.


**3.5 Load Balancing**
Swarm distributes traffic automatically across replicas.




<hr>

<h4 align='center'> HandsOn  </h4>

<hr>


**Step-1: Initialize Swarm**
```bash
docker swarm init
```
![Initialize Swarm](./Images/1.png)

_Explanation:-_
* `docker` → Docker CLI
* `swarm` → Swarm management command
* `init` → Initialize a new swarm cluster


**Step-2: Verify Node**
```bash
docker node ls
```
![Verify Node](./Images/2.png)

### Explanation

* `node` → Manage swarm nodes
* `ls` → List nodes

**Step-3: Create Service**
```bash
docker service create \
  --name webapp \
  --replicas 3 \
  -p 8080:80 \
  nginx
```
![Create Service](./Images/3.png)

_Explanation:-_
* `docker service` → Manage swarm services
* `create` → Create new service
* `--name webapp` → Assign service name
* `--replicas 3` → Run 3 container instances
* `-p 8080:80` → Publish port
  * 8080 → Host port
  * 80 → Container port
* `nginx` → Image name


**Step-4: Check Services**
```bash
docker service ls
```
![Verify Service](./Images/4.png)

* `ls` → List all services



**Step-5: List Containers**
```bash
docker service ps webapp
```
![List Containers](./Images/5.png)

* `ps` → Show tasks (containers) of service
* `webapp` → Service name



**Step-6:- Inspect Services**
```bash
docker service inspect webapp
```
![Inspect Services](./Images/6.png)


_Explanation:-_
* `inspect` → Show detailed JSON configuration
* Useful for debugging


**Step-7:- For formatted output:**
```bash
docker service inspect --pretty webapp
```
![Initialize Swarm](./Images/7.png)

* `--pretty` → Human-readable format



**Step-8:- Scale the Service**
Traffic increased during sale → need 6 replicas.

```bash
docker service scale webapp=6
```
![Scale Service](./Images/8.png)

_Explanation:-_
* `scale` → Change replica count
* `webapp=6` → Desired replicas


**Step-9:- Verify:**
```bash
docker service ls
```
![Verify Swarm](./Images/9.png)

> Swarm automatically:
* Creates new containers
* Distributes them across nodes


**Step-10:- Delete the Service**
```bash
docker service rm webapp
```
![Remove Service](./Images/10.png)

_Explanation:-_
* `rm` → Remove service
* Deletes all replicas automatically


**Step-11:- Verify that services Stop or Not**
```bash
docker service ls
```
![Verify Stop Service](./Images/11.png)


<hr>

<h4 align='center'> Key Takeaways  </h4>

<hr>

#### Summary of Core Commands

| Action          | Command                                 |
| --------------- | --------------------------------------- |
| Init swarm      | docker swarm init                       |
| Join swarm      | docker swarm join                       |
| List nodes      | docker node ls                          |
| Create service  | docker service create                   |
| List services   | docker service ls                       |
| Inspect service | docker service inspect                  |
| Scale service   | docker service scale                    |
| Update service  | docker service update                   |
| Drain node      | docker node update --availability drain |
| Remove service  | docker service rm                       |



#### Real Industry Use Case Example

E-commerce Website:

| Layer           | Swarm Usage                 |
| --------------- | --------------------------- |
| Frontend        | 5 replicas                  |
| Backend API     | 4 replicas                  |
| Redis           | 3 replicas                  |
| Database        | Stateful service            |
| Rolling updates | During deployments          |
| Node drain      | During hardware maintenance |


#### `Docker Run` vs `Docker Swarm`

| Feature             | docker run       | Docker Swarm (docker service) |
| ------------------- | ---------------- | ----------------------------- |
| Scope               | Single container | Cluster-managed service       |
| Environment         | Single host      | Multi-node cluster            |
| Scaling             | Manual           | Automatic                     |
| Load balancing      | No               | Yes                           |
| Self-healing        | No               | Yes                           |
| Rolling updates     | No               | Yes                           |
| Desired state model | No               | Yes                           |
| Production-ready    | Limited          | Yes                           |


#### Common Flags (Same Purpose, Different Context)

| Purpose        | docker run       | docker service create | Notes                   |
| -------------- | ---------------- | --------------------- | ----------------------- |
| Name           | `--name`         | `--name`              | Same                    |
| Publish Port   | `-p`             | `-p`                  | Swarm uses routing mesh |
| Environment    | `-e`             | `-e`                  | Same                    |
| Mount Volume   | `-v` / `--mount` | `--mount`             | Slight syntax diff      |
| Network        | `--network`      | `--network`           | Swarm uses overlay      |
| Detach         | `-d`             | Default               | Swarm runs detached     |
| Restart policy | `--restart`      | `--restart-condition` | Swarm more advanced     |


#### Different Flags (Swarm-Specific)

These DO NOT exist in `docker run`:

| Swarm Flag             | Purpose                     |
| ---------------------- | --------------------------- |
| `--replicas`           | Number of containers        |
| `--mode`               | replicated / global         |
| `--update-delay`       | Rolling update delay        |
| `--update-parallelism` | Update concurrency          |
| `--constraint`         | Node placement              |
| `--limit-cpu`          | Resource limit cluster-wide |
| `--reserve-memory`     | Resource reservation        |
| `--with-registry-auth` | Pass registry credentials   |
| `--placement-pref`     | Spread tasks                |


#### docker run Architecture

User → Docker Engine → Container

Single machine only.


#### Docker Swarm Architecture

User → Manager Node → Scheduler → Worker Nodes → Tasks

Cluster-based.


#### Final Summary

| Capability           | docker run | Docker Swarm  |
| -------------------- | ---------- | ------------- |
| Runs container       | Yes        | Yes (as task) |
| Multi-node           | No         | Yes           |
| Scaling              | Manual     | Built-in      |
| Auto-healing         | No         | Yes           |
| Rolling updates      | No         | Yes           |
| Routing mesh         | No         | Yes           |
| Placement rules      | No         | Yes           |
| Resource scheduling  | Basic      | Advanced      |
| Production readiness | Limited    | Yes           |
