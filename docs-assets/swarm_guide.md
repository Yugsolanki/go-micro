# Docker Swarm Deployment Guide

## 1. Build and Push Service Images

For **each service** (`auth`, `logger`, `listener`, `mailer`, `broker`):

```bash
docker build -f logger-service.dockerfile -t yug21/logger-service:1.0.0 .
docker push yug21/logger-service:1.0.0
```

---

## 2. Swarm Commands

### Initialize Swarm

```bash
docker swarm init
```

---

### Add a Worker Node

```bash
docker swarm join --token <worker-token> <manager-ip>:2377
```

Example:

```bash
docker swarm join --token <SWARM_JOIN_TOKEN> <MANAGER_IP>:2377
```

---

### Add a Manager Node

```bash
docker swarm join-token manager
```

---

### View Join Commands

```bash
docker swarm join-token worker
# or
docker swarm join-token manager
```

---

### Deploy the Stack

```bash
docker stack deploy -c swarm.yml myapp
```

---

## 3. Scaling Services

**Scale up to 3 instances:**

```bash
docker service scale myapp_listener-service=3
```

**Scale down to 2 instances:**

```bash
docker service scale myapp_listener-service=2
```

**Check running services:**

```bash
docker service ls
```

---

## 4. Updating a Service

If you’ve made changes to a service:

1. **Rebuild the image:**

   ```bash
   docker build -f logger-service.dockerfile -t yug21/logger-service:1.0.1 .
   docker push yug21/logger-service:1.0.1
   ```

2. **Ensure at least two instances** are running to avoid downtime (services are updated one at a time).

3. **Update the running service:**

   ```bash
   docker service update --image yug21/logger-service:1.0.1 myapp_logger-service
   ```

4. **Update the `swarm.yml` file** with the new image version to keep it consistent.

---

## 5. Rolling Back a Service

```bash
docker service update --image yug21/logger-service:1.0.0 myapp_logger-service
```

---

## 6. Stopping Services

**Stop a specific service (without removing it):**

```bash
docker service scale myapp_broker-service=0
```

**Remove all services in the stack:**

```bash
docker stack rm myapp
```

---

## 7. Leaving the Swarm

```bash
docker swarm leave --force
```

> Use `--force` on the manager node (the one used for `docker swarm init`).

---


## 8. Monitoring and Inspecting

**View detailed info about a service:**

```bash
docker service inspect myapp_logger-service
```

(Use `--pretty` for a human-friendly output.)

**View tasks (containers) of a service:**

```bash
docker service ps myapp_logger-service
```

**Check logs of a service:**

```bash
docker service logs myapp_logger-service
```

(Use `-f` to follow logs in real time.)

**List nodes in the swarm:**

```bash
docker node ls
```

**Inspect a specific node:**

```bash
docker node inspect <node-id>
```

---

## 9. Node Management

**Drain a node (prevent new tasks from running on it):**

```bash
docker node update --availability drain <node-id>
```

**Set a node back to active:**

```bash
docker node update --availability active <node-id>
```

---

## 10. Stack and Service Management

**List all stacks:**

```bash
docker stack ls
```

**List all services in a stack:**

```bash
docker stack services myapp
```

**List tasks (containers) in a stack:**

```bash
docker stack ps myapp
```

---

## 11. Service Rolling Update Controls

**Force an update without changing the image (useful to restart all containers):**

```bash
docker service update --force myapp_logger-service
```

**Set update parallelism (how many tasks update at a time):**

```bash
docker service update --update-parallelism 2 myapp_logger-service
```

**Set delay between updates:**

```bash
docker service update --update-delay 10s myapp_logger-service
```

---