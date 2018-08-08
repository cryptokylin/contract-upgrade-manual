# Deploy eosio.sudo on Kylin Testnet


### 1. 使用 multisig 创建 eosio.sudo 账号

####第一步，生成创建账户的 transaction 文件。

```
cleos system newaccount -s -j -d --transfer --stake-net "1.000 EOS" --stake-cpu "1.000 EOS" --buy-ram-kbytes 50 eosio eosio.sudo EOS8MMUW11TAdTDxqdSwSqJodefSoZbFhcprndomgLi9MeR2o8MT4 > generated_account_creation_trx.json
```

上述命令中的公钥可随意指定，因为最终创建 eosio.sudo 的账号的 transaction 中使用的是 eosio@active 权限。

上述命令生成的 generated_account_creation_trx.json 文件内容如下：

```
$ cat generated_account_creation_trx.json

{
  "expiration": "2018-06-29T17:11:36",
  "ref_block_num": 16349,
  "ref_block_prefix": 3248946195,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "newaccount",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea305501000000010003c8162ea04fed738bfd5470527fd1ae7454c2e9ad1acbadec9f9e35bab2f33c660100000001000000010003c8162ea04fed738bfd5470527fd1ae7454c2e9ad1acbadec9f9e35bab2f33c6601000000"
    },{
      "account": "eosio",
      "name": "buyrambytes",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea305500c80000"
    },{
      "account": "eosio",
      "name": "delegatebw",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea3055102700000000000004535953000000001027000000000000045359530000000001"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

####第二步，创建一个名为 newaccount_payload.json 的文件，文件内容如下：


```
$ cat newaccount_payload.json

{
   "creator": "eosio",
   "name": "eosio.sudo",
   "owner": {
      "threshold": 1,
      "keys": [],
      "accounts": [{
         "permission": {"actor": "eosio", "permission": "active"},
         "weight": 1
      }],
      "waits": []
   },
   "active": {
      "threshold": 1,
      "keys": [],
      "accounts": [{
         "permission": {"actor": "eosio", "permission": "active"},
         "weight": 1
      }],
      "waits": []
   }
}
```

####第三步，使用 newaccount_payload.json 生成最终创建账户的 transaction 数据：

使用 newaccount_payload.json 作为 newaccount 的 action data，生成最终创建账户的 transaction 数据

```
$ cleos push action -s -j -d eosio newaccount newaccount_payload.json -p eosio > generated_newaccount_trx.json
$ cat generated_newaccount_trx.json
{
  "expiration": "2018-06-29T17:11:36",
  "ref_block_num": 16349,
  "ref_block_prefix": 3248946195,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "newaccount",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea30550100000000010000000000ea305500000000a8ed32320100000100000000010000000000ea305500000000a8ed3232010000"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

####第四步，生成 setpriv 的 action data

生成 setpriv 的 action data，用于将 eosio.sudo 账户设置为 privileged account：

```
$ cleos push action -s -j -d eosio setpriv '{"account": "eosio.sudo", "is_priv": 1}' -p eosio > generated_setpriv_trx.json
$ cat generated_setpriv_trx.json
{
  "expiration": "2018-06-29T17:11:36",
  "ref_block_num": 16349,
  "ref_block_prefix": 3248946195,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setpriv",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "00004d1a03ea305501"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}

```

####第五步，准备最终的 transaction data。

将 generated_newaccount_trx.json 复制一份，命名为 create_sudo_account_trx.json

编辑 create_sudo_account_trx.json 文件内容，将 ref_block_num 和 ref_block_prefix 设置为 0，将 expiration 的日期向后推迟一定的时间（比如 1 天后），这个时间表示 transaction 的过期时间。由于这个 transaction 最终需要等待 BP 多签生效，因此过期时间不能太短。

将 generated_account_creation_trx.json 中除了第一个 action 之外的 action，追加到 create_sudo_account_trx.json 文件中 newaccount 这个 action 的后面。将 generated_setpriv_trx.json 文件中的 action 追加到 create_sudo_account_trx.json 文件中 action 列表的最后。

最终得到一个类似下面这一结构的 create_sudo_account_trx.json 文件，该文件可以被用于通过 multisig 提起 proposal 供 BP 多签进行升级。

```

$ cat create_sudo_account_trx.json
{
  "expiration": "2018-07-06T12:00:00",
  "ref_block_num": 0,
  "ref_block_prefix": 0,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "newaccount",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea30550100000000010000000000ea305500000000a8ed32320100000100000000010000000000ea305500000000a8ed3232010000"
    },{
      "account": "eosio",
      "name": "buyrambytes",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea305500c80000"
    },{
      "account": "eosio",
      "name": "delegatebw",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea305500004d1a03ea3055102700000000000004535953000000001027000000000000045359530000000001"
    },{
      "account": "eosio",
      "name": "setpriv",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "00004d1a03ea305501"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}

```

####第六步，生成 producer_permissions.json 文件

假设网络中有 21 个 BP，名字分别为，blkproducera，blkproducerb，......，blkproduceru。则 producer_permissions.json 的文件内容如下图所示：

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

####第七步，Propose Approve and exec

发起 proposal：

```
$ cleos multisig propose_trx createsudo producer_permissions.json create_sudo_account_trx.json PROPOSER_ACCOUNT
```

BP 分别 approve

```
cleos multisig approve blkproducera createsudo '{"actor": "BP_ACCOUNT", "permission": "active"}' -p BP_ACCOUNT
```

approval 达到 15/21 之后，执行该 proposal：

```
cleos multisig exec PROPOSER_ACCOUNT createsudo BP_ACCOUNT
```

*注意，任何账户都可以执行 approval 达到 15/21 账户，即使该账户不是 BP。


Proposal 执行成功后，我们可以查看 eosio.sudo 这个账户，应该可以看到类似下面的输出：


```
$ cleos get account eosio.sudo
privileged: true
permissions:
     owner     1:    1 eosio@active,
        active     1:    1 eosio@active,
memory:
     quota:     49.74 KiB    used:     3.33 KiB  

net bandwidth:
     staked:          1.0000 EOS           (total stake delegated from account to self)
     delegated:       0.0000 SYS           (total staked delegated to account from others)
     used:                 0 bytes
     available:        2.304 MiB  
     limit:            2.304 MiB  

cpu bandwidth:
     staked:          1.0000 EOS           (total stake delegated from account to self)
     delegated:       0.0000 SYS           (total staked delegated to account from others)
     used:                 0 us   
     available:        460.8 ms   
     limit:            460.8 ms   

producers:     <not voted>
```

至此，eosio.sudo 账户就创建成功了，接下来就是通过 multisig 部署 sudo 合约到该账户。