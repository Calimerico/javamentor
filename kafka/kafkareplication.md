### How kafka replication works?

#### Introduction

Imagine having 3 brokers with topic that has 1 partition replicated on all 3 brokers. You should have in mind that there
are different kinds of replicas:
- Leader: If you want to produce message for partition you must communicate with this partition replica
- Followers: Other replicas that are constantly replicating messages from the leader. Purpose of those replicas is that if leader become unavailable, one of them can become a leader and we will not loose data.
Among followers there is subtle difference:
- In sync replicas: Replicas that replicated all messages from the leader in last 10 seconds. This can be overriden with config
`replica.lag.time.max.ms`, default is `10000` or 10 seconds.
- Replicas that are not in sync: Replicas that haven't replicated all messages for whathever reason
Leader should know which replicas are in sync and which ones are out of sync.
