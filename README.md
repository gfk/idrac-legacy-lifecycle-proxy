# iDRAC Legacy Lifecycle Proxy

A Dockerized solution to fix firmware update failures (Error LC023/RED006) on legacy Dell PowerEdge servers (specifically 13th Generation / iDRAC8 and older).

## The Problem
As of 2024-2025, updating a PowerEdge R430/R530/R630/R730 via the iDRAC "HTTPS" method fails because:
1. **TLS Incompatibility:** iDRAC8 cannot negotiate modern TLS handshakes with Dell's Akamai CDN.
3. **New Calalog.xml.gz location:** The Dell `Catalog.xml.gz` file has been moved on the Dell server.

## The Solution
This project creates a two-stage "Bridge":
1. **DNS Resolver (Dnsmasq):** Spoofs `downloads.dell.com` to point to this local bridge.
2. **Reverse Proxy (Nginx):** Receives legacy HTTP requests, upgrades them to HTTPS with modern TLS, spoofs a valid Browser User-Agent, and relays the files to the iDRAC.

## Prerequisites
* Docker and Docker Compose installed on a Linux host (e.g., Debian/Ubuntu).
* Port 80 (HTTP) and Port 53 (DNS) must be available on the host. iDRAC does not support making requests to ports other than 80/tcp for HTTP and 53/udp for DNS.
* **Note on Windows/macOS**: While this may work on Docker Desktop, Linux is recommended to avoid Port 53 binding conflicts and to ensure reliable UDP (DNS) routing from external hardware.

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/gfk/idrac-legacy-lifecycle-proxy.git
   cd idrac-legacy-lifecycle-proxy
   ```
   
2. Edit `dnsmasq.conf` and replace `##SERVERIP##` with the IP address of your Docker host that's hosting the nginx reverse proxy.
   
4. Deploy the stack:
   ```bash
   docker-compose up -d
   ```
## iDRAC configuration

Go to the iDRAC Web Interface and perform the following steps:

### Network Settings
* Navigate to **iDRAC Settings** > **Network** > **IPv4 Settings**.
* Set **Static DNS Server 1** to your Docker Host IP you set in the `dnsmasq.conf` file above.
* Click **Apply**.

### Update and Rollback
* Navigate to **Overview** > **iDRAC Settings** > **Update and Rollback**.
* Change the **File Location** radio button to **HTTP**.
* In the **HTTP Address** field, enter: `downloads.dell.com`
  * *Note: Do not use the IP address here. The iDRAC must use the hostname so the local DNS resolver can intercept it.*
* Leave the **Path** and **Catalog Filename** fields blank.
* Click **Check for Update**.
