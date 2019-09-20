官方文档路径

<https://developers.eos.io/eosio-home/docs>

### eos模块

#### eos节点nodeos

- `nodeos` (node + eos = nodeos) - the core EOSIO **node** daemon that can be configured with plugins to run a node. Example uses are block production, dedicated API endpoints, and local development.

#### 客户端 cleos

- `cleos` (cli + eos = cleos) - command line interface to interact with the blockchain and to manage wallets.

#### eos账户keosd

- `keosd` (key + eos = keosd) - component that securely stores EOSIO keys in wallets.

![EOSIO Architecture](https://files.readme.io/582e059-411_DevRelations_NodeosGraphic_Option3.png)



### nodeos



### cleos

`cleos` can be connected to different nodes by using the `--url` or `--wallet-url` optional arguments.s

#### 创建账户

root@wudq-virtual-machine:/usr/opt/eosio/1.7.3/bin# cleos wallet create -n eosyun --to-console
Creating wallet: eosyun
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5HtMwTXGYfmTbFpy83xPVLsnpjqEnqrNw5qmf8RGebGCmoZBn6S"



##### 密码

PW5HtMwTXGYfmTbFpy83xPVLsnpjqEnqrNw5qmf8RGebGCmoZBn6S

创建私钥

公钥：EOS7tiMf57ooGBqxUdiiSpM3hkXD1zCya8cv9sz99RUeUZ8iZHjAN

私钥：5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3

EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV



#### running nodeos

nodeos -e -p eosio --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --plugin eosio::state_history_plugin --data-dir /Users/mydir/eosio/data --config-dir /Users/mydir/eosio/config --access-control-allow-origin='*' --contracts-console --http-validate-host=false --state-history-dir /shpdata --trace-history --chain-state-history --verbose-http-errors --filter-on='*' --disable-replay-opts >> nodeos.log 2>&1 &





cleos create account eosio bob EOS7tiMf57ooGBqxUdiiSpM3hkXD1zCya8cv9sz99RUeUZ8iZHjAN 
cleos create account eosio alice EOS7tiMf57ooGBqxUdiiSpM3hkXD1zCya8cv9sz99RUeUZ8iZHjAN

Rest

幂等

服务熔断、限流

分布式锁

分布式事务