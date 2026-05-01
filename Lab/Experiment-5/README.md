## Experiment 5: Docker - Volumes, Environment Variables, Monitoring & Networks


<hr>

<h4 align="center"> HandsOn </h4>

<hr>

**Step-1: Run a test container**
```bash
# Create a container that writes data
docker run -it --name test-container ubuntu /bin/bash

# Inside container:
echo "Hello World" > /data/message.txt
cat /data/message.txt  # Shows "Hello World"
exit

# Restart container
docker start test-container
docker exec test-container cat /data/message.txt
# ERROR: File doesn't exist! Data was lost.
```
![Create Conatiner](./Images/1.png)

**Step-2:- Create `Anonymous Volumes`**
```bash
# Create anonymous volume (auto-generated name)
docker run -d -v /app/data --name web1 nginx

# Check volume
docker volume ls
# Shows: anonymous volume with random hash

# Inspect container to see volume mount
docker inspect web1 | grep -A 5 Mounts

```
![Create Anonymous Volumes](./Images/2.png)

**Step-3:- Create Named Volumes**
```bash
# Create named volume
docker volume create mydata

# Use named volume
docker run -d -v mydata:/app/data --name web2 nginx

# List volumes
docker volume ls
# Shows: mydata

# Inspect volume
docker volume inspect mydata

```
![Create Named Volumes](./Images/3.png)


**Step-4: Bind Mounts**
```bash
# Create directory on host
mkdir ~/myapp-data

# Mount host directory to container
docker run -d -v ~/myapp-data:/app/data --name web3 nginx

# Add file on host
echo "From Host" > ~/myapp-data/host-file.txt

# Check in container
docker exec web3 cat /app/data/host-file.txt
# Shows: From Host

```
![Bind Mounts](./Images/4.png)


**Step-5:- Database with Persistent Storage**
```bash
# MySQL with named volume
docker run -d \
  --name mysql-db \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Check data persists
docker stop mysql-db
docker rm mysql-db

# New container with same volume
docker run -d \
  --name new-mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
# Data is preserved!

```
![Persistent Storage](./Images/5.png)


**Step-6: Environment Variable**
```bash
docker run -d \
  --name app1 \
  -e DATABASE_URL="postgres://user:pass@db:5432/mydb" \
  -e DEBUG="true" \
  -p 3000:3000 \
  my-node-app
```
![Env Variable](./Images/6.png)

**Step-7:- List .env Variables**
```bash
echo "DATABASE_HOST=localhost" > .env
echo "DATABASE_PORT=5432" >> .env
```
![List .env Variables](./Images/7.png)


**Step-8: Run Container with `.env`**
```bash
docker run -d \
  --env-file .env \
  --name app2 \
  my-app
```
![Add Tag](./Images/8.png)


**Step-9:- Dockerfile Env**
```bash
nano Dockerfile
cat Dockerfile
```
Paste this:- 

```Dockerfile
FROM python:3.9-slim
ENV PORT=5000
ENV DEBUG=false
CMD ["sleep","3600"]
```
![Dockerfile](./Images/9.png)


**Step-10:- Build Flask App with the Dockerfile**
```bash
docker build -t flask-app .
docker run -d --name flask-app -p 5000:5000 flask-app
```
![Flask App](./Images/10.png)


**Step-11:- Monitoring**
```bash
docker stats --no-stream
```
![Monitor](./Images/11.png)


**Step-12: Monitor**
```bash
docker stats --no-stream
```
![List Images](./Images/12.png)


**Step-13:- Get Stats via Top**
```bash
docker top flask-app
```
![Get Stats](./Images/13.png)


**Step-14:- Inspect Container details**
```bash
docker inspect flask-app
```
![Inspect Image](./Images/14.png)


**Step-15: Inspect Container Status Only**
```bash
docker inspect --format='{{.State.Status}}' flask-app
```
![Container Status](./Images/15.png)


**Step-16:- View Events**
```bash
docker events
```
![Events](./Images/16.png)


**Step-17:- View Conatainer Events Only**
```bash
docker events --filter 'type=container' 
```
![Containers Events](./Images/17.png)


**Step-18:- Create `monitor.sh`**
```bash
nano monitor.sh
cat monitor.sh
```

Paste This:- 
```bash
#!/bin/bash

echo "=== Docker Monitoring Dashboard ==="
echo "Time: $(date)"
echo

echo "1. Running Containers:"
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
echo

echo "2. Resource Usage:"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
echo

echo "3. Recent Events:"
docker events --since '5m' --until '0s' --format "{{.Time}} {{.Type}} {{.Action}}" | tail -5
echo

echo "4. System Info:"
docker system df
```
![monitor.sh](./Images/18.png)



**Step-19:- bash `monitor.sh`**
```bash
bash monitor.sh
```
![Bash](./Images/19.png)


**Step-20:- System Info**
```bash
docker system df
```
![System Info](./Images/20.png)


**Step-21:- List Networks**
```bash
docker networks ls
```
![List Networks](./Images/21.png)


**Step-22:- Remove Network** 
```bash
docker network rm my-network
```
![Remove Network](./Images/22.png)


**Step-23: Create network**
```bash
docker network create my-network
```
![Create Network](./Images/23.png)



**Step-24:- Run Containers with Bridge Network**
```bash
docker rm -f web1 web2
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx

docker exec web1 curl http://web2
```
![Build Bridge Network](./Images/24.png)


**Step-25:- Run `nginx` Container**
```bash
docker run -d --network host nginx
curl https://localhost
```
![Run Container](./Images/25.png)


**Step-26:- Run `alpine` Container**
```bash
docker run -d --network none alpine sleep 3600
```
![Run Container](./Images/26.png)


**Step-27:- Get Ip Address of Container**
```bash
docker exec <Container ID> ip addr
```
![Ip Address](./Images/27.png)


**Step-28:- Create a Network**
```bash
docker network create app-network
```
![Create Network](./Images/28.png)


**Step-29:- Connect network with Container**
```bash
docker network connect app-network web1
docker network connect app-network web2
docker network connect app-network flask-app
docker network inspect app-network
```
![Connect Network](./Images/29.png)


**Step-30:- curl in Container**
```bash
docker exec web1 curl http://web2
```
![curl](./Images/30.png)


**Step-31:- Disconnect Network**
```bash
docker network disconnect app-network web1
docker network disconnect app-network web2
docker network disconnect app-network flask-app
docker network rm app-network
```
![Disconect Network](./Images/31.png)


**Step-32:- Create a new Network**
```bash
docker network create my-app-network
```
![Create Network](./Images/32.png)


**Step-33:- Run `postgres` Container**
```bash
docker run -d --name postgres \
--network myapp-network \
-e POSTGRES_PASSWORD=secret \
postgres:15
```
![Run Container](./Images/33.png)


**Step-34:- Run a `redis` Container**
```bash
docker run -d --name redis \
--network myapp-network \
redis:7-alpine
```
![Redis Container](./Images/34.png)


**Step-35:- Run a flask app with the previous container**
```bash
docker run -d --name flask-app \
--network myapp-network \
-p 5000:5000 \
-e DATABASE_URL="postgres://postgres:secret@postgres:5432/mydb" \
-e REDIS_URL="redis://redis:6379" \
```
![Create package file](./Images/35.png)


**Step-36:- List Containers**
```bash
docker ps
```
![List Containers](./Images/36.png)


**Step-37:- Monitor**
```bash
docker stats --no-stream
```
![Gets Stats](./Images/37.png)


**Step-38:- Ping inside container**
```bash
docker exec -it flask-app bash
apt update
apt install -y iputils-ping
```
![Test in Container](./Images/38.png)


**Step-39:- Ping `postgres` Container**
```bash
ping -c 2 postgres
```
![Ping Container](./Images/39.png)


**Step-40:- Ping `redis` Container**
```bash
ping -c 2 redis
```
![Ping Container](./Images/40.png)


**Step-41:- Inspect Network**
```bash
docker network inspect myapp-network
```
![Ping Container](./Images/40.png)






<h4 align="center"> Conclusion </h4>

<hr>


## **Key Takeaways**

1. **Volumes** persist data beyond container lifecycle
2. **Environment variables** configure containers dynamically
3. **Monitoring commands** help debug and optimize containers
4. **Networks** enable secure container communication
5. **Always use named volumes** for production data
6. **Custom networks** provide better isolation and DNS
7. **Monitor resource usage** to prevent issues
8. **Use .env files** for sensitive configuration

> This experiment covers essential Docker features for building, configuring, and managing production-ready containerized applications.



<hr>

<div align="center">

<a href="../Experiment-4/" class="btn btn-outline">⬅️ Previous</a>
&nbsp;&nbsp;
<a href="../" class="btn">🏠︎ Home</a>
&nbsp;&nbsp;
<a href="../Experiment-6/" class="btn btn-primary">Next ➡️</a>

</div>