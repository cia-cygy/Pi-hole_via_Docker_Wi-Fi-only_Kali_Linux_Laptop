# Pi-hole_via_Docker_Wi-Fi-only_Kali_Linux_Laptop


Most tutorials assume your system has a standard eth0 interface and predictable Docker networking. If you're on a laptop connected over Wi-Fi, things break.

This guide helps you:

- Avoid Docker network misconfigurations
- Fix interface eth0 does not currently exist errors
- Fix connection refused during gravity updates
- Ensure Pi-hole can resolve and reach the internet
- Properly use network_mode: host on Wi-Fi-only setups

Pre-Requisites:

- A Linux machine (in this case: Kali Linux) connected via Wi-Fi
- Docker and Docker Compose installed
- Basic familiarity with Linux command line


✅ Step 1: Install Docker & Docker Compose

Avoid broken plugin conflicts by installing Docker from official sources:

```bash
sudo apt purge docker-buildx-plugin docker-compose-plugin docker-compose
sudo apt install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

📂 Step 2: Create Your Pi-hole Project

mkdir ~/pihole-docker && cd ~/pihole-docker
mkdir etc-pihole etc-dnsmasq.d

Then create your docker-compose.yml file in pihole-docker directory:
`sudo nano docker-compose.yml`

Copy and paste the below text into the docker-compose.yml file:

```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    network_mode: "host"  # <--- Important for Wi-Fi users
    environment:
      TZ: "Australia/Sydney"
      WEBPASSWORD: "yourpassword"
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
```

⚠️ Note:
We use "network_mode: host" to bypass Docker bridge networking, which causes outbound issues on Wi-Fi-only systems.
Don't use the ports: comment out the ports statements (If any) when using host networking.


🚀 Step 3: Launch Pi-hole

```bash
docker compose up -d
Wait a few seconds, then go to:
http://localhost/admin or http://<your-laptop-ip>:80/admin
```

Log in using the password you set in WEBPASSWORD.


⚠️ Common Issues & Fixes


❌ interface eth0 does not currently exist
Pi-hole tries to bind to eth0 by default, but Wi-Fi systems use wlan0 (Or similar).

Fix:
Edit the file inside the container:

```bash
docker exec -it pihole bash
echo "interface=wlan0" >> /etc/dnsmasq.d/99-custom.conf
exit
docker restart pihole
```

❌ connection refused when updating gravity (blocklists)

Even though DNS works, Pi-hole can't reach the internet (e.g., GitHub blocklists). This usually happens when using Docker's default bridge on Wi-Fi.

Fix:
Use network_mode: host (already done above).

You can confirm it works:

`docker exec -it pihole bash`
`curl https://google.com`

Then update blocklists:

`docker exec -it pihole pihole -g`

❌ Admin password doesn’t work?

Sometimes the WEBPASSWORD env variable is ignored if you restart Pi-hole without clearing volumes.

Fix manually:

`docker exec -it pihole bash`
`pihole setpassword`

Or reset volumes:

```bash
docker compose down -v
sudo rm -rf ./etc-pihole ./etc-dnsmasq.d
mkdir etc-pihole etc-dnsmasq.d
docker compose up -d
```

📶 Set Your Devices to Use Pi-hole DNS

Option 1: Set Pi-hole’s IP (192.168.1.xxx) as the Primary DNS in your router (This will essentially make Pi-hole the DNS server for all the devices that connect to your router)

Option 2: Manually set DNS on each device


🧠 My future additions would be:

Add Unbound to turn Pi-hole into a privacy-respecting recursive DNS server
Set up cloudflared to use DNS-over-HTTPS
Visualize query logs with Grafana + InfluxDB
Use Pi-hole as DHCP server if your router doesn’t allow custom DNS


💡 Final Thoughts

This tutorial avoids the usual pitfalls when setting up Pi-hole on Wi-Fi-only laptops:

- eth0 assumptions break things
- Docker networking doesn't always play nice with Wi-Fi
- Password env vars don't always get applied without a volume reset
- Gravity fails silently if Pi-hole has no outbound web access

If you're facing strange networking issues, always try using network_mode: host on a trusted local network — it’s the simplest way to give Pi-hole full access.


🙌 Contributions Welcome

Open an issue or PR if you’ve faced similar edge cases.
Let’s make Pi-hole easier to deploy on laptops and Kali/Arch-type systems.


## License

- 📘 **Text and tutorial content**: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- 💻 **Code snippets and examples**: [MIT License](https://opensource.org/licenses/MIT)

This tutorial and code were created with the assistance of ChatGPT. You are free to use and modify the material in accordance with the respective licenses.
