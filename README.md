A LogStash Configuration to Publish Event Counts to Graphite
==========================================================

Graphite is used to display a stream of measurement/timestamp pairs.
This is not directly suitable for displaying events because
Graphite's minimum time box is 1 second.
This means that high frequency events exceeding this rate are lost.

To solve this problem statsd is often used as an intermediary between LogStash and Graphite.
statsd aggregates discrete events into a frequency or events per time period such as logins for each hour.

The LogStash metrics plugin offers a similar solution to this problem.

This is generally OK, but it means that event metrics published to Graphite 
have a timestamp corresponding to the aggregation time.
This is not useful if historical log files need to be ingested.

The LogStash configuration published here uses a different aggregation strategy
that allows historical logs and real time logs to be ingested.
The basic approach is to examine the timestamp of the incoming log record
and to increment a bin count defined by the surrounding whole hour.
Whenever a new record is seen that does not fit into the current bin a 
new bin is created around the timestamp of the new record.

Aggregated records passed to Graphite will have a timestamp
which corresponds to the end of the hour encompassing a group of records.
That is just after the timestamp of the last record in the bin.

The binning process is determined by transitions from one group of events
to the next group of events with grouping defined by proximity in time.
When testing the config file this may be confusing as the last group of log records
will not produce an output until a new group comes along.

To run LogStash from the command line:

```
./bin/logstash -f /Users/developer/git/logstash_config/configuration/login_stats_graphite3.conf
```
To test using the command line the .sincedb timestamp database needs to be reset:
```
rm ~/.sincedb_*; ./bin/logstash -f /Users/developer/git/logstash_config/configuration/login_stats_graphite3.conf
```
