title: mongodb-concurrent-findOneAndUpdate
date: 2017-03-05 16:22:30
tags:

- mongodb
- concurrent

---

## 环境
* mongodb server version: 3.2.7
* client driver: [node-mongodb-native#2.1.21](https://github.com/mongodb/node-mongodb-native)

sudo yum install -y mongodb-org-3.2.12 mongodb-org-server-3.2.12 mongodb-org-shell-3.2.12 mongodb-org-mongos-3.2.12 mongodb-org-tools-3.2.12

## node-mongodb-native连接池是否有效

[db.serverStatus](https://docs.mongodb.com/v3.2/reference/method/db.serverStatus/)

使用[node-mongodb-native#2.1.21](https://github.com/mongodb/node-mongodb-native)进行压测，压测过程中，通过mongostat看到的连接数量稳定，并没有因为tps变高导致connection飙高


## 并发场景下，是否导致锁和慢sql

通过[A few show tricks to find slow queries in mongodb](https://gist.github.com/rantav/3433277)的指导，开启profile

在[findAndModify/#upsert-and-unique-index](https://docs.mongodb.com/manual/reference/command/findAndModify/#upsert-and-unique-index)提到了upsert和unique-index。并发场景下，如果没有唯一索引，可能会导致insert多条数据。

在没有使用唯一索引的情况下，增大压测脚本的并发，command指标没有提高。使用索引之后，command段时间和压测并发接近，但是随着压测时间的增加，仍会出现慢sql

```log
2017-03-06T13:40:12.886+0800 I COMMAND  [conn151] command test.DailyStatsWithIndex command: findAndModify { findandmodify: "DailyStatsWithIndex", query: { cursor: "2017-03-06", routingKey: "online_pay_make", type: 3, role: 1 }, new: false, remove: false, upsert: true, update: { $inc: { distribution.9:6: 1, count: 1 } } } update: { $inc: { distribution.9:6: 1, count: 1 } } keysExamined:1 docsExamined:1 nMatched:1 nModified:1 keyUpdates:0 writeConflicts:0 numYields:0 reslen:13222 locks:{ Global: { acquireCount: { r: 1, w: 1 } }, Database: { acquireCount: { w: 1 } }, Collection: { acquireCount: { w: 1 } } } protocol:op_query 154ms
```

通过profile查看具体的信息

```bash
db.system.profile.find({ns:'xx'},{docsExamined:1,locks:1,millis:1}).sort({ts:-1})
```

结果类似如下，每个字段的含义见 [database-profiler#system.profile.locks](https://docs.mongodb.com/v3.2/reference/database-profiler/#system.profile.locks)

```json
{
    "op" : "command",
    "ns" : "test.DailyStatsWithIndex",
    "command" : {
        "findandmodify" : "DailyStatsWithIndex",
        "query" : {
            "cursor" : "2017-03-06",
            "routingKey" : "reject",
            "type" : 3,
            "role" : 3
        },
        "new" : false,
        "remove" : false,
        "upsert" : true,
        "update" : {
            "$inc" : {
                "distribution.8:19" : 1,
                "count" : 1
            }
        }
    },
    "updateobj" : {
        "$inc" : {
            "distribution.8:19" : 1,
            "count" : 1
        }
    },
    "keysExamined" : 1,
    "docsExamined" : 1,
    "nMatched" : 1,
    "nModified" : 1,
    "keyUpdates" : 0,
    "writeConflicts" : 17,
    "numYield" : 0,
    "locks" : {
        "Global" : {
            "acquireCount" : {
                "r" : NumberLong(18),
                "w" : NumberLong(18)
            }
        },
        "Database" : {
            "acquireCount" : {
                "w" : NumberLong(18)
            }
        },
        "Collection" : {
            "acquireCount" : {
                "w" : NumberLong(18)
            }
        }
    },
    "responseLength" : 12944,
    "protocol" : "op_query",
    "millis" : 484,
    "execStats" : {},
    "ts" : ISODate("2017-03-06T05:39:12.998Z"),
    "client" : "10.12.78.97",
    "allUsers" : [],
    "user" : ""
}
```

两个关注的点：**writeConflicts**和**locks**，再看另外一篇文章 [faq-concurrency-locking](https://docs.mongodb.com/v3.2/faq/concurrency/#faq-concurrency-locking)

[serverstatus.wiredTiger.concurrentTransactions](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.wiredTiger.concurrentTransactions)

## Reference

* [Collection.prototype.findOneAndUpdate](https://github.com/mongodb/node-mongodb-native/blob/V2.2.11/lib/collection.js#L2335)
* [Does findAndModify effectively lock the document to prevent read conflicts?](http://stackoverflow.com/questions/26845716/does-findandmodify-effectively-lock-the-document-to-prevent-read-conflicts)
* [MongoDB - Atomic Operations](http://www.tutorialspoint.com/mongodb/mongodb_atomic_operations.htm)
* [findAndModify](https://docs.mongodb.com/manual/reference/command/findAndModify)


