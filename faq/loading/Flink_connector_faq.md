# Flink Connector FAQ

## flink-connector-jdbc_2.11sink is 8 hours late in StarRocks

**Issue description:**

The time generated by localtimestap function is normal in Flink. But it became 8 hours late when sunk to StarRocks. Flink server and StarRocks server are located in the same timezone, namely Asia/Shanghai UTC/GMT+08:00. Flink version is 1.12. Driver: flink-connector-jdbc_2.11. Can I ask how to resolve this issue?

**Solution:**

Please try configure the time parameter 'server-time-zone' = 'Asia/Shanghai' in Flink sink table. You may also add &serverTimezone=Asia/Shanghai in jdbc url. An example is shown below:

```sql
CREATE TABLE sk (
    sid int,
    local_dtm TIMESTAMP,
    curr_dtm TIMESTAMP
)
WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://192.168.110.66:9030/sys_device?characterEncoding=utf-8&serverTimezone=Asia/Shanghai',
    'table-name' = 'sink',
    'driver' = 'com.mysql.jdbc.Driver',
    'username' = 'doris',
    'password' = 'doris123',
    'server-time-zone' = 'Asia/Shanghai'
);
```

## In flink import, only the kafka clusters deployed in StarRocks clusters can be imported

**Issue description:**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**Solution:**

Kafka communication needs the hostname. Users need to configure the host name resolution /etc/hosts in StarRocks cluster nodes.

## Can StarRocks export 'create table statements' in batches?

**Solution:**

You can use Doris Tools to export the statements.

## When the query is not happening, BE memory usage and cpu usage are still 100%

**Issue description:**

Why are they still 100%?

**Solution:**

This is because of regular collection of statistics. CUP is not occupied for a long term. Memory of 10G or less, once fully taken, will not be released. BE will manage it by itself. Users can limit memory usage limit through tc_use_memory_min.

tc_use_memory_min is set to be 10737418240 by default. TCmalloc defines the minimum reserved memory. StarRocks only gives idle memory back to the operation system when the memory usage exceeds this value. Users can configure it in the BE configuration file "be.conf".

## Memory requested by BE is not released back to to the operation system

This is a normal phenomenon, as large blocks of memory allocated to the database from the operating system are reserved during allocation and deferred during release in order to reuse the memory and make memory allocation more convenient. It is recommended that users validate the test environment by monitoring memory usage over a longer period of time to see if the memory can be released.

## Flink connector does not work after being downloaded

**Issue description:**

This package needs to be obtained through Aliyun mirror address.

**Solution:**

Please make sure that the mirror part of /etc/maven/settings.xml is all configured to be obtained through Aliyun mirror address.

If it is, please change it to the following:

 <mirror>
    <id>aliyunmaven </id>
    <mirrorf>central</mirrorf>
    <name>阿里云公共仓库</name>
    <url>https：//maven.aliyun.com/repository/public</url>
</mirror>

## The meaning of parameter sink.buffer-flush.interval-ms in Flink-connector-StarRocks

**Issue description:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

If this parameter is set as 15s with checkpoint interval being equal to 5 mins, does this value still take effect?

**Solution:**

Whichever of the three thresholds is reached first, that one will take effect first. This is not affected by the checkpoint interval value which only works for exactly once. Interval-ms is used by at_least_once.
