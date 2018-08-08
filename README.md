# Deploy eosio.sudo on Kylin Testnet


### 1. 使用 multisig 创建 eosio.sudo 账号

#### 第一步，生成创建账户的 transaction 文件。

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

#### 第二步，创建一个名为 newaccount_payload.json 的文件，文件内容如下：


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

#### 第三步，使用 newaccount_payload.json 生成最终创建账户的 transaction 数据：

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

#### 第四步，生成 setpriv 的 action data

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

#### 第五步，准备最终的 transaction data。

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

#### 第六步，生成 producer_permissions.json 文件

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

#### 第七步，Propose Approve and exec

发起 proposal：

```
$ cleos multisig propose_trx createsudo producer_permissions.json create_sudo_account_trx.json PROPOSER_ACCOUNT
```

BP 分别 approve

```
cleos multisig approve PROPOSER_ACCOUNT createsudo '{"actor": "BP_ACCOUNT", "permission": "active"}' -p BP_ACCOUNT
```

approval 达到 15/21 之后，执行该 proposal：

```
cleos multisig exec PROPOSER_ACCOUNT createsudo BP_ACCOUNT
```

*注意，任何账户都可以执行 approval 达到 15/21 的 proposal，即使该账户不是 BP。


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
     delegated:       0.0000 EOS           (total staked delegated to account from others)
     used:                 0 bytes
     available:        2.304 MiB  
     limit:            2.304 MiB  

cpu bandwidth:
     staked:          1.0000 EOS           (total stake delegated from account to self)
     delegated:       0.0000 EOS           (total staked delegated to account from others)
     used:                 0 us   
     available:        460.8 ms   
     limit:            460.8 ms   

producers:     <not voted>
```

至此，eosio.sudo 账户就创建成功了，接下来就是通过 multisig 部署 sudo 合约到该账户。


### 2. 使用 multisig 部署 eosio.sudo 合约

该 repo 已经包含 eosio.sudo 官方合约的编译版本，位于 eosio.sudo 目录，源码请参见：https://github.com/EOSIO/eos/tree/v1.1.4/contracts/eosio.sudo

#### 第一步，生成部署 eosio.sudo 合约的 transaction 数据

```
$ cleos set contract -s -j -d eosio.sudo ./eosio.sudo/ > deploy_sudo_contract_trx.json
Reading WAST/WASM from contracts/eosio.sudo/eosio.sudo.wasm...
Using already assembled WASM...
Publishing contract...
$ cat deploy_sudo_contract_trx.json
{
  "expiration": "2018-06-29T19:55:26",
  "ref_block_num": 18544,
  "ref_block_prefix": 562790588,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "setcode",
      "authorization": [{
          "actor": "eosio.sudo",
          "permission": "active"
        }
      ],
      "data": "00004d1a03ea30550000c8180061736d01000000013e0c60017f006000017e60027e7e0060017e006000017f60027f7f017f60027f7f0060037f7f7f017f60057f7e7f7f7f0060000060037e7e7e0060017f017f029d010803656e7610616374696f6e5f646174615f73697a65000403656e760c63757272656e745f74696d65000103656e760c656f73696f5f617373657274000603656e76066d656d637079000703656e7610726561645f616374696f6e5f64617461000503656e760c726571756972655f61757468000303656e760d726571756972655f6175746832000203656e760d73656e645f64656665727265640008030f0e0505050400000a05070b050b000904050170010202050301000107c7010b066d656d6f72790200165f5a6571524b3131636865636b73756d32353653315f0008165f5a6571524b3131636865636b73756d31363053315f0009165f5a6e65524b3131636865636b73756d31363053315f000a036e6f77000b305f5a4e35656f73696f3132726571756972655f6175746845524b4e535f31367065726d697373696f6e5f6c6576656c45000c155f5a4e35656f73696f347375646f34657865634576000d056170706c79000e066d656d636d700010066d616c6c6f630011046672656500140908010041000b02150d0a9a130e0b002000200141201010450b0b002000200141201010450b0d0020002001412010104100470b0a00100142c0843d80a70b0e002000290300200029030810060b9e0102017e027f410028020441206b2202210341002002360204200029030010050240024010002200418104490d002000101121020c010b410020022000410f6a4170716b22023602040b2002200010041a200041074b41101002200341186a2002410810031a2003290318100520032903182101200310013703002003200137030820032003290318200241086a2000410010074100200341206a3602040bfd0403027f047e017f4100410028020441206b220936020442002106423b2105412021044200210703400240024002400240024020064206560d0020042c00002203419f7f6a41ff017141194b0d01200341a5016a21030c020b420021082006420b580d020c030b200341d0016a41002003414f6a41ff01714105491b21030b2003ad42388642388721080b2008421f83200542ffffffff0f838621080b200441016a2104200642017c2106200820078421072005427b7c2205427a520d000b024020072002520d0042002106423b2105413021044200210703400240024002400240024020064204560d0020042c00002203419f7f6a41ff017141194b0d01200341a5016a21030c020b420021082006420b580d020c030b200341d0016a41002003414f6a41ff01714105491b21030b2003ad42388642388721080b2008421f83200542ffffffff0f838621080b200441016a2104200642017c2106200820078421072005427b7c2205427a520d000b200720015141c00010020b0240024020012000510d0042002106423b2105412021044200210703400240024002400240024020064206560d0020042c00002203419f7f6a41ff017141194b0d01200341a5016a21030c020b420021082006420b580d020c030b200341d0016a41002003414f6a41ff01714105491b21030b2003ad42388642388721080b2008421f83200542ffffffff0f838621080b200441016a2104200642017c2106200820078421072005427b7c2205427a520d000b20072002520d010b20092000370318200242808080808080a0aad700520d00200941003602142009410136021020092009290310370208200941186a200941086a100f1a0b4100200941206a3602040b8c0101047f4100280204220521042001280204210220012802002101024010002203450d00024020034180044d0d00200310112205200310041a200510140c010b410020052003410f6a4170716b22053602042005200310041a0b200020024101756a210302402002410171450d00200328020020016a28020021010b200320011100004100200436020441010b4901037f4100210502402002450d000240034020002d0000220320012d00002204470d01200141016a2101200041016a21002002417f6a22020d000c020b0b200320046b21050b20050b0900418001200010120bcd04010c7f02402001450d00024020002802c041220d0d004110210d200041c0c1006a41103602000b200141086a200141046a41077122026b200120021b210202400240024020002802c441220a200d4f0d002000200a410c6c6a4180c0006a21010240200a0d0020004184c0006a220d2802000d0020014180c000360200200d20003602000b200241046a210a034002402001280208220d200a6a20012802004b0d002001280204200d6a220d200d28020041808080807871200272360200200141086a22012001280200200a6a360200200d200d28020041808080807872360200200d41046a22010d030b2000101322010d000b0b41fcffffff0720026b2104200041c8c1006a210b200041c0c1006a210c20002802c8412203210d03402000200d410c6c6a22014188c0006a28020020014180c0006a22052802004641d0c200100220014184c0006a280200220641046a210d0340200620052802006a2107200d417c6a2208280200220941ffffffff07712101024020094100480d000240200120024f0d000340200d20016a220a20074f0d01200a280200220a4100480d012001200a41ffffffff07716a41046a22012002490d000b0b20082001200220012002491b200941808080807871723602000240200120024d0d00200d20026a200420016a41ffffffff07713602000b200120024f0d040b200d20016a41046a220d2007490d000b41002101200b4100200b28020041016a220d200d200c280200461b220d360200200d2003470d000b0b20010f0b2008200828020041808080807872360200200d0f0b41000b870501087f20002802c44121010240024041002d00a643450d0041002802a84321070c010b3f002107410041013a00a6434100200741107422073602a8430b200721030240024002400240200741ffff036a41107622023f0022084d0d00200220086b40001a4100210820023f00470d0141002802a84321030b41002108410020033602a84320074100480d0020002001410c6c6a210220074180800441808008200741ffff037122084181f8034922061b6a2008200741ffff077120061b6b20076b2107024041002d00a6430d003f002103410041013a00a6434100200341107422033602a8430b20024180c0006a210220074100480d01200321060240200741076a417871220520036a41ffff036a41107622083f0022044d0d00200820046b40001a20083f00470d0241002802a84321060b4100200620056a3602a8432003417f460d0120002001410c6c6a22014184c0006a2802002206200228020022086a2003460d020240200820014188c0006a22052802002201460d00200620016a2206200628020041808080807871417c20016b20086a72360200200520022802003602002006200628020041ffffffff07713602000b200041c4c1006a2202200228020041016a220236020020002002410c6c6a22004184c0006a200336020020004180c0006a220820073602000b20080f0b02402002280200220820002001410c6c6a22034188c0006a22012802002207460d0020034184c0006a28020020076a2203200328020041808080807871417c20076b20086a72360200200120022802003602002003200328020041ffffffff07713602000b2000200041c4c1006a220728020041016a22033602c0412007200336020041000f0b2002200820076a36020020020b7b01037f024002402000450d0041002802c04222024101480d004180c10021032002410c6c4180c1006a21010340200341046a2802002202450d010240200241046a20004b0d00200220032802006a20004b0d030b2003410c6a22032001490d000b0b0f0b2000417c6a2203200328020041ffffffff07713602000b0300000b0bcf01060041040b04b04900000041100b0572656164000041200b086f6e6572726f72000041300b06656f73696f000041c0000b406f6e6572726f7220616374696f6e277320617265206f6e6c792076616c69642066726f6d207468652022656f73696f222073797374656d206163636f756e74000041d0c2000b566d616c6c6f635f66726f6d5f6672656564207761732064657369676e656420746f206f6e6c792062652063616c6c6564206166746572205f686561702077617320636f6d706c6574656c7920616c6c6f636174656400"
    },{
      "account": "eosio",
      "name": "setabi",
      "authorization": [{
          "actor": "eosio.sudo",
          "permission": "active"
        }
      ],
      "data": "00004d1a03ea3055df040e656f73696f3a3a6162692f312e30030c6163636f756e745f6e616d65046e616d650f7065726d697373696f6e5f6e616d65046e616d650b616374696f6e5f6e616d65046e616d6506107065726d697373696f6e5f6c6576656c0002056163746f720c6163636f756e745f6e616d650a7065726d697373696f6e0f7065726d697373696f6e5f6e616d6506616374696f6e0004076163636f756e740c6163636f756e745f6e616d65046e616d650b616374696f6e5f6e616d650d617574686f72697a6174696f6e127065726d697373696f6e5f6c6576656c5b5d0464617461056279746573127472616e73616374696f6e5f68656164657200060a65787069726174696f6e0e74696d655f706f696e745f7365630d7265665f626c6f636b5f6e756d0675696e743136107265665f626c6f636b5f7072656669780675696e743332136d61785f6e65745f75736167655f776f7264730976617275696e743332106d61785f6370755f75736167655f6d730575696e74380964656c61795f7365630976617275696e74333209657874656e73696f6e000204747970650675696e74313604646174610562797465730b7472616e73616374696f6e127472616e73616374696f6e5f6865616465720314636f6e746578745f667265655f616374696f6e7308616374696f6e5b5d07616374696f6e7308616374696f6e5b5d167472616e73616374696f6e5f657874656e73696f6e730b657874656e73696f6e5b5d046578656300020865786563757465720c6163636f756e745f6e616d65037472780b7472616e73616374696f6e01000000000080545704657865630000000000"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

紧接着，将上述文件中的 ref_block_num 和 ref_block_prefix 改为 0，将 expiration 的日期向后推迟一定的时间（比如 1 天后），这个时间表示 transaction 的过期时间。由于这个 transaction 最终需要等待 BP 多签生效，因此过期时间不能太短。

#### 第二步，Propose， Approve and exec

假设 producer 依然是创建账户时候的情况，我们依然沿用 producer_permissions.json 文件。

发起 proposal：

```
$ cleos multisig propose_trx deploysudo producer_permissions.json deploy_sudo_contract_trx.json PROPOSER_ACCOUNT
```

BP 分别 approve

```
cleos multisig approve PROPOSER_ACCOUNT deploysudo '{"actor": "BP_ACCOUNT", "permission": "active"}' -p BP_ACCOUNT
```

approval 达到 15/21 之后，执行该 proposal：

```
cleos multisig exec PROPOSER_ACCOUNT deploysudo BP_ACCOUNT
```

*注意，任何账户都可以执行 approval 达到 15/21 的 proposal，即使该账户不是 BP。


Proposal 执行成功后，我们可以查看 eosio.sudo 的 code，应该可以看到类似下面的输出：

```
cleos get code eosio.sudo
1b3456a5eca28bcaca7a2a3360fbb2a72b9772a416c8e11a303bcb26bfe3263c
```

至此，eosio.sudo 合约就通过多签的方式部署成功了。

接下来，我们可以试着使用多签调用 sudo 合约，执行一些诸如更改 owner key 等操作。






