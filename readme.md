# Solr cloud

In folder `collection_backup` there are the backup of collection `omeka` made from `solrcloud`

In folder `omeka` there are the backup of solr core `omeka` made from `standalone`, and its data. 

```mermaid
classDiagram
class ZK {
    zookeeper:2181
    solrconfig.xml
    configSet()
}
ZK <|-- solr1
ZK <|-- solr2 
ZK <|-- `More Workers`
class solr1 {
    solrcloud:80
    data
}
class solr2 {
    solrcloud:80
    data
}
class `More Workers` {
    solrcloud:80
    data
}
```

### softCommit vs commit
- softCommit: the changes are not written to disk, but are visible to all other users.
- commit: the changes are written to disk, and are visible to all other users.

In the context of Solr cloud, softCommit triggers synchronization between nodes, while commit triggers synchronization between nodes and disk.

All the changes are written to the transaction log, which is used for recovery in case of failure. The transaction log is written to disk, and is synchronized between nodes.

Whatever not yet written to disk is lost in case of failure. In case of softCommit, the changes are lost, while in case of commit, the changes are not lost. 

### Collection commands
#### Create collection
```bash
curl "http://solr1:80/solr/admin/collections?action=CREATE&name=omeka&numShards=2&replicationFactor=2&maxShardsPerNode=2&collection.configName=omeka"
```
> **Note:** I tested relicationFactoer = Shards = 2, the number of shards and replication factor are the same. I am not aware/sure whether there will be side effects when they are different. What does this mean for HA?

#### Delete collection
```bash
curl "http://solr1:80/solr/admin/collections?action=DELETE&name=omeka"
```

#### Backup collection
```bash
curl "http://solr1:80/solr/omeka/replication?command=backup&location=/opt/solr/server/solr/collection_backup&name=omeka"
```
> **Note:** The backup folder is tricky to setup, permissions, sub-structure should be set correspondingly according to backup scheduling. 

#### Restore collection
```bash
curl "http://solr1:80/solr/omeka/replication?command=restore&location=/opt/solr/server/solr/collection_backup&name=omeka"
```
