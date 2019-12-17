# rule-backfill

A practice-purposed cli tool to do retroactive recording rule evaluation for Prometheus.

Aims to fix issue https://github.com/prometheus/prometheus/issues/11.

## Build

`
GO111MODULE=on go build -o rule-backfill main.go
`

## How to use

```
➜  rule-backfill -h
usage: rule-backfill [<flags>] <rule-file> [<db path>] [<dest path>]

Tooling for backfilling Prometheus Recording Rules.

Flags:
  -h, --help     Show context-sensitive help (also try --help-long and --help-man).
      --version  Show application version.

Args:
  <rule-file>    The rule file to do backfilling.
  [<db path>]    database path (default is data)
  [<dest path>]  path to generate new block
```

## Tutorial

1. Start Prometheus in the local environment


2. Use tsdbcli to check the metrics in tsdb dir `data/`. There is no metric name containing `test`.

```
./tsdbcli dump data | grep test
```

3. Do backfilling based on rule file `example.yaml`. 

The first `data` arg specifies the tsdb dir to query the past data. 

The second `data` arg specifies the dir to generate the new block.

```
./rule-backfill example_rule.yaml data data
level=info msg="replaying WAL, this may take awhile"
level=info msg="WAL segment loaded" segment=0 maxSegment=1
level=info msg="WAL segment loaded" segment=1 maxSegment=1
level=info msg="write block" mint=1576563064320 maxt=1576563859000 ulid=01DW98EQVKD55FCJ0QJV2FTT0P duration=770.543238ms
blockId=data/01DW98EQVKD55FCJ0QJV2FTT0P
```

4. Check the metrics in tsdb dir again.

```
./tsdbcli dump data | grep test | head
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563069000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563074000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563079000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563084000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563089000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563094000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563099000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563104000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563109000
{__name__="test",instance="localhost:9090",job="prometheus",key="value"} 2 1576563114000
```

5. Currently the tool cannot load the block into Prometheus directly. So simply restart the Prometheus and check the UI.

![alt text](exp.png)