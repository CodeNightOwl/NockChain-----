[English Version](README_en.md) | [中文版](README.md)

# NockChain 挖矿部署教程 ubuntu22.04
#### 不建议用什么一键脚本，本来图简单，到时候各种报错不成功，建议自己部署同时参考官方文档

https://nockstats.com/  这里首页有项目状态
## 1.设置临时代理梯子，没有git不了（有外网跳过此步）
```bash
#（本地局域网代理机,或自己的代理）
export https_proxy=http://192.168.1.90:10809
```
### 2.安装rust
```bash
#首先安装 Rust,提示选择1即可
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Cargo 工具链添加到系统的环境变量 PATH 中，以便在命令行中可以直接使用 Cargo 命令
export PATH="$HOME/.cargo/bin:$PATH"

#验证安装是否成功,这两条命令应该能正确显示版本号而不尝试下载更新。
cargo --version
rustc --version

# Debian/Ubuntu 上运行，请确保已安装以下依赖项
sudo apt install clang llvm-dev libclang-dev
```
## 3.克隆nockchain的仓库
```bash
#如果上次拉了旧的，可以先删除之前的nockchain，重新clone
cd ~
rm -rf nockchain
```
```bash
#克隆仓库
git clone https://github.com/zorp-corp/nockchain.git
```
### 4.进入nockchain项目目录进行环境的编译启动
```bash
#进入项目目录
cd nockchain

#复制环境文件
cp .env_example .env

#如果没make,`make`只是`build-essential`中的一个组件,所以不要单独安装make，省不了什么空间
sudo apt update && sudo apt install build-essential

#安装 Hoon 编译器
make install-hoonc

#构建 Nockchain 和钱包二进制文件
make build

#安装钱包, 不同机器，可能等待较久
make install-nockchain-wallet

#安装nockchain，, 不同机器，可能等待较久
make install-nockchain
```

### 5.创建钱包，修改配置，
```bash
#生成新的密钥对
nockchain-wallet keygen
```
执行后会看到如下字样，保存下来,这是我们挖矿的钱包了
```bash
New Public Key
"3C9pWwmhxxxxxxxxxxxxxxxxxxxxxxxxx...."
New Private Key
"66ydDpsdkxPWMnJxxxxxxxxxxxxxxxxxxx..."
```
其他查看命令
```bash
#要备份密钥，请运行：
nockchain-wallet export-keys #这会将您的密钥保存到当前目录中调用的文件中keys.export。
#稍后可以通过以下方式导入它们：
nockchain-wallet import-keys --input keys.export

#我们可以用如下命令来显示24位助记词
nockchain-wallet show-seedphrase
#也可以查看公钥， 也就是我们new的时候看到的New Public Key
nockchain-wallet show-master-pubkey
#也可以查看私钥， 也就是我们new的时候看到的New Private Key
nockchain-wallet show-master-privkey
```
接下来我们要修改下.env 文件里的钱包公钥地址，为我们上面创建的公钥
```bash
#新手用nano 编辑器，比较友好一点，ctrl+x就是退出.   当然习惯用vi vim的也可
nano .env

改文件第3行 export MINING_PUBKEY := 创建的公钥
```
### 6.关闭防火墙
```bash
#我们先关闭防火墙(重启后会自动打开，需要手动关闭)
sudo ufw disable

#如果永久关闭防火墙,也可以如下操作
sudo systemctl stop ufw
sudo systemctl disable ufw
```

### 7.启动挖矿,按官方文档可以启动多个节点
```bash
#启用内存过量分配
sudo sysctl -w vm.overcommit_memory=1

#如果不在目录下，先cd到目录下,按上面步骤肯定是在nockchain目录下了
cd nockchain:

# 我们创建两个节点目录
mkdir node1 node2

#给两个节点复制一份配置文件
cp .env node1/
cp .env node2/

# 在其自己的目录中运行每个节点，如果你有多个节点，重复此步骤，如果只运行一个，就node1即可
cd node1 && sh ../scripts/run_nockchain_miner.sh
cd node2 && sh ../scripts/run_nockchain_miner.sh


#如上上面两行命令有问题，可以按如下

#例如启动第1个节点挖矿
cd ~/nockchain/node1
../scripts/run_nockchain_miner.sh

#例如启动第2个节点挖矿
cd ~/nockchain/node2
../scripts/run_nockchain_miner.sh



#如上用sh不成功，可以改成如下命令启动，修改对应的<PUB_KEY>为我们上面创建的公钥
# # 启动矿工并连接节点（可在nockchain电报群寻找优质节点）

#注意 如果您曾经停止过 node1，要再次重新启动它，请先删除旧的数据文件：
#当然也需要先cd到目录下
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
开始启动的时候提示一堆报错，不用担心

#### 怎么判定成功?
成功后一直刷新类似如下样子，提示有报错不用担心，
只要有 generating new candidate 就表示成功地挖掘了区块
```bash
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)(/ip4/175.9.43.23/udp/1025/quic-v1/p2p/12D3KooWLoiB7gDBRXHe975Qphy5LA6Enn7ucSkH6XEnv1fYWKjG: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)(/ip4/175.8.89.113/udp/17600/quic-v1/p2p/12D3KooWLoiB7gDBRXHe975Qphy5LA6Enn7ucSkH6XEnv1fYWKjG: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)(/ip4/175.8.89.113/udp/19906/quic-v1/p2p/12D3KooWLoiB7gDBRXHe975Qphy5LA6Enn7ucSkH6XEnv1fYWKjG: : Timeout has been reached)]
E (08:41:43) nc: Failed outgoing connection to Some(PeerId("12D3KooWQoFPaKKbM7UD942mANvQBMU7tVWyFpy8TykmVcXDNyQ6")): Failed to negotiate transport protocol(s): [(/ip4/120.244.161.68/udp/14384/quic-v1/p2p/12D3KooWQoFPaKKbM7UD942mANvQBMU7tVWyFpy8TykmVcXDNyQ6: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)]
E (08:41:43) nc: Failed outgoing connection to Some(PeerId("12D3KooWGNusarbpbLwdNQYFg5CEoGt2rVReDY5hoZJGQaVfUQtd")): Failed to negotiate transport protocol(s): [(/ip4/61.83.29.162/udp/55309/quic-v1/p2p/12D3KooWGNusarbpbLwdNQYFg5CEoGt2rVReDY5hoZJGQaVfUQtd: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)]
E (08:41:48) nc: Failed outgoing connection to Some(PeerId("12D3KooWEWvU983kpHUyAzQtisPyngPhKLGbHEWZ9TtYEbNje7Nb")): Failed to negotiate transport protocol(s): [(/ip4/216.18.207.50/udp/40591/quic-v1/p2p/12D3KooWEWvU983kpHUyAzQtisPyngPhKLGbHEWZ9TtYEbNje7Nb: : Multiple dial errors occurred:
 - Handshake with the remote timed out.: Multiple dial errors occurred:
 - Handshake with the remote timed out.: Handshake with the remote timed out.)]
I (08:41:49) nc: SEvent: friendship ended with 12D3KooWL5nxQ5mq8QVrP6XM4kquBZ96wYbh9t2nEZ4zPBhaFHtD via: Dialer { address: /ip4/34.18.98.38/udp/30000/quic-v1, role_override: Dialer, port_use: Reuse }. cause: Some(IO(Custom { kind: Other, error: Connection(ConnectionError(TimedOut)) }))
I (08:42:44) block 2G3iinY7LguLcXx5A73Dh22GUuFcNATkQ8fNg71v3rBk3ESNsrXu6QX added to validated blocks at 4
I (08:42:44) 2G3iinY7LguLcXx5A73Dh22GUuFcNATkQ8fNg71v3rBk3ESNsrXu6QX is new heaviest block
I (08:42:44) dumbnet: new heaviest block!
I (08:42:44) generating new candidate block with parent: 2G3iinY7LguLcXx5A73Dh22GUuFcNATkQ8fNg71v3rBk3ESNsrXu6QX
I (08:42:44) [%mining-on 17.379.422.570.636.714.290 2.540.074.895.902.743.373 7.862.909.409.740.633.746 7.792.854.681.471.865.165 3.363.312.769.252.194.957]
I (08:42:44) fact: heard-block
I (08:42:44) candidate block timestamp updated: 0x8000000d36ce4284
I (08:42:44) command: timer
I (08:42:44) fact: heard-block
I (08:42:44) fact: heard-block
I (08:42:44) fact: heard-block
I (08:42:44) fact: heard-block
I (08:42:45) fact: heard-block
I (08:42:45) fact: heard-block

```
#### 怎么查看余额?
查看钱包余额,注意文件路径(要cd 到 node1目录下)，启动挖矿的时候需另外启动一个命令行窗口
```bash
#进入到节点目录下
cd ~/nockchain/node1

# List all notes (UTXOs) that your node has seen
nockchain-wallet --nockchain-socket .socket/nockchain_npc.sock list-notes

#如果你有余额，将看到如下类似字样
- name: [first='xxxx' last='xxxx']
- assets: xxxx
- source: [p=[BLAH] is-coinbase=%.y]
```


### 8.(我这里必须要代理才能正常)如果做了第一步开启了代理，那么清掉HTTPS代理(可以跳过)
不用代理，能直接参与就节省代理流量，具体情况按实际操作.
```bash
unset https_proxy
```


### 9.网络错误,dns问题
```
1.如果是网络错误，就要搞好代理
2.如果dns有问题，就设置好dns(自行搜索，ai)
3.其他问题，看下面官方git的文档
```


### 10.如上都测试没问题了，那我们怎么后台多开？
```
1.可以用pm2等工具
2.可以用screen终端复用
3.可以用Docker容器来运行多个挖矿节点后台
等等,看自己的需求
```

```
#安装nvm,nvm是node的版本管理工具,如果用pm2的话
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
source ~/.bashrc
nvm install 18
```
#### 例: 用pm2来运行多个实例,在node1,node2 ...目录下创建nockchain.json文件
```
{
  "name": "nockchain-miner1",
  "script": "nockchain",
  "args": [
    "--mine",
    "--mining-pubkey",
    "上面创建的公钥",
    "--peer", "/ip4/95.216.102.60/udp/3006/quic-v1",
    "--peer", "/ip4/65.108.123.225/udp/3006/quic-v1",
    "--peer", "/ip4/65.109.156.108/udp/3006/quic-v1",
    "--peer", "/ip4/65.21.67.175/udp/3006/quic-v1",
    "--peer", "/ip4/65.109.156.172/udp/3006/quic-v1",
    "--peer", "/ip4/34.174.22.166/udp/3006/quic-v1",
    "--peer", "/ip4/34.95.155.151/udp/30000/quic-v1",
    "--peer", "/ip4/34.18.98.38/udp/30000/quic-v1",
    "--peer", "/ip4/96.230.252.205/udp/3006/quic-v1",
    "--peer", "/ip4/94.205.40.29/udp/3006/quic-v1",
    "--peer", "/ip4/159.112.204.186/udp/3006/quic-v1",
    "--peer", "/ip4/217.14.223.78/udp/3006/quic-v1"
  ],
  "env": {
    "RUST_LOG": "info,nockchain=info,nockchain_libp2p_io=info,libp2p=info,libp2p_quic=info",
    "MINIMAL_LOG_FORMAT": "true"
  },
  "log_date_format": "YYYY-MM-DD HH:mm Z",
  "error_file": "logs/error.log",
  "out_file": "logs/out.log",
  "merge_logs": true,
  "autorestart": true
}
```
```bash
#进入节点目录
cd ~/nockchain/node1

#重新启动，请先删除旧的数据文件：
rm -rf ./.data.nockchain .socket/nockchain_npc.sock

#用pm2启动挖矿节点
pm2 start nockchain.json

#查看pm2状态,如下面表格，内存占大约17.4G这个才是表示成功的
pm2 list

#查看日志，确定是否正常
pm2 logs nockchain-miner1

#保存pm2进程列表和状态
pm2 save
``
┌────┬─────────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name                │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼─────────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 1  │ nockchain-miner1    │ default     │ N/A     │ fork    │ 52925    │ 20m    │ 0    │ online    │ 0%       │ 17.4gb   │ ubuntu   │ disabled │
│ 2  │ nockchain-miner2    │ default     │ N/A     │ fork    │ 53320    │ 18m    │ 0    │ online    │ 0%       │ 17.4gb   │ ubuntu   │ disabled │
│ 4  │ nockchain-miner3    │ default     │ N/A     │ fork    │ 53662    │ 5m     │ 0    │ online    │ 0%       │ 17.4gb   │ ubuntu   │ disabled │
│ 5  │ nockchain-miner4    │ default     │ N/A     │ fork    │ 53789    │ 4m     │ 0    │ online    │ 0%       │ 2.4gb    │ ubuntu   │ disabled │
│ 6  │ nockchain-miner5    │ default     │ N/A     │ fork    │ 53903    │ 3m     │ 0    │ online    │ 0%       │ 724.0mb  │ ubuntu   │ disabled │
└────┴─────────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

### 11.那现在都正常了，我怎么知道机器的资源占用，到底能开几个？
```bash
#安装htop
sudo apt install htop
#启动htop
htop
```
有空闲资源可以按第7步继续加节点,cpu占用很低，但是内存需要很多，所以侧重内存占用

### 项目官方git
https://github.com/zorp-corp/nockchain/

### 钱包官方文档
https://github.com/zorp-corp/nockchain/blob/master/crates/nockchain-wallet/README.md