# NockChain 挖矿部署教程
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
#使 Rust 环境变量生效
source ~/.cargo/env
#验证安装是否成功,这两条命令应该能正确显示版本号而不尝试下载更新。
cargo --version
rustc --version
#安装依赖
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
cd nockchain
#安装make,`make`只是`build-essential`中的一个组件,所以不要单独安装make，省不了什么空间
sudo apt update && sudo apt install build-essential
make install-hoonc
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
### 6.启动挖矿，检测成功
```bash
#启动挖矿, 我这里用这个是不成功的
make run-nockchain

#如果上面一直报节点错误，那么就启动一个节点，同时启动挖矿(我看总是报错，所以自己这么理解了) 
#因此启动这个命令，成功挖矿
nockchain --mining-pubkey <上面的创建的公钥> --mine

```

成功后一直刷新类似如下样子
```bash
2025-05-22T18:46:51.643456Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/1123/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=18}: quinn_proto::connection: sending 1200 bytes in 1 datagrams
2025-05-22T18:46:51.643471Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/3007/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=20}:send{space=Initial pn=4}: quinn_proto::connection: CRYPTO: off 0 len 279
2025-05-22T18:46:51.643488Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/3007/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=20}:send{space=Initial pn=4}: quinn_proto::connection::packet_builder: PADDING * 862
2025-05-22T18:46:51.643507Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/3007/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=20}: quinn_proto::connection: sending 1200 bytes in 1 datagrams
2025-05-22T18:46:51.659892Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}: quinn_proto::connection: timeout timer=LossDetection
2025-05-22T18:46:51.659922Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}: quinn_proto::connection: PTO fired in_flight=3600 count=1 space=Initial
2025-05-22T18:46:51.659964Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}:send{space=Initial pn=3}: quinn_proto::connection: CRYPTO: off 0 len 270
2025-05-22T18:46:51.659983Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}:send{space=Initial pn=3}: quinn_proto::connection::packet_builder: PADDING * 871
2025-05-22T18:46:51.660008Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}: quinn_proto::connection: sending 1200 bytes in 1 datagrams
2025-05-22T18:46:51.660048Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}:send{space=Initial pn=4}: quinn_proto::connection: CRYPTO: off 0 len 270
2025-05-22T18:46:51.660065Z TRACE Transport::dial{address=/ip4/82.66.206.63/udp/2101/quic-v1/p2p/12D3KooWDeTg8wLuZj6SMTrsNDmyfpt9bVevXrLgRGgcHSAa4rSo}:drive{id=24}:send{space=Initial pn=4}: quinn_proto::connection::packet_builder: PADDING * 871

```

### 7.(我这里必须要代理才能正常)如果做了第一步开启了代理，那么清掉HTTPS代理(可以跳过)
不用代理，能直接参与就节省代理流量，具体情况按实际操作.
```bash
unset https_proxy
```


### 8.网络错误,dns问题
1.如果是网络错误，就要搞好代理
2.如果dns有问题，就设置好dns(自行搜索，ai)



### 项目官方git
github.com/zorp-corp/nockchain/

### 钱包官方文档
github.com/zorp-corp/nockchain/blob/master/crates/nockchain-wallet/README.md