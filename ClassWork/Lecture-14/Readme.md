<h2 align='center'> Scaling WordPress with Docker Compose </h2>


<hr>

<h4 align='center'> Hands On  </h4>

<hr>

**Step-1:- Create a Simple docker Compose**
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - wordpress-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_content:/var/www/html/wp-content
    depends_on:
      - mysql
    networks:
      - wordpress-network

volumes:
  mysql_data:
  wp_content:

networks:
  wordpress-network:
```

**Step-2:- Create container with scaling**
```bash
docker-compose up -d  --scale wordpress=3
```
![Create Container](./Images/1.png)
![Create Container Error](./Images/2.png)


**Step-3:- Verify Container List**
```bash
docker ps
```
![Create Container](./Images/3.png)


**Note:-** \
You will notice that the WordPress container doesn’t scale. Only one container is created due to the port mapping issue and because a specific container name is defined.


**Step-4:- Create a following **`nginx.config`** file for solving port mapping issue via Reverse Proxy.**
```yaml
events {}

http {
    upstream wordpress {
        server wordpress:80;  # Docker DNS resolves to all replicas
    }

    server {
        listen 80;
        location / {
            proxy_pass http://wordpress;
            proxy_set_header Host $host;
        }
    }
}
```

**Step-5:- Create a new **`docker-compose`** file with updated code.**
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - wordpress-network

  wordpress:
    image: wordpress:latest
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_content:/var/www/html/wp-content    # Shared uploads
    expose:
      - "80"                                    # Internal only
    depends_on:
      - mysql
    networks:
      - wordpress-network

  nginx:
    image: nginx:latest
    ports:
      - "8080:80"                                # Single entry point
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - wordpress
    networks:
      - wordpress-network

volumes:
  mysql_data:
  wp_content:

networks:
  wordpress-network:
```


**Step-6:- Create new Containers**
```bash
docker-compose up -d --scale wordpress=3
```
![Create Container](./Images/4.png)


**Step-7:- List Containers**
```bash
docker ps 
```
![List Container](./Images/5.png)

**Note:-**\
You will notice that now there are _3_ containers for the Wordpress.



**Step-8:- Stop Containers via docker-compose**
```bash
dokcer-compose down
```
![Stop Container](./Images/6.png)


**Step-9:- List Containers**
```bash
docker ps
```
![List Container](./Images/7.png)


<hr>

<h4 align='center'> Key Takeaways  </h4>

<hr>

#### Scaling Challenges

**Challenge 1: Port Conflict**
```bash
$ docker compose up --scale wordpress=3
# ERROR: Bind for 0.0.0.0:8080 failed: port is already allocated
```
**Why?** Host port `8080` can only be used once.

**Challenge 2: MySQL Can't Scale**
```bash
$ docker compose up --scale mysql=2
# ERROR: Volume conflict - multiple containers can't share mysql_data
```
**Why?** MySQL needs clustering (Galera, Group Replication) for scaling.


#### Architecture Overview

```
User Request
    ↓
Port 8080 (Host)
    ↓
Nginx (Load Balancer)
    ↓
wordpress-1 ─┐
wordpress-2 ─┼─→ MySQL (Single Instance)
wordpress-3 ─┘
    ↓
Shared Volume: wp_content (Media, Plugins, Themes)
```

#### Key Takeaways

| Component | Scaling Approach | Why |
|-----------|------------------|-----|
| **WordPress** | Scale with `--scale` | Stateless app logic |
| **MySQL** | Don't scale | Needs clustering first |
| **Ports** | Use `expose` only | Avoid host conflicts |
| **Entry Point** | Nginx on `ports` | Single access point |
| **Uploads** | Shared volume `wp_content` | All instances see same files |

#### Quick Commands Reference

```bash
# Start with 3 WordPress instances
docker compose up --scale wordpress=3 -d

# Scale to 5 instances
docker compose up --scale wordpress=5 -d

# Check logs
docker compose logs -f nginx

# Stop everything
docker compose down
```

> This approach gives you horizontal scaling for WordPress while keeping a single database and shared storage for media files.