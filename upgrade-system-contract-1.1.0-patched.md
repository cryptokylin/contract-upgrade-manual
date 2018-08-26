# Upgrade System Contract 1.1.0-patched

## 背景
主网上的 eosio 等系统账户的 RAM 资源没有限制（unlimited）。在 EOS 网络中进行 transfer 等操作的时候，如果对方是一个合约账户，存在 RAM 被对方账户恶意占用的风险。另外，还有一项涉及 namebid 功能的风险，具体细节请参见：https://www.eoscanada.com/en/update-on-the-august-21st-2018-security-update-to-the-eosio-system-contract-dubbed-v1.1.0-sec.patch-3

在彻底修复上述问题之前，Block.One 出了一个针对 1.1.0 系统合约的补丁，该补丁包括如下内容：

1. 在 eosio.system 合约内新增 setalimits 接口，该接口可以将系统账户的 RAM/NET/CPU 资源从无限（unlimited）限制为特定值。
2. 优化 bidrefund 逻辑。

1.1.0 的 patch 版本代码参见：https://github.com/EOSLaoMao/eosio.contracts/tree/laomao/v1.1.0-new-patch

1.1.0 patch 版本的 build 位于本项目的 `eosio.system-1.1.0-patched` 目录。

本次升级合约的目标：

1. 部署 patch 版 1.1.0 系统合约。
2. 将除了 eosio 之外的系统账户的 RAM 设定为该账户当前已用的 RAM 值。


## 升级系统合约

### 1. 生成升级合约的 transaction 数据

```
cleos set contract -s -j -d eosio eosio.system > upgrade_system_contract_trx.json
```

数据结构如下：

```

$ cat upgrade_system_contract_trx.json

{
  "expiration": "2018-08-26T12:44:31",
  "ref_block_num": 0,
  "ref_block_prefix": 0,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [
    {
      "account": "eosio",
      "name": "setcode",
      "authorization": [
        {
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": {
        "account": "eosio",
        "vmtype": 0,
        "vmversion": 0,
        "code": "CODECODECODECODE"
      }
    },
    {
      "account": "eosio",
      "name": "setabi",
      "authorization": [
        {
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": {
        "account": "eosio",
        "abi": "ABIABIABIABIABIABI"
      }
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```


### 2. 生成设置系统账户 RAM 的 transaction 数据

在升级合约时，主网上各个系统账户的 RAM 使用情况如下（单位 byte）：

```
eosio.bpay = 5184
eosio.vpay = 8720
eosio.msig = 221325
eosio.token = 217920
eosio.names = 46106292
eosio.ram = 305824
eosio.ramfee = 2688
eosio.saving = 2688
eosio.stake = 403952

```

拿账户 eosio.bpay 举例，通过调用 setalimits 接口，可以得到将 eosio.bpay 的 RAM 限制为 5184 byte 的 transaction 数据：

```
cleos push action -s -j -d eosio setalimits '{"account": "eosio.vpay", "ram_bytes": 8720, "net_weight": -1, "cpu_weight": -1}' > eosio.vpay.json
```

其文件内容如下：

```
cat eosio.vpay.json

{
  "expiration": "2018-08-26T07:26:02",
  "ref_block_num": 1245,
  "ref_block_prefix": 462537812,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [],
      "data": "0080377503ea30554014000000000000ffffffffffffffffffffffffffffffff"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

简单起见，本 repo 提供了生成限制所有系统账户 RAM 的脚本 `gen_ram_limit.sh`，使用方式如下：

```
./gen_ram_limit.sh > ramlimit.json
```

生成的 `ramlimit.json` 文件结构如下：

```
cat ramlimit.json

{
  "expiration": "2018-08-26T07:31:17",
  "ref_block_num": 1809,
  "ref_block_prefix": 2805690369,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "008037f500ea30554014000000000000ffffffffffffffffffffffffffffffff"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
{
  "expiration": "2018-08-26T07:31:17",
  "ref_block_num": 1809,
  "ref_block_prefix": 2805690369,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0080377503ea30551022000000000000ffffffffffffffffffffffffffffffff"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}

......

{
  "expiration": "2018-08-26T07:31:17",
  "ref_block_num": 1809,
  "ref_block_prefix": 2805690369,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0014341903ea3055f029060000000000ffffffffffffffffffffffffffffffff"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

`ramlimit.json` 文件中的 actions 数据就是我们需要的限制各系统账户的 transaction 数据。



### 3. 构造用于 proposal 的 transaction 数据。

1.将 `upgrade_system_contract_trx.json` 文件中的 `ref_block_num` 和 `ref_block_prefix` 改为 0，将 `expiration` 的日期向后推迟一定的时间（比如 1 天后），这个时间表示 transaction 的过期时间。由于这个 transaction 最终需要等待 BP 多签生效，因此过期时间不能太短。

2.将 `ramlimit.json` 中的 actions 数据，追加到  `upgrade_system_contract_trx.json` 原有的两个 action 之后（setcode setabi），最终得到的 `upgrade_system_contract_trx.json` 格式如下：

```
{
  "expiration": "2018-08-27T06:48:52",
  "ref_block_num": 0,
  "ref_block_prefix": 0,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setcode",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "CODECODECODE"
    },{
      "account": "eosio",
      "name": "setabi",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "ABIABIABI"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "008037f500ea30554014000000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000735802ea30558d60030000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "00b0926602ea3055b486bf0200000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "000090e602ea3055a0aa040000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "a0d492e602ea3055800a000000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "c0a6db0603ea3055800a000000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0014341903ea3055f029060000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "00a6823403ea30554053030000000000ffffffffffffffffffffffffffffffff"
    },{
      "account": "eosio",
      "name": "setalimits",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0080377503ea30551022000000000000ffffffffffffffffffffffffffffffff"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}

```

该文件包含 11 个 action，其中 1 个 setcode，1 个 setabi，9 个 setalimits。

 
#### 4. 生成 producer_permissions.json 文件

假设网络中有 21 个 BP，名字分别为 blkproducera，blkproducerb，......，blkproduceru。则 producer_permissions.json 的文件内容如下所示：

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

#### 5. 发起多签 proposal，approve 并执行

发起 proposal：

```
$ cleos multisig propose_trx upgradesys producer_permissions.json upgrade_system_contract_trx.json PROPOSER_ACCOUNT
```

通知各 BP 分别 approve：

```
cleos multisig approve PROPOSER_ACCOUNT upgradesys '{"actor": "BP_ACCOUNT", "permission": "active"}' -p BP_ACCOUNT
```

approval 达到 15/21 之后，执行该 proposal：

```
cleos multisig exec PROPOSER_ACCOUNT upgradesys ANY_ACCOUNT
```


#### 6. 升级之后的检查


1.检查 code hash 是否为 `ec02a22b7f15064f3ff86564ef9e34e2c68ac9061023b11b47e1adfceedf1368`：

```
cleos get code eosio

code hash: ec02a22b7f15064f3ff86564ef9e34e2c68ac9061023b11b47e1adfceedf1368
```

2.检查各个系统账户的 RAM 是否已经从 unlimited 变为固定值，比如 eosio.bpay：

```
cleos get account eosio.bpay

permissions:
     owner     1:    1 eosio@active,
        active     1:    1 eosio@active,
memory:
     quota:     5.062 KiB    used:     2.625 KiB

net bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```


至此，基于 1.1.0 的 patch 版本升级成功。


*注意，任何账户都可以执行 approval 达到 15/21 的 proposal，即使该账户不是 BP。
