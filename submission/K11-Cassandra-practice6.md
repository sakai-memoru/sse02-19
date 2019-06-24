# K11-Cassandra-practice6

番号: sse02-19 日付：19/06/12

## 課題

演習マニュアルを元に以下の検証を行い、結果を元に説明をしてください。
1. 演習6の結果をもとに、データの断片化が各種メトリクス・サーバリソースにどういった影響を与えるかと、その理由を考察して説明ください

### 1.  演習6の結果をもとに考察する

##### データの断片化が各種メトリクス・サーバリソースにどういった影響を与えるかと、その理由について

##### 取得したデータ

- 演習時の環境と、取得したデータは、「演習内容」に記載した。
- 取得したデータについて
- 操作①のOffset No : 1057 (08:45:21時点)
    - SSTables per Read  : 1
    - Write Latency : 0.06～0.103 msec  (異常値4.768 msecが１回ある)
    - Read Latency : 182.785 msec
- 操作①のOffset No : 1057（09:03:53時点）
        - SSTables per Read  : 7
    - Write Latency : 0.149～0.301 msec  (異常値6.866 msecが１回ある)
        - Read Latency : 152.321～545.791 msec
- 操作①のOffset No : 1786（09:06:17時点）
        - SSTables per Read  : 26
    - Write Latency : 0.124～0.258 msec  (異常値5.722 msecが１回ある)
        - Read Latency : 182.785～654.949 msec
    - 操作②のOffset No: 5（09:44:14時点）
        - SSTables per Read  : 75 or 3
        - Write Latency : 0.072～0.446 msec
        - Read Latency : 0.60～3.973 msec
    - 操作②のOffset No: 39（09:45:11時点）
        - SSTables per Read  : 5, 12,15, 19, 11
        - Write Latency : 0.179～0.310 msec  (異常値4.768 msecが１回ある)
        - Read Latency : 0.535～11.864 msec
    - 操作②のOffset No: 86（09:46:30時点）
    - SSTables per Read  : 10,25,31,37,43
        - Write Latency : 0.149～0.372 msec  (異常値8.239 msec, 11.864msecが１回づつある)
    - Read Latency : 0.642～29.521 msec
    - 操作②のOffset No: 600（10:00:48時点）
        - SSTables per Read  : 17
        - Write Latency : 0.124～0.372 msec  (異常値8.239 msecが１回ある)
        - Read Latency : 8.239～105.778 msec

##### 考察

- アクセスするSSTable数について
    - 最初の1000回の更新、読み込み時は、以下のように変遷。
        - １→7 →26
        - 書き込んでいるデータは、{‘abc’, ‘name1’という同一のPrimary Keyのカラムデータであるが、}断片化が進んでいる。断片化が進むことで、アクセスするSSTable数も増えている。
    - Compact実行後に、すぐ、次の1000回の更新、読み込みを実行したため、以下のように変遷。
        - 3, 75 → 5, 12,15, 19, 11→10,25,31,37,43→17
        - 最初の1000回の更新後に、26まで増えていたSSTableへの平均アクセス数であるが、減っている。ただ、Compactionが実施されている最中のためか、一時期、アクセス数が増える事象も発生した。
- Write Latencyについて
    - 基本的に、全取得データについて、0.06msecから0.446msecの間で処理が完了している。性能の劣化は見受けられない。
    - なお、時々、待ち時間が、4.768msec～11.864secであることが７回発生。
- Read Latencyについて
    - 断片化が進んだ時点では、152.321～654.949 msecの、待ち時間が発生していた。が、Compaction直後は、0.535～29.521 msecにアクセス待ち時間が改善された。

##### 演習内容

- 環境

    - Cluster名：test 

    - Cluster数：3

    - Test用keyspace : anti

    - Test用table : test (PRIMARY KEY (id), Column数 2)

        ```
        cqlsh:anti> DESCRIBE keyspace anti
        CREATE KEYSPACE anti WITH replication = {
          'class': 'SimpleStrategy',
          'replication_factor': '3'
        };
        USE anti;
        CREATE TABLE test (
          id text,
          name text,
          PRIMARY KEY ((id))
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
          compaction={'min_threshold': '9999', 'class': 'SizeTieredCompactionStrategy', 'max_threshold': '9999'} AND
          compression={'sstable_compression': 'LZ4Compressor'};
        ```

- 操作

    - ① 対象のスキーマに以下を同時に実行し、その性能統計値を取得する。

        - データを書き込む（MEMTABLEに保持）、それをFlushすることを繰り返す。

        ```bash
        for i in {1..1000}; do ccm node1 cqlsh -f insert.cql && ccm node1 nodetool flush; done
        ```

        

        - データを読み込むことを繰り返す。

        ```bash
        for i in {1..1000}; do ccm node1 cqlsh -f select.cql; done
        ```
        
        - 統計量取得は、以下のコマンドを発行。
        
            ```
            $ ccm node1 nodetool cfhistograms anti test
            ```
    
    - ② 操作①で、断片化されたSSTableを、圧縮して、再度、どうような処理を行い、統計量を取得する。
    
- 統計量として、取得できるのは、以下の情報となる。

    - SSTables per Read　：読み込み時にアクセスするSSTableの数
    - Write Latency (microseconds) ：書き込み待ち時間 (μsec)
    - Read Latency (microseconds) ：読み込み待ち時間 (μsec)
    - Partition Size (bytes)：パーティションサイズ (byte)
    - Cell Count per Partition  : パーティション毎のカラム数

- 結果

    - 操作①のI/Oの統計量

        - 開始当初のデータ

            - Offset no : 1057

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 08:45:21 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                1109 sstables: 1
                
                Write Latency (microseconds)
                  60 us: 1
                  72 us: 2
                  86 us: 1
                 103 us: 2
                 124 us: 0
                 149 us: 0
                 179 us: 0
                 215 us: 0
                 258 us: 0
                 310 us: 0
                 372 us: 0
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 0
                 924 us: 0
                1109 us: 0
                1331 us: 0
                1597 us: 0
                1916 us: 0
                2299 us: 0
                2759 us: 0
                3311 us: 0
                3973 us: 0
                4768 us: 1
                
                Read Latency (microseconds)
                182785 us: 1
                
                Partition Size (bytes)
                72 bytes: 1057
                
                Cell Count per Partition
                2 cells: 1057
                ```

                

        - 断片化が進んだ時点で取得した２つのデータ

            - Offset no : 1706

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:03:53 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                1331 sstables: 7
                
                Write Latency (microseconds)
                 149 us: 2
                 179 us: 4
                 215 us: 9
                 258 us: 4
                 310 us: 3
                 372 us: 0
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 0
                 924 us: 0
                1109 us: 0
                1331 us: 0
                1597 us: 0
                1916 us: 0
                2299 us: 0
                2759 us: 0
                3311 us: 0
                3973 us: 0
                4768 us: 0
                5722 us: 0
                6866 us: 1
                
                Read Latency (microseconds)
                152321 us: 1
                182785 us: 0
                219342 us: 0
                263210 us: 0
                315852 us: 1
                379022 us: 0
                454826 us: 0
                545791 us: 5
                
                Partition Size (bytes)
                72 bytes: 1706
                
                Cell Count per Partition
                2 cells: 1706
                ```

                

            - Offset no : 1786

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:06:17 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                1331 sstables: 26
                
                Write Latency (microseconds)
                 124 us: 1
                 149 us: 5
                 179 us: 11
                 215 us: 26
                 258 us: 12
                 310 us: 6
                 372 us: 0
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 0
                 924 us: 0
                1109 us: 0
                1331 us: 0
                1597 us: 0
                1916 us: 0
                2299 us: 0
                2759 us: 0
                3311 us: 0
                3973 us: 0
                4768 us: 0
                5722 us: 1
                
                Read Latency (microseconds)
                182785 us: 2
                219342 us: 1
                263210 us: 0
                315852 us: 0
                379022 us: 2
                454826 us: 0
                545791 us: 17
                654949 us: 4
                
                Partition Size (bytes)
                72 bytes: 1786
                
                Cell Count per Partition
                2 cells: 1786
                ```

    - 操作②I/Oの統計量 (compact 実行後)

        - 開始当初のデータ

            - Offset No : 5

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:44:14 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                1 sstables: 75
                2 sstables: 3
                3 sstables: 3
                4 sstables: 3
                5 sstables: 3
                
                Write Latency (microseconds)
                 72 us: 1
                 86 us: 0
                103 us: 0
                124 us: 0
                149 us: 0
                179 us: 0
                215 us: 0
                258 us: 2
                310 us: 2
                372 us: 0
                446 us: 1
                
                Read Latency (microseconds)
                  60 us: 1
                  72 us: 15
                  86 us: 27
                 103 us: 12
                 124 us: 10
                 149 us: 3
                 179 us: 3
                 215 us: 1
                 258 us: 0
                 310 us: 0
                 372 us: 1
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 1
                 924 us: 0
                1109 us: 0
                1331 us: 3
                1597 us: 3
                1916 us: 1
                2299 us: 1
                2759 us: 1
                3311 us: 2
                3973 us: 2
                
                Partition Size (bytes)
                72 bytes: 5
                
                Cell Count per Partition
                2 cells: 5
                
                ```

                

            - Offset No : 39

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:45:11 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                20 sstables: 5
                24 sstables: 12
                29 sstables: 15
                35 sstables: 19
                42 sstables: 11
                
                Write Latency (microseconds)
                 179 us: 1
                 215 us: 4
                 258 us: 10
                 310 us: 4
                 372 us: 0
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 0
                 924 us: 0
                1109 us: 0
                1331 us: 0
                1597 us: 0
                1916 us: 0
                2299 us: 0
                2759 us: 0
                3311 us: 0
                3973 us: 0
                4768 us: 1
                
                Read Latency (microseconds)
                  535 us: 1
                  642 us: 1
                  770 us: 0
                  924 us: 0
                 1109 us: 0
                 1331 us: 0
                 1597 us: 0
                 1916 us: 0
                 2299 us: 0
                 2759 us: 1
                 3311 us: 11
                 3973 us: 11
                 4768 us: 21
                 5722 us: 7
                 6866 us: 3
                 8239 us: 0
                 9887 us: 4
                11864 us: 2
                
                Partition Size (bytes)
                72 bytes: 39
                
                Cell Count per Partition
                2 cells: 39
                ```

                

            - Offset No:86

                ```
                ​```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:46:30 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                42 sstables: 10
                50 sstables: 25
                60 sstables: 31
                72 sstables: 37
                86 sstables: 43
                
                Write Latency (microseconds)
                  149 us: 1
                  179 us: 2
                  215 us: 14
                  258 us: 18
                  310 us: 8
                  372 us: 2
                  446 us: 0
                  535 us: 0
                  642 us: 0
                  770 us: 0
                  924 us: 0
                 1109 us: 0
                 1331 us: 0
                 1597 us: 0
                 1916 us: 0
                 2299 us: 0
                 2759 us: 0
                 3311 us: 0
                 3973 us: 0
                 4768 us: 0
                 5722 us: 0
                 6866 us: 0
                 8239 us: 1
                 9887 us: 0
                11864 us: 1
                
                Read Latency (microseconds)
                  642 us: 4
                  770 us: 7
                  924 us: 1
                 1109 us: 2
                 1331 us: 1
                 1597 us: 0
                 1916 us: 0
                 2299 us: 0
                 2759 us: 0
                 3311 us: 0
                 3973 us: 0
                 4768 us: 1
                 5722 us: 20
                 6866 us: 30
                 8239 us: 31
                 9887 us: 16
                11864 us: 6
                14237 us: 8
                17084 us: 11
                20501 us: 6
                24601 us: 1
                29521 us: 1
                
                Partition Size (bytes)
                72 bytes: 86
                
                Cell Count per Partition
                2 cells: 86
                ```

                

            - Offset No :

                ```
                ​```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 09:46:30 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                42 sstables: 10
                50 sstables: 25
                60 sstables: 31
                72 sstables: 37
                86 sstables: 43
                
                Write Latency (microseconds)
                  149 us: 1
                  179 us: 2
                  215 us: 14
                  258 us: 18
                  310 us: 8
                  372 us: 2
                  446 us: 0
                  535 us: 0
                  642 us: 0
                  770 us: 0
                  924 us: 0
                 1109 us: 0
                 1331 us: 0
                 1597 us: 0
                 1916 us: 0
                 2299 us: 0
                 2759 us: 0
                 3311 us: 0
                 3973 us: 0
                 4768 us: 0
                 5722 us: 0
                 6866 us: 0
                 8239 us: 1
                 9887 us: 0
                11864 us: 1
                
                Read Latency (microseconds)
                  642 us: 4
                  770 us: 7
                  924 us: 1
                 1109 us: 2
                 1331 us: 1
                 1597 us: 0
                 1916 us: 0
                 2299 us: 0
                 2759 us: 0
                 3311 us: 0
                 3973 us: 0
                 4768 us: 1
                 5722 us: 20
                 6866 us: 30
                 8239 us: 31
                 9887 us: 16
                11864 us: 6
                14237 us: 8
                17084 us: 11
                20501 us: 6
                24601 us: 1
                29521 us: 1
                
                Partition Size (bytes)
                72 bytes: 86
                
                Cell Count per Partition
                2 cells: 86
                ```

                

        - １５分後

            - Offset No: 600

                ```
                mitsuru@DESKTOP-SFCGKJL:~$ date
                Tue Jun 11 10:00:48 DST 2019
                mitsuru@DESKTOP-SFCGKJL:~$ ccm node1 nodetool cfhistograms anti test
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/cluster_factory.py:22: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                /home/mitsuru/.local/lib/python2.7/site-packages/ccmlib/node.py:143: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
                  data = yaml.load(f)
                
                anti/test histograms
                
                SSTables per Read
                642 sstables: 17
                
                Write Latency (microseconds)
                 124 us: 1
                 149 us: 6
                 179 us: 12
                 215 us: 15
                 258 us: 12
                 310 us: 4
                 372 us: 2
                 446 us: 0
                 535 us: 0
                 642 us: 0
                 770 us: 0
                 924 us: 0
                1109 us: 0
                1331 us: 0
                1597 us: 0
                1916 us: 0
                2299 us: 0
                2759 us: 0
                3311 us: 0
                3973 us: 0
                4768 us: 0
                5722 us: 0
                6866 us: 0
                8239 us: 2
                
                Read Latency (microseconds)
                  8239 us: 1
                  9887 us: 0
                 11864 us: 0
                 14237 us: 0
                 17084 us: 0
                 20501 us: 0
                 24601 us: 0
                 29521 us: 0
                 35425 us: 0
                 42510 us: 0
                 51012 us: 0
                 61214 us: 0
                 73457 us: 1
                 88148 us: 9
                105778 us: 6
                
                Partition Size (bytes)
                72 bytes: 600
                
                Cell Count per Partition
                2 cells: 600
                ```

以上