# Test eosio.wrap on Kylin Testnet

### 准备测试账户

Kylin 测试网络新建了两个账户用于 sudo 合约测试，分别为 `updatemyauth` 和 `transmytoken`，两个账户的 active 和 owner key 均为 `EOS1111111111111111111111111111111114T1Anm`，如下所示：

```
cleos -u https://api-kylin.eoslaomao.com get account updatemyauth
permissions:
     owner     1:    1 EOS1111111111111111111111111111111114T1Anm
        active     1:    1 EOS1111111111111111111111111111111114T1Anm
memory:
     quota:     40.91 KiB    used:     2.926 KiB

net bandwidth:
     delegated:       1.0000 EOS           (total staked delegated to account from others)
     used:                 0 bytes
     available:        835.2 KiB
     limit:            835.2 KiB

cpu bandwidth:
     delegated:       1.0000 EOS           (total staked delegated to account from others)
     used:                 0 us
     available:        121.9 ms
     limit:            121.9 ms
```

```     

cleos -u https://api-kylin.eoslaomao.com get account transmytoken
permissions:
     owner     1:    1 EOS1111111111111111111111111111111114T1Anm
        active     1:    1 EOS1111111111111111111111111111111114T1Anm
memory:
     quota:     40.91 KiB    used:     2.926 KiB

net bandwidth:
     delegated:       1.0000 EOS           (total staked delegated to account from others)
     used:                 0 bytes
     available:        835.2 KiB
     limit:            835.2 KiB

cpu bandwidth:
     delegated:       1.0000 EOS           (total staked delegated to account from others)
     used:                 0 us
     available:        121.9 ms
     limit:            121.9 ms

EOS balances:
     liquid:          100.0000 EOS
     staked:            0.0000 EOS
     unstaking:         0.0000 EOS
     total:           100.0000 EOS
```

其中 transmytoken 账户有 100 EOS 的余额，用于 CASE 3 测试。

在线查看 Kylin 网络中的上述账户，请访问：

https://tools.cryptokylin.io/#/tx/updatemyauth

https://tools.cryptokylin.io/#/tx/transmytoken




### CASE 1. sudo 变更 updatemyauth 账户的 active key


#### 第一步，生成变更 active key 的 transaction 数据

我们将 updatemyauth 账户的 active key 改为 EOS6GR1JqeNmp5d7FGRt3Rx1BGQ1QxUk65nkfcnuprAs1BVNNWqSD

首先，我们生成 updateauth 数据：

```
cleos set account permission -s -j -d updatemyauth active EOS6GR1JqeNmp5d7FGRt3Rx1BGQ1QxUk65nkfcnuprAs1BVNNWqSD owner > updatemyauth.json

cat updatemyauth.json

{
  "expiration": "2018-08-09T09:40:33",
  "ref_block_num": 8064,
  "ref_block_prefix": 90673647,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "updateauth",
      "authorization": [{
          "actor": "updatemyauth",
          "permission": "active"
        }
      ],
      "data": "d0b2365eaa6c52d500000000a8ed32320000000080ab26a701000000010002b57ac63750eca19d2e7a36ba46057e4d8a6a981849f883b5c731c33b86fdc82a01000000"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}

```

将上述文件中的 ref_block_num 和 ref_block_prefix 改为 0，将 expiration 改为 `1970-01-01T00:00:00`

#### 第二步，调用 sudo 合约执行上述命令，生成最终的 transaction data

生成 sudo_update_updatemyauth_active_trx.json 文件：

```
cleos wrap exec -s -j -d BP_ACCOUNT updatemyauth.json > sudo_update_updatemyauth_active_trx.json

cat sudo_update_updatemyauth_active_trx.json

{
  "expiration": "2018-08-09T09:43:33",
  "ref_block_num": 8424,
  "ref_block_prefix": 3790583261,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio.sudo",
      "name": "exec",
      "authorization": [{
          "actor": "eoslaomaocom",
          "permission": "active"
        },{
          "actor": "eosio.sudo",
          "permission": "active"
        }
      ],
      "data": "2029a246521331550000000000000000000000000000010000000000ea30550040cbdaa86c52d501d0b2365eaa6c52d500000000a8ed323243d0b2365eaa6c52d500000000a8ed32320000000080ab26a701000000010002b57ac63750eca19d2e7a36ba46057e4d8a6a981849f883b5c731c33b86fdc82a0100000000"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

将上述文件中的 ref_block_num 和 ref_block_prefix 设置为 0，将 expiration 的日期向后推迟一定的时间（比如 1 天后），这个时间表示 transaction 的过期时间。由于这个 transaction 最终需要等待 BP 多签生效，因此过期时间不能太短。


#### 第三步，生成 producer_permissions.json 文件

假设网络中有 21 个 BP，名字分别为，blkproducera，blkproducerb，......，blkproduceru。则 producer_permissions.json 的文件内容如下所示：

```
$ cat producer_permissions.json
[
   {"actor": "blkproducera", "permission": "active"},
   {"actor": "blkproducerb", "permission": "active"},
   {"actor": "blkproducerc", "permission": "active"},
   {"actor": "blkproducerd", "permission": "active"},
   {"actor": "blkproducere", "permission": "active"},
   {"actor": "blkproducerf", "permission": "active"},
   {"actor": "blkproducerg", "permission": "active"},
   {"actor": "blkproducerh", "permission": "active"},
   {"actor": "blkproduceri", "permission": "active"},
   {"actor": "blkproducerj", "permission": "active"},
   {"actor": "blkproducerk", "permission": "active"},
   {"actor": "blkproducerl", "permission": "active"},
   {"actor": "blkproducerm", "permission": "active"},
   {"actor": "blkproducern", "permission": "active"},
   {"actor": "blkproducero", "permission": "active"},
   {"actor": "blkproducerp", "permission": "active"},
   {"actor": "blkproducerq", "permission": "active"},
   {"actor": "blkproducerr", "permission": "active"},
   {"actor": "blkproducers", "permission": "active"},
   {"actor": "blkproducert", "permission": "active"},
   {"actor": "blkproduceru", "permission": "active"}
]
```


#### 第四步，发起多签 proposal，approve 并执行


发起 proposal：

```
$ cleos multisig propose_trx updateactive producer_permissions.json sudo_update_updatemyauth_active_trx.json PROPOSER_ACCOUNT
```

BP 分别 approve

```
cleos multisig approve PROPOSER_ACCOUNT updateactive '{"actor": "BP_ACCOUNT", "permission": "active"}' -p BP_ACCOUNT
```

approval 达到 15/21 之后，执行该 proposal：

```
cleos multisig exec PROPOSER_ACCOUNT updateactive ANY_ACCOUNT
```

*注意，任何账户都可以执行 approval 达到 15/21 的 proposal，即使该账户不是 BP。

Kylin 测试网络中已经提交 updateactive proposal：https://tools.cryptokylin.io/#/msig?proposer=eoslaomaocom&proposal=updateactive



### CASE 2. sudo 变更 updatemyauth 账户的 owner key

我们将 updatemyauth 账户的 owner key 改为 EOS6GR1JqeNmp5d7FGRt3Rx1BGQ1QxUk65nkfcnuprAs1BVNNWqSD

CASE 2 和 CASE 1 类似，唯一的不同在于第一步骤的时候，生成 updatemyauth.json 的命令稍有不同。

更改 owner key 的命令不需要指定 parent permission：

```
cleos set account permission -s -j -d updatemyauth owner EOS6GR1JqeNmp5d7FGRt3Rx1BGQ1QxUk65nkfcnuprAs1BVNNWqSD > updatemyauth.json
```

其余步骤请参考 CASE1。

Kylin 测试网络中已经提交 updateowner proposal：https://tools.cryptokylin.io/#/msig?proposer=eoslaomaocom&proposal=updateowner


### CASE 3. sudo 转移 transmytoken 账户中的 EOS

#### 第一步，生成 transfer data

我们将 transmytoken 账户的 1 EOS 转移给 eosio 账户，命令如下：


```
cleos transfer -s -j -d transmytoken eosio "1 EOS" "steal 1 EOS from you!" > transmytoken.json

cat transmytoken.json
{
  "expiration": "2018-08-09T09:52:27",
  "ref_block_num": 25799,
  "ref_block_prefix": 1477572235,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio.token",
      "name": "transfer",
      "authorization": [{
          "actor": "transmytoken",
          "permission": "active"
        }
      ],
      "data": "3015a4d94b3ccdcd0000000000ea3055102700000000000004454f530000000015737465616c203120454f532066726f6d20796f7521"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

将上述文件中的 ref_block_num 和 ref_block_prefix 改为 0，将 expiration 改为 `1970-01-01T00:00:00`


其余步骤请参考 CASE1。

Kylin 测试网络中已经提交 transtoken proposal：https://tools.cryptokylin.io/#/msig?proposer=eoslaomaocom&proposal=transtoken
