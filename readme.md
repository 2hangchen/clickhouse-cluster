# Clickhouse集群搭建-两台机器部署两分片两副本的集群

## 基本规划
### 节点IP
#### clickhouse
    192.168.1.151
        9100
        8123    
        9109
    192.168.1.152
        9200
        8223
        9209
#### zookeeper
    192.168.1.153:3181
    192.168.1.154:3181
    192.168.1.155:3181

## Clickhouse节点配置
|        | 副本01             | 副本02             |
|--------|--------------------|--------------------|
| 分片01 | 192.168.1.151:9100 | 192.168.1.152:9200 |
| 分片02 | 192.168.1.152:9100 | 192.168.1.151:9200 |


    config.xml（server01）
     <log>/var/log/clickhouse-server/clickhouse-server01.log</log>
    <errorlog>/var/log/clickhouse-server/clickhouse-server01.err.log</errorlog>
    <http_port>8123</http_port>
    <tcp_port>9100</tcp_port>
    <interserver_http_port>9109</interserver_http_port>

    config.xml（server02）
     <log>/var/log/clickhouse-server/clickhouse-server01.log</log>
    <errorlog>/var/log/clickhouse-server/clickhouse-server01.err.log</errorlog>
    <http_port>8123</http_port>
    <tcp_port>9100</tcp_port>
    <interserver_http_port>9109</interserver_http_port>

metrika.xml
<!--集群相关配置-->
    <clickhouse_remote_servers>
        <!--自定义集群名称 sbcluster_2shards_2replicas-->
        <sbcluster_2shards_2replicas>
            <!--分片1-->
            <shard>
                <internal_replication>true</internal_replication>
                <!--副本1-->
                <replica>
                    <host>192.168.1.151</host>
                    <port>9100</port>
                </replica>
                <!--副本2-->
                <replica>
                    <host>192.168.1.152</host>
                    <port>9200</port>
                </replica>
            </shard>
            <!--分片2-->
            <shard>
                <internal_replication>true</internal_replication>
                <!--副本1-->
                <replica>
                    <host>192.168.1.152</host>
                    <port>9100</port>
                </replica>
                <!--副本2-->
                <replica>
                    <host>192.168.1.151</host>
                    <port>9200</port>
                </replica>
            </shard>
        </sbcluster_2shards_2replicas>
    </clickhouse_remote_servers>
    <!--zookeeper相关配置-->
    <zookeeper-servers>
        <node index="1">
            <host>192.168.1.153</host>
            <port>3181</port>
        </node>
        <node index="2">
            <host>192.168.1.154</host>
            <port>3181</port>
        </node>
        <node index="3">
            <host>192.168.1.155</host>
            <port>3181</port>
        </node>
    </zookeeper-servers>
    <!--压缩算法-->
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
###### metrika中各节点具体配置
    #192.168.1.151 9100 metrika.xml
    <macros>
       <shard>01</shard>
       <replica>sbcluster-01-01</replica>
    </macros>

    #192.168.1.151 9200 metrika.xml
    <macros>
       <shard>02</shard>
       <replica>sbcluster-02-02</replica>
    </macros>

    #192.168.1.152 9100 metrika.xml
    <macros>
       <shard>02</shard>
       <replica>sbcluster-02-01</replica>
    </macros>

    #192.168.1.152 9200 metrika.xml
    <macros>
       <shard>01</shard>
       <replica>sbcluster-01-02</replica>
    </macros>

## Zookeeper节点规划
    192.168.1.153:3181
    192.168.1.154:3181
    192.168.1.155:3181

#### 创建两个目录分别存放两个副本的配置及数据
    clickhouse-server01
        conf
        data 
    clickhouse-server02
        conf
        data 

### nomad文件挂载相关配置
    #clickhouse01
    volumes = [
    "/etc/localtime:/etc/localtime",
    "/data/container/clickhouse/clickhouse/clickhouse-server01/conf/config.xml:/etc/clickhouse-server/config.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server01/conf/users.xml:/etc/clickhouse-server/users.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server01/conf/metrika.xml:/etc/clickhouse-server/metrika.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server01/data:/var/lib/clickhouse/",
    "/data/container/clickhouse/clickhouse/logs/clickhouse-server01.err.log:/var/log/clickhouse-server/clickhouse-server.err.log",
    "/data/container/clickhouse/clickhouse/logs/clickhouse-server01.log:/var/log/clickhouse-server/clickhouse-server.log"
    ]
    
    #clickhouse02
    volumes = [
    "/etc/localtime:/etc/localtime",
    "/data/container/clickhouse/clickhouse/clickhouse-server02/conf/config.xml:/etc/clickhouse-server/config.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server02/conf/users.xml:/etc/clickhouse-server/users.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server02/conf/metrika.xml:/etc/clickhouse-server/metrika.xml",
    "/data/container/clickhouse/clickhouse/clickhouse-server02/data:/var/lib/clickhouse/",
    "/data/container/clickhouse/clickhouse/logs/clickhouse-server02.err.log:/var/log/clickhouse-server/clickhouse-server.err.log",
    "/data/container/clickhouse/clickhouse/logs/clickhouse-server02.log:/var/log/clickhouse-server/clickhouse-server.log"
    ]

