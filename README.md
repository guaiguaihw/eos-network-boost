## eos测试网络构建步骤
这里已经默认安装和编译好eos了，安装编译环节详见wiki：https://github.com/EOSIO/eos/wiki/Local-Environment

### 1.启动起始节点
区块的配置目录和数据目录默认放在以下目录，config目录里存放配置文件config.ini和genesis.json，data目录里存放区块数据。   
也可以自己指定config和data目录。   
```
mac os：~/Library/Application Support/eosio/nodeos/
Linux：~/.local/share/eosio/nodeos/
```
启动起始节点：
```
nodeos --config-dir /path/config --data-dir /path/data
```
