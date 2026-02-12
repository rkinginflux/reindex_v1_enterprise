## From each data node, do the following. 

1) Stop InfluxDB by stopping the influxd process.

`systemctl stop influxdb.service`

2) Add the following to the data section in your influxd.conf file; if the settings exist do not change. 

```bash
[data]
  index-version = "tsi1"
  cache-max-memory-size = 1000000000
  cache-snapshot-memory-size = 26214400
  max-index-log-file-size = "128k"
```
Additional info regarding the setting can be found here:

https://docs.influxdata.com/enterprise_influxdb/v1/administration/configure/config-data-nodes/#cache-max-memory-size
https://docs.influxdata.com/enterprise_influxdb/v1/administration/configure/config-data-nodes/#cache-snapshot-memory-size
https://docs.influxdata.com/enterprise_influxdb/v1/administration/configure/config-data-nodes/#max-index-log-file-size

3) Remove all _series directories. By default, _series directories are stored at /var/lib/influxdb/data/<dbName>/_series; make sure to double check the location of your data directory.

   `e.g.  find /var/lib/influxdb/data/ | grep _series$ | xargs rm -fr`

4) Remove all index directories. By default, index directories are stored at /var/lib/influxdb/data/<dbName/<rpName>/<shardID>/index. 
   
   `e.g.  find /var/lib/influxdb/data/ | grep index$ | xargs rm -fr`

5) Rebuild the TSI index. Use the influx_inspect command line client (CLI) to rebuild the TSI index, 
   make sure to include `-max-cache-size` & `-max-log-file-size` settings corrisponding to the `cache-max-memory-size` & `max-index-log-file-size` from the influxdb.conf file :

e.g.  `influx_inspect buildtsi -max-cache-size 1000000000 -max-log-file-size 131072 -datadir /var/lib/influxdb/data/ -waldir /var/lib/influxdb/wal/`

6) Start up the Influx service

`systemctl start influxdb.service`

### Suggestions
* Reindex each data node one at a time; don't reindex all data nodes at the same time.

* When using the influx_inspect buildtsi command, make sure to pay attention to the directory ownship of the directories in /var/lib/influxdb. 
  If the directories were owned by user influxdb, and if you run the influx_inspect buildtsi command as root, you'll need to change the permissions of the directories back to the influxdb user. 

* You may see Hinted-Handoff (HH) build up when you are reindexing the 1st data node. Before you continue to reindex the 2nd data node, make sure there's no HH build up. Once HH is gone, you can continue to the 2nd data node. 

* For the cache-max-memory-size & cache-snapshot-memory-size settings, I included the default settings in step 2. However, if needed, you can increase the settings to the following. 

```bash
[data]
  index-version = "tsi1"
  cache-max-memory-size = 2147483648
  cache-snapshot-memory-size = 52428800
  max-index-log-file-size = "128k"
```
