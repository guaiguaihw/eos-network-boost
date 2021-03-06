## eos测试网络构建步骤
这里已经默认安装和编译好eos了，安装编译环节详见wiki：https://github.com/EOSIO/eos/wiki/Local-Environment

### 1.启动起始节点
区块的配置目录和数据目录默认放在以下目录，config目录里存放配置文件config.ini，data目录里存放区块数据。 也可以自己指定config和data目录。
```
mac os：~/Library/Application Support/eosio/nodeos/
Linux：~/.local/share/eosio/nodeos/
```
启动起始节点：
```
nodeos --config-dir /path/config --data-dir /path/data
```

### 2.创建钱包，生成密钥对，导入密钥
生成的密钥对，用来做其他账户的owner和active的权限密钥。eosio也有默认的密钥对，在keys.txt文件中查看。
```
cleos wallet create
cleos create key
cleos import <private key>
```

### 3.加载系统合约、设置初始出块节点
```
cleos set contract eosio contracts/eosio.bios -p eosio
cleos push action eosio setprods '{"schedule":[{"producer_name":"eosio","block_signing_key":"$EOSIO_PRODUCER_PUB_KEY"}]}' -p eosio
```

### 4.创建一系列系统账号
需要创建的系统账号有：eosio.bpay、eosio.msig、eosio.names、eosio.ram、eosio.ramfee、eosio.saving、eosio.stake、eosio.token、eosio.vpay
```
cleos create account eosio eosio.token <owner public key> <active public key>
```

### 5.安装其他系统合约
eosio.token是用来发币的合约；eosio.msig是用来提出提案，更改宪法的合约
```
cleos set contract eosio.token contracts/eosio.token -p eosio.token
cleos set contract eosio.msig contracts/eosio.msig -p eosio.msig
cleos push action eosio setpriv '["eosio.msig",1]' -p eosio
```

### 6.发币
发行10亿eos，可以将10亿分给系统账户eosio，用于创建账户等操作
```
cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOS", 0, 0, 0]' -p eosio.token
cleos push action eosio.token issue '[ "eosio", "1000000000.0000 EOS", "memo"]'  -p eosio
```

### 7.安装投票系统合约
```
cleos set contract eosio contracts/eosio.system -p eosio
```

### 8.创建账户
这里会创建投票账户voter1、voter2、voter3，创建节点账户bp1、bp2、bp3。   
分给voter1、voter2、voter3分别1亿的eos，需要让bp出块，全网需要抵押1.5亿的eos(delegatebw cpu/net)，并执行投票(voteproducer)
```
cleos system newaccount eosio <account name> <owner_public_key> <active_public_key> --stake-net "0.2 EOS" --stake-cpu "0.1 EOS" --buy-ram "0.2 EOS"
cleos transfer eosio <voter account> "100000000.0000 EOS" "transfer to voter"
```

### 9.bp账号注册为出块节点
```
cleos system regproducer <bp account> <public key>
```

### 10.voter账户抵押、投票
注意，bp节点必须排序且不能重复
```
#每个voter账户抵押5亿eos
cleos system delegatebw <voter account> <voter account> "25000000.0000 EOS" "25000000.0000 EOS"
#每个voter投票给bp账户
cleos system voteproducer prods <voter account> bp1 bp2 bp3
#查看账户投票情况
cleos get account <voter account>
#查看账户余额
cleos get currency balance eosio.token <voter account> EOS
#查看现在的区块生产者
cleos system listproducers
```

### 11.启动其他节点
做完10步后，初始节点应该就不出块了。这里按照步骤一，启动其他的节点。    
修改配置文件config.ini，将producer_name改为<bp account>，将signature-provider替换成bp的公私钥。   
修改p2p-peer-address、p2p-listen-endpoint，让不同节点能彼此发现，互相同步区块

### 12.重新分配权限
将eosio的权限交给eosio.prods控制，eosio.prods是由21个超级节点控制的系统账户，生效交易需要15/21个节点签名     
```
cleos push action eosio updateauth '{"account": "eosio", "permission": "owner",  "parent": "",  "auth": { "threshold": 1, "keys": [], "waits": [], "accounts": [{ "weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"} }] } } ' -p eosio@owner
cleos push action eosio updateauth '{"account": "eosio", "permission": "active",  "parent": "owner",  "auth": { "threshold": 1, "keys": [], "waits": [], "accounts": [{ "weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"} }] } }' -p eosio@active
```
将其他的系统账户的权限交给eosio控制，这里系统账户有：eosio.bpay、eosio.msig、eosio.names、eosio.ram、eosio.ramfee、eosio.saving、eosio.stake、eosio.token、eosio.vpay   
```
cleos push action eosio updateauth '{"account": "比如eosio.token", "permission": "owner",  "parent": "",  "auth": { "threshold": 1, "keys": [], "waits": [], "accounts": [{ "weight": 1, "permission": {"actor": "eosio", "permission": "active"} }] } } ' -p eosio.token@owner
cleos push action eosio updateauth '{"account": "比如eosio.token", "permission": "active",  "parent": "owner",  "auth": { "threshold": 1, "keys": [], "waits": [], "accounts": [{ "weight": 1, "permission": {"actor": "eosio", "permission": "active"} }] } }' -p eosio.token@active

```

### 13.主网创世快照生成
快照生成工具地址：https://github.com/EOS-Mainnet/genesis    
快照导入节点工具地址：https://github.com/EOSArgentina/firestarter    
