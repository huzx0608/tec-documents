# Benchmark about NewSQL Database


## YardStick benchmark

Yardstick is a framework for writing benchmarks. Specifically it helps with writing benchmarks for clustered or otherwise distributed systems.

The framework comes with a default set of probes that collect various metrics during benchmark execution. Probes can be turned on or off in configuration. You can use a probe for measuring throughput and latency, or a probe that gathers vmstat statistics, etc... At the end of benchmark execution, Yardstick automatically produces files with probe points.

### How to Write benchmark




## Examples

- GridGain Benchmark: https://github.com/gridgain/yardstick-gridgain
- Yahoo Benchmark: https://github.com/Danny-Hazelcast/YCSB


## Ref

- [x] [GridGain Benchmark](https://ignite.apache.org/docs/latest/perf-and-troubleshooting/yardstick-benchmarking)
- [x] [YardStick Benchmark](https://github.com/gridgain/yardstick)

