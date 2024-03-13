# Capacity Planning

> The System Capacity Plan documents the projections of system capacity requirements for servers, disk, processing, memory and network requirements and the corresponding detail worksheets that include the document the details, formulae, and assumptions on which the estimates were based. For more information see the [Capacity Planning](https://ibm.biz/BdzZhy) work product in [IBM MethodHub](https://ibm.biz/BdzY8S).

### Prometheus

#### Scale

The Prometheus TSDB supports a local storage architecture but there is no support for clustering. A single instance of the local storage scales up to:

* 1 million+ samples/second
* Millions of series
* 1.3 bytes per sample

Suitable for keeping a few weeks or months of data. Beyond this, for HA, the recommendation is to run multiple, fully independent instances of Prometheus, each with identical data, and no knowledge of each other. Each instance then computes independently.

#### Remote Storage 

The local storage is not intended to be used for durable, long term, multi-year data retention. For long term storage of metrics, Prometheus supports sending data to an adapter that writes to a remote storage system such as InfluxDB, OpenTSDB, and/or Graphite. 

[Cortex](https://cortexmetrics.io/), hosted Prometheus in the cloud, provides a multi-tenant solution that includes long term data retention. 

#### Prometheus Data Storage

By default, Prometheus stores up to 15 days of metrics in a default `data` directory. The Docker containers running in the lab show the following utilization: 

| Info | Value |
| --- | --- |
| Scrape Frequency | 1 min |
| Metrics | 620 | 
| Data Retention Period | 15 days |
| Samples in Data Retention Period | 620 * (15 * 1,440 mins/day) = 13,392,000 | 
| Total Disk Utilization | 45.7 Mb |

```
[arodriguez@dal09-dev-analytics-01a cloud-analytics-poc]$ sudo docker exec -it prometheus1 du -csh data|sort -n|tail
45.7M	data
45.7M	total
```

```
[arodriguez@dal09-dev-analytics-01a cloud-analytics-poc]$ sudo docker exec -it prometheus1 ls -ltrh data
total 120K   
-rw-r--r--    1 root     root           0 Jan  7 04:34 lock
drwxr-xr-x    3 root     root        4.0K Jan 24 03:00 01EWS5BXB86XDQ5ZWS0WXW1310
drwxr-xr-x    3 root     root        4.0K Jan 24 21:00 01EWV35ENGF3CT5S4N4W5CP1R2
drwxr-xr-x    3 root     root        4.0K Jan 25 15:00 01EWX0YZTJE9B44PWB0AES65NM
drwxr-xr-x    3 root     root        4.0K Jan 26 09:00 01EWYYRH7BN1JH3PQ3EAHM09ZQ
drwxr-xr-x    3 root     root        4.0K Jan 27 03:00 01EX0WJ2ARB8J360QYGS9G2HNR
drwxr-xr-x    3 root     root        4.0K Jan 27 21:00 01EX2TBKK6ABPTTT9D9Z45PZAZ
drwxr-xr-x    3 root     root        4.0K Jan 28 15:00 01EX4R54R7K949G5H23DBBGAEN
drwxr-xr-x    3 root     root        4.0K Jan 29 09:00 01EX6NYP41E5H3A66Z3950NCYS
drwxr-xr-x    3 root     root        4.0K Jan 30 03:00 01EX8KR78CWWH3BNN0TKHFTEHG
drwxr-xr-x    3 root     root        4.0K Jan 30 21:00 01EXAHHRKFV788FCC40S03A597
drwxr-xr-x    3 root     root        4.0K Jan 31 15:00 01EXCFB9SSM2AQQRQ71WPV9AQ6
drwxr-xr-x    3 root     root        4.0K Feb  1 09:00 01EXED4V475VTC3K1G6KNVMWEY
drwxr-xr-x    3 root     root        4.0K Feb  2 03:00 01EXGAYCB6XRNTHVJ3B3NB79R9
drwxr-xr-x    3 root     root        4.0K Feb  2 21:00 01EXJ8QXKT5AGQ0SV5Q4RAYH3B
drwxr-xr-x    3 root     root        4.0K Feb  3 15:00 01EXM6HESWS4MMHX04R6K48YCN
drwxr-xr-x    3 root     root        4.0K Feb  4 09:00 01EXP4B03VES7NV07XM3C2X2YW
drwxr-xr-x    3 root     root        4.0K Feb  5 03:00 01EXR24H969FZJDWM62WMX9WX8
drwxr-xr-x    3 root     root        4.0K Feb  5 21:00 01EXSZY2JQW6ERAYQ09QA2V4GA
-rw-r--r--    1 root     root       19.5K Feb  6 05:36 queries.active
drwxr-xr-x    3 root     root        4.0K Feb  6 15:00 01EXVXQKWY0M18DXQSZVS1DYF0
drwxr-xr-x    3 root     root        4.0K Feb  7 09:00 01EXXVH53WCJ8V11B69758H8TB
drwxr-xr-x    3 root     root        4.0K Feb  7 15:00 01EXYG4APN5W2AV6TJ9JPZKDN7
drwxr-xr-x    3 root     root        4.0K Feb  7 21:00 01EXZ4QG9QTRPCPVNBBFXH9TTR
drwxr-xr-x    3 root     root        4.0K Feb  7 21:00 wal
drwxr-xr-x    2 root     root        4.0K Feb  7 21:00 chunks_head
drwxr-xr-x    3 root     root        4.0K Feb  7 21:00 01EXZ4QGF83RMDJD74HAXNW313
```

<!-- Do not edit -->
<hr/>
<footer>
<span>Page Last Updated: {docsify-updated}</span>
</footer>
