# c10udx0101 – Data Server

## Project Overview
The c10udx0101 – Data Server is a multifunctional homelab server designed to deliver local and cloud-based data services. This rack-ready deployment provides:
- GitLab CE for internal code hosting and version control.
- Nextcloud for file sync, collaboration, and secure storage.
- Samba for LAN-based file sharing across operating systems.
- PXE Boot for network-based deployment of tools, diagnostics, and OS installations.

This build demonstrates:
- Proficiency in provisioning and securing Linux servers.
- Network integration in a VLAN-segmented environment.
- Multi-service deployment with proper isolation and firewall rules.
- Clear, replicable documentation.

The goal is to showcase system administration and networking skills while laying the foundation for future deep-dive projects into each service.

## Environment
Hardware:
- Lark Box Mini PC 

Operating System:
- Ubuntu Server 24.04 LTS

Network:
- VLAN20 (10.0.20.0/24) – dedicated for lab data services
- Static IP: 10.0.20.20
- Gateway: 10.0.20.1
- DNS: local resolver, fallback to public

Access:
- SSH from trusted workstations only (VLAN20)
- Managed switch with VLAN tagging and trunk configuration

## Build Phases

### Phase 1 – Base OS Installation
1. Booted from Ubuntu Server ISO.
2. Hostname: homelab-file, User: cloudx0101, SSH enabled during install.
3. Configured static IP via Netplan:

```
   network:
     ethernets:
       enp2s0:
         addresses: [10.0.20.20/24]
         gateway4: 10.0.20.1
         nameservers:
   
          addresses: [10.0.20.1,8.8.8.8]
```
     version: 2
4. Updated packages:
```   
   sudo apt update && sudo apt upgrade -y
```   
5. Reserved IP in DHCP server.

### Phase 2 – Network and Firewall Baseline
1. Verified VLAN20 trunk port connection.
2. Enabled UFW:
```
   sudo ufw allow 22/tcp
   sudo ufw allow from 10.0.20.0/24 to any port 80 proto tcp
   sudo ufw allow from 10.0.20.0/24 to any port 443 proto tcp
```   
3. Confirmed connectivity from admin workstation.

### Phase 3 – GitLab CE Installation
1. Installed dependencies:
```
   sudo apt install -y curl openssh-server ca-certificates tzdata perl
```
2. Added GitLab repository:
   ```
   curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
3. Installed GitLab CE:
```   
   sudo EXTERNAL_URL="http://10.0.20.20" apt install gitlab-ce -y
```
4. Verified service and accessed web UI for initial root password setup.

### Phase 4 – Nextcloud Installation
1. Installed Apache, MariaDB, PHP, and extensions:
```   
   sudo apt install apache2 mariadb-server libapache2-mod-php php-gd php-mysql php-curl php-mbstring php-intl php-gmp php-bcmath php-xml php-zip unzip -y
```
2. Secured MariaDB and created Nextcloud DB/user.
3. Downloaded and extracted Nextcloud:
```   
   wget https://download.nextcloud.com/server/releases/latest.zip
   unzip latest.zip -d /var/www
   sudo chown -R www-data:www-data /var/www/nextcloud
```
4. Configured Apache vhost for Nextcloud.
5. Enabled required modules and restarted Apache.

### Phase 5 – Samba Server Installation
1. Installed Samba:
```
   sudo apt install samba -y
```
2. Created shared directory:
```
   sudo mkdir -p /srv/samba/share
   sudo chown nobody:nogroup /srv/samba/share
   sudo chmod 0775 /srv/samba/share
```
3. Added share definition in /etc/samba/smb.conf and restarted Samba.

### Phase 6 – PXE Boot Service
1. Installed Apache, TFTP, Syslinux, iPXE:
```   
   sudo apt install apache2 tftpd-hpa syslinux-common pxelinux ipxe -y
```
2. Created PXE root and copied bootloader files.
3. Configured PXE menu with basic entries (local boot, memtest).
4. Set DHCP options 66 (server IP) and 67 (boot file).
5. Verified PXE boot on test client.

## Service Overview
GitLab CE – Internal repo hosting, CI/CD potential, restricted to VLAN20.  
Nextcloud – LAN-based file sync and collaboration, web interface via Apache.  
Samba – Cross-platform LAN file sharing.  
PXE Boot – Network-based tool/OS deployment starter menu.

## Network/Service Diagram
```
               +-----------------------------+
               |     HOMELAB CORE SWITCH      |
               | VLAN20 (10.0.20.0/24)        |
               +--------------+---------------+
                              |
                       (Trunk Port)
                              |
                    +-------------------+
                    |  FILE/DATA SERVER |
                    |  10.0.20.20/24    |
                    +-------------------+
                    | GitLab CE (80/tcp)|
                    | Nextcloud (80/tcp)|
                    | Samba (137-139,445)|
                    | PXE Boot (69,80)  |
                    | SSH (22/tcp)      |
                    +-------------------+
```

## Security Snapshot
- UFW default deny inbound.
- Allowed ports restricted to VLAN20 subnets.
- SSH key-based authentication from admin workstations.
- No public exposure of management interfaces.
- Static IP with DHCP reservation to prevent conflicts.

## Validation & Testing
- GitLab – Created and cloned test repo from workstation.
- Nextcloud – Uploaded/downloaded test file from VLAN20 client.
- Samba – Mounted share from Windows 11 and Ubuntu clients, verified read/write.
- PXE Boot – Booted test laptop into memtest86+ via network.
- Firewall – Confirmed no access to services from non-VLAN20 networks.

## Lessons Learned
- The Lark Box is sufficient for this role, but not for high-load monitoring tasks.
- VLAN segmentation greatly simplifies service isolation and security.
- PXE menu should be expanded in future to include OS installers and rescue tools.
- Having a “rack-ready” baseline build speeds up service-specific lab work later.

## Future Work
- Configure SSL/TLS for GitLab and Nextcloud.
- Expand PXE boot menu with additional payloads.
- Integrate backup and snapshot strategy.
- Build separate monitoring node with more powerful hardware.

## Upcoming Projects and Future Direction
This project revealed that while the Lark Box Mini PC performs well as a dedicated data server, it does not have the hardware resources to handle more demanding continuous monitoring workloads. As a result, a future project will focus on sourcing a more powerful machine to serve as a dedicated monitoring node within the homelab. That system will run SIEM, IDS/IPS, and other real-time analysis tools as part of an enterprise-grade network monitoring and defense stack.

In addition, this rack-ready deployment of GitLab CE, Nextcloud, Samba, and PXE Boot is only the first phase. Each of these services will receive its own dedicated repository in the future, where their configurations, optimizations, and security hardening steps will be documented in detail. These follow-on repos will cover:
- Advanced GitLab CE configuration, CI/CD pipelines, user management, and backup strategies.
- Full Nextcloud deployment with SSL/TLS, federation, external storage, and user collaboration workflows.
- Samba configuration for fine-grained permissions, domain integration, and secure file access policies.
- PXE Boot menu expansion to include OS installers, recovery environments, and automated provisioning scripts.

## Related Repositories
- https://github.com/charlesX0101/homelab-build
- https://github.com/charlesX0101/ubuntu-secure-server-setup
- https://github.com/charlesX0101/opnsense-firewall-build
- https://github.com/charlesX0101/homelab-network-validation-routine

