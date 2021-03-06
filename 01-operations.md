
# Running multiple Elasticsearch Nodes

**NOTE: If running on a mac, you will need to give the Docker VM _lots_ of memory.**

Feel free to skip this.

To run multiple instances of Elasticsearch on your local machine:

```bash
./scripts/up-multi-node
```

Wait a while.  It will take quite a while for everything to start!

You will now have:

- A three-node Elasticsearch cluster for containing user data
- A one-node Elasticsearch cluster for containing monitoring data
- Two Kibana instances:
  - One for the main three node cluster at http://localhost:15601/
  - One for the monitoring cluster at http://localhost:25601/

Have a look at the Monitoring tab in Monitoring Kibanana 
http://localhost:25601/app/monitoring

# Catalog

Look at:

- http://localhost:9200/_cat
- http://localhost:9200/_cat/indices?v


# Controlling shard and replica counts

Open the dev tools tab of the main cluster

http://localhost:15601/app/kibana#/dev_tools

Try the following:

```
DELETE /test

PUT /test
 {
   "settings": {
     "number_of_replicas": 1,
     "number_of_shards": 7
   }
 }

POST /test/doc
{
  "title": "test doc"
}
```

Back in the monitoring Kibana:
To see the shard allocation, navigate to:

`Monitoring > Indices > test`

(scroll to the bottom of the page).

## Updating the replica count

```
PUT /test/_settings
{
  "number_of_replicas": 5
}
```

Look at shard allocation again (it may take a little while to update).

Use the "cat" interface to determine the health:

```
GET /_cat/indices?v
```

Change the replica count back to 1.

```
PUT /test/_settings
{
  "number_of_replicas": 1
}
```

And see how the replicas are never allocated to the same nodes as the primaries.

## You cannot update the shard count

Try the following **and see it fail**

```
PUT /test/_settings
{
  "number_of_shards": 4
}
```

**You cannot change the shard count on an index.**


# Cluster resiliency

Kill one of the nodes (not the master one yet).

Look at the allocation of shards to nodes.

Wait 60 seconds and have another look.
You should find that the shards have been re-allocated.

Restart the node and have another look.


# Master re-election

Kill the master node these URLs.
The dead node will respond with a 404, and the others will hang until they have elected a new master.

- http://localhost:19200/_cat/nodes?v
- http://localhost:29200/_cat/nodes?v
- http://localhost:39200/_cat/nodes?v

However, while this is happening, the remaining two nodes should serve queries.

- http://localhost:19200/test/_search
- http://localhost:29200/test/_search
- http://localhost:39200/test/_search

It will take about a minute, but the cluster should eventually elect a new master and rebalance the shards.


# Finally

Shutdown:

```
./scripts/destroy
```
