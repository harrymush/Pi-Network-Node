# Pi-Network-Node
Setting up a Pi Network Node for an iMac and a MacBook Pro

Absolutely! Here's a polished version of your Pi Network Node setup and troubleshooting as a **GitHub-ready Markdown README**:

---

```markdown
# Pi Network Node Setup on macOS (Monterey & High Sierra)

This write-up documents how I set up and troubleshot a Pi Network Node using Docker on two Macs: a **MacBook Pro (Monterey)** and an older **iMac (High Sierra)**. It covers everything from port forwarding and firewall configuration to debugging Docker container behavior.

---

## Requirements

- Docker (latest compatible version for your OS)
- Access to your router for port forwarding
- Terminal access (macOS)
- Optional: [`docker-compose`](https://docs.docker.com/compose/)

---

## Step-by-Step Setup

### 1. Install Docker

#### Monterey (MacBook Pro)
- Docker installed without issues.
- Containers and Pi Node ran successfully out of the box.

#### High Sierra (iMac)
- Installed Intel-compatible Docker manually.
- Verified architecture using:
  ```sh
  docker version
  ```
- Docker ran, but container networking needed troubleshooting.

---

### 2. Required Ports for Pi Node

| Purpose              | Ports              |
|----------------------|--------------------|
| Pi Node communication | 31400–31409 (TCP) |
| General web access    | 80, 443 (TCP)     |

---

### 3. Configure Router (GPON)

- Accessed router admin panel.
- Forwarded **TCP ports 31400–31409** to the internal IP of each Mac.

---

### 4. Configure macOS Firewall

#### On iMac (High Sierra):

Used **`pf` (Packet Filter)** since `ufw` is Linux-only.

**Edit rules:**
```sh
sudo nano /etc/pf.conf
```

**Add rules:**
```pf
pass in quick proto tcp from 192.168.1.67 to any port 31400:31409
block in quick proto tcp from any to any port 31400:31409
```

**Apply them:**
```sh
sudo pfctl -f /etc/pf.conf
sudo pfctl -e
```

---

### 5. Run the Pi Node Container

#### Working Command (with explicit environment variable):
```sh
docker run -p 31400-31409:31400-31409 \
  -e PORTS="31400,31401,31402,31403,31404,31405,31406,31407,31408,31409" \
  ese7en/node-port-test
```

#### On iMac (due to networking issues), use:
```sh
docker run --network=host -e PORTS="31400,31401,...,31409" ese7en/node-port-test
```

---

### 6. Docker Compose (for persistence)

#### `docker-compose.yml`:
```yaml
version: '3'
services:
  node-port-test:
    image: ese7en/node-port-test
    restart: always
    network_mode: "host"
    environment:
      - PORTS=31400,31401,31402,31403,31404,31405,31406,31407,31408,31409
```

#### Run:
```sh
docker compose up -d
```

---

## Debugging & Troubleshooting

### Check if container is running:
```sh
docker ps
```

### Check logs:
```sh
docker logs <container_id>
```

### Verify open ports:
```sh
sudo lsof -i :31400-31409
```

### Check response:
```sh
curl http://localhost:31400
```

### Confirm port accessibility (from another machine):
```sh
curl http://<your-imac-ip>:31400
```

### Check external accessibility:
Use [yougetsignal.com/tools/open-ports](https://yougetsignal.com/tools/open-ports)

---

## 🛠️ Common Issues & Fixes

| Error | Fix |
|------|-----|
| `EADDRINUSE` | Kill process using the port: `sudo lsof -i :31409` then `sudo kill -9 <PID>` |
| Port not listening | Restart Docker, check `docker run` args, verify pf rules |
| Env variable not read | Always use `-e PORTS="..."` explicitly |
| pf warning | Normal on High Sierra, ALTQ is not supported and not needed |

---

## Final Check

You should see:

- `curl http://localhost:31400` returns `OK FROM PORT 31400`
- Same response from another machine on LAN
- Port test passes on public IP (if port forwarding is set)

---

## Optional Security (pf rules)

To restrict access to specific IPs:
```pf
pass in quick proto tcp from 203.0.113.5 to any port 31400:31409
block in quick proto tcp from any to any port 31400:31409
```

Then reload:
```sh
sudo pfctl -f /etc/pf.conf
```

---

## Lessons Learned

- Docker on older macOS versions can behave differently—expect quirks.
- Explicitly declaring ports and env vars helps a lot.
- Use `--network=host` if bridge mode misbehaves.
- Docker Compose makes restarts effortless.

---

## Credits

Thanks to the Pi Network community and open source tools like Docker & pf for making this possible!

```

