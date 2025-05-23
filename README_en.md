[中文版](README.md) | [English Version](README_en.md)

# NockChain Mining Deployment Tutorial ubuntu22.04
#### It is not recommended to use one-click scripts as they may cause various errors. Manual deployment while referring to the official documentation is preferred.

https://nockstats.com/ Project status can be checked on the homepage
## 1. Set up temporary proxy (Skip this step if you have direct internet access)
```bash
# (Local LAN proxy or your own proxy)
export https_proxy=http://192.168.1.90:10809
```
### 2. Install Rust
```bash
# First install Rust, select option 1 when prompted
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add Cargo toolchain to system PATH for direct command line usage
export PATH="$HOME/.cargo/bin:$PATH"

# Verify installation success - these commands should show version numbers without attempting updates
cargo --version
rustc --version

# For Debian/Ubuntu, ensure these dependencies are installed
sudo apt install clang llvm-dev libclang-dev
```
## 3. Clone nockchain repository
```bash
# If you previously cloned an old version, you can delete it first
cd ~
rm -rf nockchain
```
```bash
# Clone repository
git clone https://github.com/zorp-corp/nockchain.git
```
### 4. Compile and start the environment in nockchain project directory
```bash
# Enter project directory
cd nockchain

# Copy environment file
cp .env_example .env

# If make is not installed, note that `make` is just a component of `build-essential`
sudo apt update && sudo apt install build-essential

# Install Hoon compiler
make install-hoonc

# Build Nockchain and wallet binaries
make build

# Install wallet (may take longer on some machines)
make install-nockchain-wallet

# Install nockchain (may take longer on some machines)
make install-nockchain
```

### 5. Create wallet and modify configuration
```bash
# Generate new key pair
nockchain-wallet keygen
```
After execution, you will see the following output. Save this as it's your mining wallet:
```bash
New Public Key
"3C9pWwmhxxxxxxxxxxxxxxxxxxxxxxxxx...."
New Private Key
"66ydDpsdkxPWMnJxxxxxxxxxxxxxxxxxxx..."
```
Other useful commands:
```bash
# To backup keys, run:
nockchain-wallet export-keys # This saves your keys to a file called keys.export in current directory
# Later you can import them with:
nockchain-wallet import-keys --input keys.export

# Display 24-word seed phrase
nockchain-wallet show-seedphrase
# View public key (same as New Public Key shown above)
nockchain-wallet show-master-pubkey
# View private key (same as New Private Key shown above)
nockchain-wallet show-master-privkey
```
Next, modify the wallet public key address in .env file to the public key we created above
```bash
# Beginners can use nano editor (more user-friendly), press ctrl+x to exit
nano .env

# Modify line 3: export MINING_PUBKEY := [your public key]
```
### 6. Disable firewall
```bash
# First disable firewall (it will auto-enable after reboot)
sudo ufw disable

# To permanently disable firewall:
sudo systemctl stop ufw
sudo systemctl disable ufw
```

### 7. Start mining (multiple nodes can be launched as per official documentation)
```bash
# Enable memory overcommit
sudo sysctl -w vm.overcommit_memory=1

# If not in the directory, cd to it first
cd nockchain:

# Create two node directories
mkdir node1 node2

# Copy config file for both nodes
cp .env node1/
cp .env node2/

# Run each node in its own directory (repeat for additional nodes)
cd node1 && sh ../scripts/run_nockchain_miner.sh
cd node2 && sh ../scripts/run_nockchain_miner.sh

# Alternative commands if above doesn't work:

# Start first mining node
cd ~/nockchain/node1
../scripts/run_nockchain_miner.sh

# Start second mining node
cd ~/nockchain/node2
../scripts/run_nockchain_miner.sh

# Alternative startup command (replace <PUB_KEY> with your public key)
# Note: If you previously stopped node1, delete old data files first:
cd ~/nockchain/node1
rm -rf ./.data.nockchain .socket/nockchain_npc.sock

RUST_LOG=info,nockchain=info,nockchain_libp2p_io=info,libp2p=info,libp2p_quic=info \
MINIMAL_LOG_FORMAT=true \
nockchain --mine \
--mining-pubkey <PUB_KEY> \
--peer /ip4/95.216.102.60/udp/3006/quic-v1 \
--peer /ip4/65.108.123.225/udp/3006/quic-v1 \
--peer /ip4/65.109.156.108/udp/3006/quic-v1 \
--peer /ip4/65.21.67.175/udp/3006/quic-v1 \
--peer /ip4/65.109.156.172/udp/3006/quic-v1 \
--peer /ip4/34.174.22.166/udp/3006/quic-v1 \
--peer /ip4/34.95.155.151/udp/30000/quic-v1 \
--peer /ip4/34.18.98.38/udp/30000/quic-v1 \
--peer /ip4/96.230.252.205/udp/3006/quic-v1 \
--peer /ip4/94.205.40.29/udp/3006/quic-v1 \
--peer /ip4/159.112.204.186/udp/3006/quic-v1 \
--peer /ip4/217.14.223.78/udp/3006/quic-v1
```
Initial startup may show many errors - this is normal.

#### How to determine success?
Successful mining will continuously output messages like below (errors can be ignored).
As long as you see "generating new candidate", it means blocks are being successfully mined.
```bash
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)(/ip4/175.9.43.23/udp/1025/quic-v1/p2p/12D3KooWLoiB7gDBRXHe975Qphy5LA6Enn7ucSkH6XEnv1fYWKjG: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)(/ip4/175.8.89.113/udp/17600/quic-v1/p2p/12D3KooWLoiB7gDBRXHe975Qphy5LA6Enn7ucSkH6XEnv1fYWKjG: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple