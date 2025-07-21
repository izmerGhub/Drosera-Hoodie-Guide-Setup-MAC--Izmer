# Drosera-Hoodie-Guide-Setup-MAC--Izmer
## Running Drosera Trap & Operator on Mac

Here's how to set up the Drosera Trap and Operator on macOS:

## Prerequisites for Mac
- macOS 10.15 (Catalina) or later
- Homebrew package manager installed
- At least 4 CPU cores and 8GB RAM recommended
- Ethereum private key with funds on Hoodi testnet

## Installation Steps

### 1. Install Homebrew (if not already installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. Install Dependencies
```bash
brew update && brew upgrade
brew install curl git wget lz4 jq make gcc nano automake autoconf tmux htop unzip
```

### 3. Install Docker for Mac
Download and install Docker Desktop from:
https://www.docker.com/products/docker-desktop/

Or via Homebrew:
```bash
brew install --cask docker
```

### 4. Install Required Tools

#### Drosera CLI
```bash
curl -L https://app.drosera.io/install | bash
source ~/.zshrc  # or ~/.bashrc if using bash
droseraup
```

#### Foundry CLI
```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.zshrc
foundryup
```

#### Bun (JavaScript runtime)
```bash
curl -fsSL https://bun.sh/install | bash
source ~/.zshrc
```

## Drosera Trap Setup

```bash
mkdir ~/my-drosera-trap
cd ~/my-drosera-trap

git config --global user.email "your_github_email@example.com"
git config --global user.name "your_github_username"

forge init -t drosera-network/trap-foundry-template
bun install
forge build
```

Edit the trap configuration:
```bash
nano drosera.toml
```
Add the following codes at `drosera.toml`:

### Trap Configuration (`drosera.toml`)

```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.helloworld]
path = "out/HelloWorldTrap.sol/HelloWorldTrap.json"
response_contract = "0x183D78491555cb69B68d2354F7373cc2632508C7"
response_function = "helloworld(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR_OPERATOR_WALLET_ADDRESS"]

# New Users/Migrate:
# address = "DELETE THIS LINE WHEN APPLYING" (it will generate address after apply trap config)

# Existing Users:
# If you've deployed a trap with your wallet previously (Hoodi not Holesky), add your trap address here:
# address = "TRAP_ADDRESS"
```

---

### Apply the Trap Config (New users/Migrate) / Re-apply (Existing users)

```bash
DROSERA_PRIVATE_KEY=your_eth_private_key_here drosera apply
```
---

### Check Trap in Dashboard

Go to [https://app.drosera.io/](https://app.drosera.io/)\
Connect your Drosera EVM wallet\
Change network to Hoodi\
Search your trap by wallet address or trap config address generated after applying\
You can send Bloom Boost or monitor your trap here

---

### Bloom Boost Trap 

Drosera lets you increase your trap’s priority on-chain by depositing Hoodi ETH to boost response speed.
To boost a trap:

```bash
drosera bloomboost --trap-address <trap_address> --eth-amount <amount>
```

---


## Drosera Operator Setup on Mac

### A. Method 1: Using Docker (Recommended)

1. Create a working directory:
```bash
mkdir ~/Drosera-Network
cd ~/Drosera-Network
```

2. Create `docker-compose.yaml`:
```bash
nano docker-compose.yaml
```

Paste the following (same as Ubuntu version):
```yaml
version: '3'
services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    network_mode: host
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
    volumes:
      - drosera_data:/data
    command: ["node"]
    restart: always

volumes:
  drosera_data:
```

3. Create `.env` file:
```bash
nano .env
```
Add:
```
ETH_PRIVATE_KEY=your_eth_private_key_here
VPS_IP=your_local_ip_here  # Use your Mac's local IP (find with ifconfig)
```

4. Pull and run the operator:
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
docker compose down -v
docker compose up -d
docker compose logs -f
```

### A. Method 2: Native Installation (Alternative)

1. Install Go:
```bash
brew install go
```

2. Clone the operator repository:
```bash
git clone https://github.com/drosera-network/drosera-operator.git
cd drosera-operator
```

3. Build and run:
```bash
go build
./drosera-operator --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com --eth-private-key your_private_key
```
## Allow Ports in macOS Firewall
Unblock the ports in macOS's built-in firewall:
```bash
# Allow TCP traffic on 31313 and 31314
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 31313,tcp,in,allow
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 31314,tcp,in,allow

# Restart firewall (optional)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --restart
```

## B. Register your Operator 

```bash
drosera-operator register \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

## C. Opt-in your trap config 
```bash
drosera-operator optin \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --trap-config-address your_trap_address_here
```

---
# 🟥🟥 I DONE EVERYTHING MY NODE STILL RED :( 🟥🟥

##  How to Fix common IP/firewall Issues Red Node

### A. ** 🛡️Step 1: VPS Setup (Safe Config)** [ON VPS]
**1. Rent VPS (Ubuntu 22.04 LTS)**  
- **Must-have**: Public IPv4 
- **Critical**: Enable **SSH (port 22)** in firewall during setup.

🔥 **RackNerd** – Reliable low-cost KVM VPS with locations worldwide  
➡️ [Explore RackNerd Offers](https://my.racknerd.com/aff.php?aff=14994) <!-- hidden referral -->

💰 **HostVDS** – Russian-based affordable VPS with custom configuration  
➡️ [Check HostVDS Plans](https://hostvds.com/?affiliate_uuid=f3d517f2-6e58-4549-9ecd-d280fa8cea3c) <!-- hidden referral -->


**2. Configure WireGuard on VPS**  
```bash
sudo apt update && sudo apt install -y wireguard resolvconf
sudo su
umask 077
wg genkey > /etc/wireguard/privatekey
wg pubkey < /etc/wireguard/privatekey > /etc/wireguard/publickey
# View the keys  <VPS_PRIVATE_KEY> <VPS_PUBLIC_KEY>
cat /etc/wireguard/privatekey
cat /etc/wireguard/publickey
```

**3. Create Safe WireGuard Config** (`nano /etc/wireguard/wg0.conf`)  
```ini
[Interface]
PrivateKey = <VPS_PRIVATE_KEY>
Address = 10.8.0.1/24
ListenPort = 51820
# Preserve SSH access + internet
PostUp = sysctl -w net.ipv4.ip_forward=1
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 31313 -j DNAT --to-destination 10.8.0.2
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 31314 -j DNAT --to-destination 10.8.0.2
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
# NO MASQUERADE (keeps internet working)
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT

[Peer]
PublicKey = <MAC_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32
```

---
**4. Save and Start it**  
   ```bash
   sudo chmod 600 /etc/wireguard/wg0.conf
   sudo systemctl enable wg-quick@wg0
   sudo systemctl start wg-quick@wg0
   ```
**5. VPS Firewall Open:**
   ```bash
   sudo ufw allow 51820/udp
   sudo ufw allow 22/tcp
   sudo ufw allow 31313/tcp
   sudo ufw allow 31314/tcp
   ```

### B. **💻 Step 2:Mac Setup (No Internet Kill)** [On Mac Device]
**1. Install WireGuard GUI**  
```bash
# Install WireGuard using Homebrew (if you don't have Homebrew: https://brew.sh)
brew install wireguard-tools

# Generate keys (no sudo needed, stored in current directory)
umask 077
wg genkey > privatekey
wg pubkey < privatekey > publickey

# View the keys <MAC_PRIVATE_KEY> <MAC_PUBLIC_KEY>
cat privatekey
cat publickey
```

**2. Create Mac Config** (`nano ~/wg0.conf`)  
```ini
[Interface]
PrivateKey = <MAC_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 8.8.8.8  # Preserves internet
# Exclude SSH traffic from tunnel
Table = off
PostUp = route -n add -net <VPS_IP> -gateway <YOUR_DEFAULT_GW>
PostUp = route -n add -net 10.8.0.0/24 -interface utun0

[Peer]
PublicKey = <VPS_PUBLIC_KEY>
Endpoint = <VPS_IP>:51820
AllowedIPs = 10.8.0.0/24  # Only tunnel VPS traffic
PersistentKeepalive = 25
```
Get <VPS_IP>: Run `curl ifconfig.me` (on VPS)

Get <YOUR_DEFAULT_GW>: Run `ip route | grep default` (on macOS)

**3. Activate Tunnel**  
```bash
sudo wg-quick up ~/wg0.conf
```
**4. Allow Ports in macOS Firewall**  
macOS uses `socketfilterfw` for firewall rules. Run these commands to open **TCP 31313, 31314, 22 (SSH)** and **UDP 51820 (WireGuard)**:  

```bash
# Allow TCP ports (SSH, custom apps)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 31313,tcp,in,allow
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 31314,tcp,in,allow
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 22,tcp,in,allow

# Allow UDP port (WireGuard VPN)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --addport 51820,udp,in,allow

# Optional: Verify and restart firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --list
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --restart
```
---

### C. **🔌 Port Forwarding (Safe Method)**
**On VPS**:  
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 31313 -j DNAT --to-destination 10.8.0.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 31314 -j DNAT --to-destination 10.8.0.2
sudo iptables -A FORWARD -p tcp --dport 31313 -j ACCEPT
sudo iptables -A FORWARD -p tcp --dport 31314 -j ACCEPT
```

**Verify SSH is Alive**:  
```bash
ssh user@<VPS_IP>  # Should still work!
```

---

### **🌐 Internet Preservation Tests**
1. **On Mac**:  
   ```bash
   curl ifconfig.me  # Should show YOUR IP
   ping google.com   # Should work
   ```
2. **On VPS**:  
   ```bash
   sudo tcpdump -i eth0 port 22  # Should see your SSH traffic
   ```
### D. **🐳 Step 3: Update your docker compose**
1. Update `docker-compose.yaml`:
```bash
nano docker-compose.yaml
```

Paste the following (same as Ubuntu version):
```yaml
version: '3'
services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    network_mode: none  # 👇 Disable Docker networking (we'll use WireGuard's network)
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=10.8.0.2  # 👇 Bind to WireGuard IP
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}  # Public IP of VPS
      - DRO__SERVER__PORT=31314
    volumes:
      - drosera_data:/data
    restart: always
    # 👇 Critical: Connect container to host's WireGuard interface
    network_mode: "service:wg-service"  # Requires Docker 20.10+

volumes:
  drosera_data:
```

2. Update `.env` file:
```bash
nano .env
```
Add:
```
ETH_PRIVATE_KEY=your_eth_private_key_here
VPS_IP=your_rent_vps_ip_here  # New Public static IP (find with ifconfig)
```
3. Re-run the operator:
```bash
docker compose down -v
docker compose up -d
docker compose logs -f
```

## Important Notes for Mac Users

1. Firewall Configuration:
   - You'll need to allow incoming connections on ports 31313 and 31314 in macOS Security & Privacy settings

2. Network Considerations:
   - If you're behind a router, you may need to set up port forwarding
   - For best results, consider using a cloud VPS instead of your local Mac

3. Performance:
   - Macs may have slightly different performance characteristics than Linux servers
   - Monitor your system resources in Activity Monitor

4. Docker Differences:
   - Docker on Mac runs in a lightweight VM, which may affect networking
   - Use `host.docker.internal` instead of localhost for some configurations

For troubleshooting, refer to the official Drosera documentation and Mac-specific Docker documentation.
