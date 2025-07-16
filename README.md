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

## Drosera Operator Setup on Mac

### Method 1: Using Docker (Recommended)

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

### Method 2: Native Installation (Alternative)

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
