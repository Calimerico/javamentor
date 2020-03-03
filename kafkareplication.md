### How kafka replication works?

#### Introduction

Imagine having 3 brokers with topic that have 1 partition replicated on all 3 brokers. If you want to produce message to that
partition you must produce that message on broker which is leader for the partition. Later, other replicas send requests
to the leader
