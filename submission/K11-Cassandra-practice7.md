# K11-Cassandra-practice7

番号: sse02-19 日付：19/06/12

## 課題

演習マニュアルを元に以下の検証を行い、結果を元に説明をしてください。
2. 演習7 YCSBを用いて、１レコードのデータサイズを何通りか変えてCassandraの負荷検証を行い、 １レコードのデータサイズが増えるとどういった傾向が出るかとその理由を考察して説明してください

##### 取得データ

[![Image from Gyazo](https://i.gyazo.com/975c47458e5c953f61194419facb5133.png)](https://gyazo.com/975c47458e5c953f61194419facb5133)

##### 考察

- Field Lengthを、10,000と100で、負荷テストを実施した。
- 上記の表により、100で実施した結果が、10000で実施した結果よりも、総じて、良いと判断される。
    - [READ]
        - 平均待ち時間は、以下。
            - データの９５％での平均待ち時間：field length 100 / field length 10000 = 1617/4151=38.9%
            - データの９９％での平均待ち時間：field length 100 / field length 10000 = 2295/7555=30.4%
    - [UPDATE]
        - 平均待ち時間は、以下。
            - データの９５％での平均待ち時間：field length 100 / field length 10000 = 1155/2489=46.4%
            - データの９９％での平均待ち時間：field length 100 / field length 10000 = 4103/7183=57.1%

- 以上より、フィールド長は、10000 byteよりも、100 byteの方が総じて、性能が上がると考察できる。なお、Load時の [INSERT]でも同様である。

    [![Image from Gyazo](https://i.gyazo.com/42d5f3cb5296c142b8904b11f4be1acb.png)](https://gyazo.com/42d5f3cb5296c142b8904b11f4be1acb)

##### 演習内容

- 環境

    - Cluster名：ycsb

    - Cluster数：1

    - Test用keyspace : ycsb

    - Test用table : usertable (PRIMARY KEY (y_id), Column数 11)

        ```
        cqlsh:ycsb> Describe keyspace ycsb
        
        CREATE KEYSPACE ycsb WITH replication = {
          'class': 'SimpleStrategy',
          'replication_factor': '1'
        };
        
        USE ycsb;
        
        CREATE TABLE usertable (
          y_id text,
          field0 text,
          field1 text,
          field2 text,
          field3 text,
          field4 text,
          field5 text,
          field6 text,
          field7 text,
          field8 text,
          field9 text,
          PRIMARY KEY ((y_id))
        ) WITH
          bloom_filter_fp_chance=0.010000 AND
          caching='KEYS_ONLY' AND
          comment='' AND
          dclocal_read_repair_chance=0.100000 AND
          gc_grace_seconds=864000 AND
          index_interval=128 AND
          read_repair_chance=0.000000 AND
          replicate_on_write='true' AND
          populate_io_cache_on_flush='false' AND
          default_time_to_live=0 AND
          speculative_retry='99.0PERCENTILE' AND
          memtable_flush_period_in_ms=0 AND
          compaction={'class': 'SizeTieredCompactionStrategy'} AND
          compression={'sstable_compression': 'LZ4Compressor'};
        ```

- 操作

    - ① ycsb（負荷ツール）で、workload-Aのデータで負荷をかける。

        - データを準備

        ```bash
        $ cd YCSB
        $ bin/ycsb load cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
        ```
    ```
    
    - 負荷ツールを実行し、実行結果を取得
    
    ​```bash
        bin/ycsb run cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
    ```
    
    - ②負荷設定ファイルを変更して、再度、
    
        ```
        # fieldlength を100にして保存
        $ bin/ycsb load cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
        $ bin/ycsb run cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
        ```
        
        
    

##### 結果

- 操作①

    - load

    ```
mitsuru@DESKTOP-SFCGKJL:~/YCSB$ bin/ycsb load cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
    [WARN]  Running against a source checkout. In order to get our runtime dependencies we'll have to invoke Maven. Depending on the state of your system, this may take ~30-45 seconds
[DEBUG]  Running 'mvn -pl com.yahoo.ycsb:cassandra-binding -am package -DskipTests dependency:build-classpath -DincludeScope=compile -Dmdep.outputFilterFile=true'
    java -cp /home/mitsuru/YCSB/cassandra/conf:/home/mitsuru/YCSB/cassandra/target/cassandra-binding-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/apache/htrace/htrace-core4/4.1.0-incubating/htrace-core4-4.1.0-incubating.jar:/home/mitsuru/.m2/repository/org/hdrhistogram/HdrHistogram/2.1.4/HdrHistogram-2.1.4.jar:/home/mitsuru/.m2/repository/io/dropwizard/metrics/metrics-core/3.1.2/metrics-core-3.1.2.jar:/home/mitsuru/.m2/repository/com/datastax/cassandra/cassandra-driver-core/3.0.0/cassandra-driver-core-3.0.0.jar:/home/mitsuru/.m2/repository/io/netty/netty-handler/4.0.33.Final/netty-handler-4.0.33.Final.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-core-asl/1.9.4/jackson-core-asl-1.9.4.jar:/home/mitsuru/.m2/repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/home/mitsuru/.m2/repository/io/netty/netty-transport/4.0.33.Final/netty-transport-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-codec/4.0.33.Final/netty-codec-4.0.33.Final.jar:/home/mitsuru/.m2/repository/com/google/guava/guava/16.0.1/guava-16.0.1.jar:/home/mitsuru/YCSB/core/target/core-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-mapper-asl/1.9.4/jackson-mapper-asl-1.9.4.jar:/home/mitsuru/.m2/repository/io/netty/netty-buffer/4.0.33.Final/netty-buffer-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-common/4.0.33.Final/netty-common-4.0.33.Final.jar com.yahoo.ycsb.Client -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada -p hosts=127.0.0.1 -load
    Command line: -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada -p hosts=127.0.0.1 -load
    YCSB Client 0.16.0-SNAPSHOT
    
    Loading workload...
    Starting test.
    SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    SLF4J: Defaulting to no-operation (NOP) logger implementation
    SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
    DBWrapper: report latency for each error is false and specific error codes to track for latency are: []'
    [OVERALL], RunTime(ms), 5485
    [OVERALL], Throughput(ops/sec), 182.31540565177758
    [TOTAL_GCS_PS_Scavenge], Count, 6
    [TOTAL_GC_TIME_PS_Scavenge], Time(ms), 81
    [TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 1.4767547857793983
    [TOTAL_GCS_PS_MarkSweep], Count, 0
    [TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0
    [TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
    [TOTAL_GCs], Count, 6
    [TOTAL_GC_TIME], Time(ms), 81
    [TOTAL_GC_TIME_%], Time(%), 1.4767547857793983
    [CLEANUP], Operations, 1
    [CLEANUP], AverageLatency(us), 2214912.0
    [CLEANUP], MinLatency(us), 2213888
    [CLEANUP], MaxLatency(us), 2215935
    [CLEANUP], 95thPercentileLatency(us), 2215935
    [CLEANUP], 99thPercentileLatency(us), 2215935
    [INSERT], Operations, 1000
    [INSERT], AverageLatency(us), 2449.511
    [INSERT], MinLatency(us), 1174
    [INSERT], MaxLatency(us), 55423
    [INSERT], 95thPercentileLatency(us), 3965
    [INSERT], 99thPercentileLatency(us), 15759
    ```
    
    - run
    
        ```
        mitsuru@DESKTOP-SFCGKJL:~/YCSB$ date
        Tue Jun 11 10:37:42 DST 2019
        mitsuru@DESKTOP-SFCGKJL:~/YCSB$ bin/ycsb run cassandra-cql -P workloads/workloada -p "hosts=127.0.0.1"
        [WARN]  Running against a source checkout. In order to get our runtime dependencies we'll have to invoke Maven. Depending on the state of your system, this may take ~30-45 seconds
        [DEBUG]  Running 'mvn -pl com.yahoo.ycsb:cassandra-binding -am package -DskipTests dependency:build-classpath -DincludeScope=compile -Dmdep.outputFilterFile=true'
        java -cp /home/mitsuru/YCSB/cassandra/conf:/home/mitsuru/YCSB/cassandra/target/cassandra-binding-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/apache/htrace/htrace-core4/4.1.0-incubating/htrace-core4-4.1.0-incubating.jar:/home/mitsuru/.m2/repository/org/hdrhistogram/HdrHistogram/2.1.4/HdrHistogram-2.1.4.jar:/home/mitsuru/.m2/repository/io/dropwizard/metrics/metrics-core/3.1.2/metrics-core-3.1.2.jar:/home/mitsuru/.m2/repository/com/datastax/cassandra/cassandra-driver-core/3.0.0/cassandra-driver-core-3.0.0.jar:/home/mitsuru/.m2/repository/io/netty/netty-handler/4.0.33.Final/netty-handler-4.0.33.Final.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-core-asl/1.9.4/jackson-core-asl-1.9.4.jar:/home/mitsuru/.m2/repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/home/mitsuru/.m2/repository/io/netty/netty-transport/4.0.33.Final/netty-transport-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-codec/4.0.33.Final/netty-codec-4.0.33.Final.jar:/home/mitsuru/.m2/repository/com/google/guava/guava/16.0.1/guava-16.0.1.jar:/home/mitsuru/YCSB/core/target/core-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-mapper-asl/1.9.4/jackson-mapper-asl-1.9.4.jar:/home/mitsuru/.m2/repository/io/netty/netty-buffer/4.0.33.Final/netty-buffer-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-common/4.0.33.Final/netty-common-4.0.33.Final.jar com.yahoo.ycsb.Client -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada -p hosts=127.0.0.1 -t
        Command line: -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada -p hosts=127.0.0.1 -t
        YCSB Client 0.16.0-SNAPSHOT
        
        Loading workload...
        Starting test.
        SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    SLF4J: Defaulting to no-operation (NOP) logger implementation
        SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
    DBWrapper: report latency for each error is false and specific error codes to track for latency are: []
        [OVERALL], RunTime(ms), 4880
    [OVERALL], Throughput(ops/sec), 204.91803278688525
        [TOTAL_GCS_PS_Scavenge], Count, 2
    [TOTAL_GC_TIME_PS_Scavenge], Time(ms), 29
        [TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.5942622950819672
        [TOTAL_GCS_PS_MarkSweep], Count, 0
        [TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0
        [TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
        [TOTAL_GCs], Count, 2
        [TOTAL_GC_TIME], Time(ms), 29
        [TOTAL_GC_TIME_%], Time(%), 0.5942622950819672
        [READ], Operations, 489
        [READ], AverageLatency(us), 2474.887525562372
        [READ], MinLatency(us), 1315
        [READ], MaxLatency(us), 29119
        [READ], 95thPercentileLatency(us), 4151
        [READ], 99thPercentileLatency(us), 7555
        [READ], Return=OK, 489
        [CLEANUP], Operations, 1
        [CLEANUP], AverageLatency(us), 2214912.0
        [CLEANUP], MinLatency(us), 2213888
        [CLEANUP], MaxLatency(us), 2215935
        [CLEANUP], 95thPercentileLatency(us), 2215935
        [CLEANUP], 99thPercentileLatency(us), 2215935
        [UPDATE], Operations, 511
        [UPDATE], AverageLatency(us), 1379.9021526418787
        [UPDATE], MinLatency(us), 483
        [UPDATE], MaxLatency(us), 20095
        [UPDATE], 95thPercentileLatency(us), 2489
        [UPDATE], 99thPercentileLatency(us), 7183
        [UPDATE], Return=OK, 511
        
        ```

- 操作②

    - load

        ```
        mitsuru@DESKTOP-SFCGKJL:~/YCSB$ bin/ycsb load cassandra-cql -P workloads/workloada2 -p "hosts=127.0.0.1"
        [WARN]  Running against a source checkout. In order to get our runtime dependencies we'll have to invoke Maven. Depending on the state of your system, this may take ~30-45 seconds
        [DEBUG]  Running 'mvn -pl com.yahoo.ycsb:cassandra-binding -am package -DskipTests dependency:build-classpath -DincludeScope=compile -Dmdep.outputFilterFile=true'
        java -cp /home/mitsuru/YCSB/cassandra/conf:/home/mitsuru/YCSB/cassandra/target/cassandra-binding-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/apache/htrace/htrace-core4/4.1.0-incubating/htrace-core4-4.1.0-incubating.jar:/home/mitsuru/.m2/repository/org/hdrhistogram/HdrHistogram/2.1.4/HdrHistogram-2.1.4.jar:/home/mitsuru/.m2/repository/io/dropwizard/metrics/metrics-core/3.1.2/metrics-core-3.1.2.jar:/home/mitsuru/.m2/repository/com/datastax/cassandra/cassandra-driver-core/3.0.0/cassandra-driver-core-3.0.0.jar:/home/mitsuru/.m2/repository/io/netty/netty-handler/4.0.33.Final/netty-handler-4.0.33.Final.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-core-asl/1.9.4/jackson-core-asl-1.9.4.jar:/home/mitsuru/.m2/repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/home/mitsuru/.m2/repository/io/netty/netty-transport/4.0.33.Final/netty-transport-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-codec/4.0.33.Final/netty-codec-4.0.33.Final.jar:/home/mitsuru/.m2/repository/com/google/guava/guava/16.0.1/guava-16.0.1.jar:/home/mitsuru/YCSB/core/target/core-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-mapper-asl/1.9.4/jackson-mapper-asl-1.9.4.jar:/home/mitsuru/.m2/repository/io/netty/netty-buffer/4.0.33.Final/netty-buffer-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-common/4.0.33.Final/netty-common-4.0.33.Final.jar com.yahoo.ycsb.Client -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada2 -p hosts=127.0.0.1 -load
        Command line: -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada2 -p hosts=127.0.0.1 -load
        YCSB Client 0.16.0-SNAPSHOT
        
        Loading workload...
        Starting test.
        SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
        SLF4J: Defaulting to no-operation (NOP) logger implementation
        SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
        DBWrapper: report latency for each error is false and specific error codes to track for latency are: []'
        
        [OVERALL], RunTime(ms), 4070
        [OVERALL], Throughput(ops/sec), 245.7002457002457
        [TOTAL_GCS_PS_Scavenge], Count, 1
        [TOTAL_GC_TIME_PS_Scavenge], Time(ms), 18
        [TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.4422604422604423
        [TOTAL_GCS_PS_MarkSweep], Count, 0
        [TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0
        [TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
        [TOTAL_GCs], Count, 1
        [TOTAL_GC_TIME], Time(ms), 18
        [TOTAL_GC_TIME_%], Time(%), 0.4422604422604423
        [CLEANUP], Operations, 1
        [CLEANUP], AverageLatency(us), 2214912.0
        [CLEANUP], MinLatency(us), 2213888
        [CLEANUP], MaxLatency(us), 2215935
        [CLEANUP], 95thPercentileLatency(us), 2215935
        [CLEANUP], 99thPercentileLatency(us), 2215935
        [INSERT], Operations, 1000
        [INSERT], AverageLatency(us), 1068.287
        [INSERT], MinLatency(us), 376
        [INSERT], MaxLatency(us), 25071
        [INSERT], 95thPercentileLatency(us), 1880
        [INSERT], 99thPercentileLatency(us), 2637
        [INSERT], Return=OK, 1000
        
        ```

    - run

        ```
        mitsuru@DESKTOP-SFCGKJL:~/YCSB$ bin/ycsb run cassandra-cql -P workloads/workloada2 -p "hosts=127.0.0.1"
        [WARN]  Running against a source checkout. In order to get our runtime dependencies we'll have to invoke Maven. Depending on the state of your system, this may take ~30-45 seconds
        [DEBUG]  Running 'mvn -pl com.yahoo.ycsb:cassandra-binding -am package -DskipTests dependency:build-classpath -DincludeScope=compile -Dmdep.outputFilterFile=true'
        java -cp /home/mitsuru/YCSB/cassandra/conf:/home/mitsuru/YCSB/cassandra/target/cassandra-binding-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/apache/htrace/htrace-core4/4.1.0-incubating/htrace-core4-4.1.0-incubating.jar:/home/mitsuru/.m2/repository/org/hdrhistogram/HdrHistogram/2.1.4/HdrHistogram-2.1.4.jar:/home/mitsuru/.m2/repository/io/dropwizard/metrics/metrics-core/3.1.2/metrics-core-3.1.2.jar:/home/mitsuru/.m2/repository/com/datastax/cassandra/cassandra-driver-core/3.0.0/cassandra-driver-core-3.0.0.jar:/home/mitsuru/.m2/repository/io/netty/netty-handler/4.0.33.Final/netty-handler-4.0.33.Final.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-core-asl/1.9.4/jackson-core-asl-1.9.4.jar:/home/mitsuru/.m2/repository/org/slf4j/slf4j-api/1.7.25/slf4j-api-1.7.25.jar:/home/mitsuru/.m2/repository/io/netty/netty-transport/4.0.33.Final/netty-transport-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-codec/4.0.33.Final/netty-codec-4.0.33.Final.jar:/home/mitsuru/.m2/repository/com/google/guava/guava/16.0.1/guava-16.0.1.jar:/home/mitsuru/YCSB/core/target/core-0.16.0-SNAPSHOT.jar:/home/mitsuru/.m2/repository/org/codehaus/jackson/jackson-mapper-asl/1.9.4/jackson-mapper-asl-1.9.4.jar:/home/mitsuru/.m2/repository/io/netty/netty-buffer/4.0.33.Final/netty-buffer-4.0.33.Final.jar:/home/mitsuru/.m2/repository/io/netty/netty-common/4.0.33.Final/netty-common-4.0.33.Final.jar com.yahoo.ycsb.Client -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada2 -p hosts=127.0.0.1 -t
        Command line: -db com.yahoo.ycsb.db.CassandraCQLClient -P workloads/workloada2 -p hosts=127.0.0.1 -t
        YCSB Client 0.16.0-SNAPSHOT
        
        Loading workload...
        Starting test.
        SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
        SLF4J: Defaulting to no-operation (NOP) logger implementation
        SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
        DBWrapper: report latency for each error is false and specific error codes to track for latency are: []'
        [OVERALL], RunTime(ms), 4104
        [OVERALL], Throughput(ops/sec), 243.66471734892787
        [TOTAL_GCS_PS_Scavenge], Count, 1
        [TOTAL_GC_TIME_PS_Scavenge], Time(ms), 22
        [TOTAL_GC_TIME_%_PS_Scavenge], Time(%), 0.5360623781676412
        [TOTAL_GCS_PS_MarkSweep], Count, 0
        [TOTAL_GC_TIME_PS_MarkSweep], Time(ms), 0
        [TOTAL_GC_TIME_%_PS_MarkSweep], Time(%), 0.0
        [TOTAL_GCs], Count, 1
        [TOTAL_GC_TIME], Time(ms), 22
        [TOTAL_GC_TIME_%], Time(%), 0.5360623781676412
        [READ], Operations, 497
        [READ], AverageLatency(us), 1351.20523138833
        [READ], MinLatency(us), 759
        [READ], MaxLatency(us), 25503
        [READ], 95thPercentileLatency(us), 1617
        [READ], 99thPercentileLatency(us), 2295
        [READ], Return=OK, 497
        [CLEANUP], Operations, 1
        [CLEANUP], AverageLatency(us), 2239488.0
        [CLEANUP], MinLatency(us), 2238464
        [CLEANUP], MaxLatency(us), 2240511
        [CLEANUP], 95thPercentileLatency(us), 2240511
        [CLEANUP], 99thPercentileLatency(us), 2240511
        [UPDATE], Operations, 503
        [UPDATE], AverageLatency(us), 905.3339960238569
        [UPDATE], MinLatency(us), 391
        [UPDATE], MaxLatency(us), 7167
        [UPDATE], 95thPercentileLatency(us), 1155
        [UPDATE], 99thPercentileLatency(us), 4103
        [UPDATE], Return=OK, 503
        
        ```

        