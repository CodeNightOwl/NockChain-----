# NockChain 挖矿部署教程 ubuntu22.04
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
cd node1
../scripts/run_nockchain_miner.sh

#例如启动第2个节点挖矿
cd node2
../scripts/run_nockchain_miner.sh
```

成功后一直刷新类似如下样子
```bash
2025-05-22T18:46:51.643456Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/1123/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=18}: quinn_proto::connection: sending 1200 bytes in 1 datagrams
2025-05-22T18:46:51.660065Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}:send{space=Initial pn=4}: quinn_proto::connection::packet_builder: PADDING * 871

```
查看钱包余额,注意文件路径(要cd 到 node1目录下)
```bash
# List all notes (UTXOs) that your node has seen
nockchain-wallet --nockchain-socket .socket/nockchain_npc.sock list-notes

# List all notes by pubkey
nockchain-wallet --nockchain-socket .socket/nockchain_npc.sock list-notes-by-pubkey <your-pubkey>
```


### 8.(我这里必须要代理才能正常)如果做了第一步开启了代理，那么清掉HTTPS代理(可以跳过)
不用代理，能直接参与就节省代理流量，具体情况按实际操作.
```bash
unset https_proxy
```


### 9.网络错误,dns问题
1.如果是网络错误，就要搞好代理
2.如果dns有问题，就设置好dns(自行搜索，ai)
3.其他问题，看下面官方git的文档


### 项目官方git
https://github.com/zorp-corp/nockchain/

### 钱包官方文档
https://github.com/zorp-corp/nockchain/blob/master/crates/nockchain-wallet/README.md