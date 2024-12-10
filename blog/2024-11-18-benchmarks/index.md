---
slug: load-tests
title: DinoDNS Performance Benchmarks
authors: [jafayer]
tags: [updates, benchmarks]
---

As DinoDNS nears a stable release, we wanted to ground its performance against an established player in the DNS server space to give prospective users an idea of what it might offer. To that end, we have benchmarked DinoDNS against [CoreDNS](https://coredns.io) using an experimental methodology that aims to evaluate each server's performance in raw query throughput.

<!-- truncate -->

## TL;DR

In singlethreaded mode throughout the course of these synthetic benchmarks, DinoDNS produces significant performance gains over CoreDNS in terms of reduced latency and increased query throughput. These gains are sustained, though diminished, with the number of vCPUs/replicas allocated.

In multithreaded workloads, CoreDNS beats DinoDNS by a consistent measure (though, again, diminishing as the server spreads across more vCPUs). This implies a significant performance overhead to running DinoDNS in cluster mode compared to CoreDNS's ability to leverage more available goroutines.

![Average latency summary](./Average%20Latency%20(ms).png)
![Queries per Second summary](./Queries%20per%20Second.png)


## Methodology

The methodology we're using is largely inspired by [this blog post](https://web.archive.org/web/20240214002936/https://northflank.com/blog/performance-testing-for-core-dns) that previously has benchmarked CoreDNS's performance. We kept our benchmarks as faithful to the ones laid out here as we could, and reimplemented the benchmarks both for CoreDNS and DinoDNS to provide a direct, head-to-head comparison.

<details>
    <summary>A note on implementation differences</summary>
    
    The linked post evaluates CoreDNS as a Kubernetes cluster DNS resolver. Because of that, the zone data being evaluated is part domains and part Kubernetes service hostnames. It is unclear whether an internal Kubernetes instance of CoreDNS would behave differently (because of networking, optimizations, or proximity to internal Kubernetes systems) than it would as an authoritative nameserver.

    To account for this, we've deviated from the original implementation to host both CoreDNS and DinoDNS as authoritative nameservers with 100 domain names, using the `template` plugin in CoreDNS's Corefile.
</details>

In brief, the experiment is as follows:

We relied on `dnsperf` to generate the DNS requests targeting each of the servers. We provisioned a Kubernetes cluster on Google Kubernetes Engine with a dedicated node for each component involved (a CoreDNS node, a DinoDNS node, and a dnsperf node, in addition to a default node). Each node other than the default node was provisioned on an `n2-standard-8` machine, which has 8 vCPUs and 32GB memory.

We ran dnsperf with generally high allowances (i.e. setting the max queries per second to 1 million) using the same settings as the Northflank benchmark to give dnsperf enough headroom to hit the servers until they maxed out of available CPU. In all tests, the server hit or approached its CPU allocation before dnsperf did, indicating good resource exhaustion during the tests.

Each test ran for 10 minutes to provide enough time to sustain the load. Only one test was run at a time to ensure there were no network or CPU interactions that may have affected the results.

In all, a matrix of three CPU tiers and two configuration modes was run to evaluate DinoDNS and CoreDNS: 1 vCPU, 2vCPU, and 4 vCPU. At each CPU tier, two tests were run for each server: one with a single instance granted the full CPU limit available (and DinoDNS running in multithreaded mode), and another with each server being given only 1 vCPU and with the Kubernetes replica count scaled to the max number of CPUs available (with DinoDNS running in singlethreaded mode). A Kubernetes service automatically load balanced between the servers to distribute the traffic.

Each server was given the same A records (a simple `127.0.0.1` response) for 100 programmatically generated domain names (simple variants of `example{X}.com`), where X was all odd numbers between 1 and 200, resulting in 100 zones. DNSPerf was given all zones between 1 and 200, resulting in 50% NOERROR queries and 50% NXDOMAIN/REFUSED queries.

DinoDNS was run using the default, in-memory storage plugin to host zone data for the 100 zones, while CoreDNS was configured with a simple static Corefile consisting of the same 100 generated zones with `template` plugin directives.

## A note on vCPU allocation

There's a big caveat to DinoDNS's performance that may help explain why its singlethreaded performance was dramatically different than its multithreaded performance even when capped to 1 vCPU. Kubernetes appears to allow pods access to all hardware cores and simply limits the vCPU utilization as a fraction of available compute. All tests were run on an 8-core CPU, meaning that even when capped to 1 vCPU when DinoDNS was running in multithreaded mode, it attempted to make use of multiple cores despite only being able to route traffic with a relatively low CPU bandwidth. This means it effectively incurs all the overhead of running in cluster mode and running the load balancer against the array of server instances, while only being able to make use of a fraction of the available compute.

This is why we ran the two additional tests at every CPU tier: to test the impact of horizontally scaling DinoDNS on the pod level against allowing Node.js's cluster mode to scale within the container.

Finally, it should be noted that these horizontally-scaled tests deviated from the methodology used for the multithreaded tests in one other way: in order to achieve close to uniform resource exhaustion among the pods, we had to modify the dnsperf configuration to set `MAX_CLIENTS` and `MAX_THREADS` (they was previously unset during the other tests, but this was causing socket reuse from dnsperf which meant only one of the pods actually received the traffic), and we had to increase dnsperf's resource allocation to 7 CPUs (unlike the other tests which were run with all components scaled across-the-board to the same vCPU allocation) to achieve resource exhaustion.

Despite this, neither CoreDNS nor DinoDNS achieved perfectly uniform resource exhaustion during the horizontally scaled tests, though for most of the test all four pods were in the high 900-mCPUs, which indicates _near_ resource exhaustion.

## Results

Below are the dnsperf results for each CPU tier.

### 1 CPU

|                                   | Queries Sent | Queries Completed | Queries Lost | QPS             | Average Latency (s) | Success Rate (%) | Failure Rate (%) |
|-----------------------------------|--------------|-------------------|--------------|-----------------|---------------------|------------------|------------------|
| CoreDNS                           | 12,514,663   | 12,514,663        | 0            | 20,855          | 0.004768            | 100.00%          | 0.00%            |
| CoreDNS (1 CPU, 1x pods)          | 12,514,663   | 12,514,663        | 0            | 20,855 (--)     | 0.004768 (--)       | 100.00%          | 0.00%            |
| DinoDNS                           | 11,151,481   | 11,151,481        | 0            | 18,584 (-11.52%)| 0.005314 (+10.83%)  | 100.00%          | 0.00%            |
| DinoDNS (singlethreaded, 1x pods) | 23,803,873   | 23,803,673        | 200          | 39,672 (62.20+%)| 0.002466 (-63.64%)  | 100.00%          | 0.00%            |

*Note: for this test, "CoreDNS" and "CoreDNS (1 CPU, 1x pods) are the same reused test result, since they effectively mean the same thing &mdash; coredns with 1 replica and 1vCPU*.

In multithreaded mode, DinoDNS was acceptably performant in terms of raw QPS and average latency, but significantly less so than CoreDNS. However, in singlethreaded mode, DinoDNS handily beats both CoreDNS and its own multithreaded performance. This implies a large overhead for running the server in cluster mode, but streamlined and efficient performance in single-threaded.

This singlethreaded result was run three times to confirm the results.

### 2 CPU

|                                   | Queries Sent | Queries Completed | Queries Lost | QPS             | Average Latency (s) | Success Rate (%) | Failure Rate (%) |
|-----------------------------------|--------------|-------------------|--------------|-----------------|---------------------|------------------|------------------|
| CoreDNS                           | 24,923,203   | 24,923,003        | 200          | 41,536          | 0.002341            | 100.00%          | 0.00%            |
| CoreDNS (1 CPU, 2x pods)          | 22,625,858   | 22,625,658        | 200          | 37,709 (-9.66%) | 0.002534 (+7.92%)   | 100.00%          | 0.00%            |
| DinoDNS                           | 22,631,427   | 22,631,427        | 0            | 37,716 (-9.64%) | 0.002562 (+9.01%)   | 100.00%          | 0.00%            |
| DinoDNS (singlethreaded, 2x pods) | 34,432,859   | 34,432,659        | 200          | 57,387 (+32.05%)| 0.001674 (-33.23%)  | 100.00%          | 0.00%            |

With 2 CPUs, CoreDNS's advantage seems to hold steady around ~10% performance increase.

At the same time, DinoDNS's horizontally-scaled singlethreaded environment holds an advantage, though significantly less than CoreDNS which now has double the CPU available.

At 2vCPU, horizontally scaling a low-spec instance does not seem an effective way to run CoreDNS, and it nets a 10% performance drop, nearly exactly in line with DinoDNS's multithreaded performance.

### 4 CPU

|                                   | Queries Sent | Queries Completed | Queries Lost | QPS             | Average Latency (s) | Success Rate (%) | Failure Rate (%) |
|-----------------------------------|--------------|-------------------|--------------|-----------------|---------------------|------------------|------------------|
| CoreDNS                           | 49,132,583   | 49,132,383        | 200          | 81,887          | 0.001171            | 100.00%          | 0.00%            |
| CoreDNS (1 CPU, 2x pods)          | 39,681,172   | 39,680,972        | 200          | 66,132 (-21.29%)| 0.001419 (+19.15%)  | 100.00%          | 0.00%            |
| DinoDNS                           | 45,105,368   | 45,105,368        | 0            | 75,172 (-8.55%) | 0.001223 (+4.34%)   | 100.00%          | 0.00%            |
| DinoDNS (singlethreaded, 2x pods) | 59,685,585   | 59,685,452        | 133          | 99,475 (+19.40%)| 0.000941 (-21.78%)  | 100.00%          | 0.00%            |

With 4 CPUs, CoreDNS's advantage begins to shrink slightly but significantly.

## Conclusions

We set out in the course of this test to establish whether DinoDNS was, at minimum, a viable option when it comes to the demands of production workloads on a DNS server. Overall, we're quite happy with these results &mdash; at minimum, they demonstrate competitive performance against an established player in the DNS server space. Broadly speaking, DinoDNS's default, in-memory storage surpassed the performance when in singledthreaded mode of CoreDNS's performance by a significant measure. These differences diminished as CPU allocation increased. DinoDNS's multithreaded mode, while useful for workloads and deployment environments that cannot easily horizontally scale, incurs a steep performance penalty, though this penalty also decreases slightly as the available vCPU increases (see [the vCPU utilization section for more info](#a-note-on-vcpu-allocation) for more information).

We believe the ease of authoring bespoke DNS servers with DinoDNS represents a substantial advantage for the DinoDNS framework and ecosystem, and enables developers from a diverse background, even Express-familiar web developers who have no experience working with DNS systems, to more easily be able to reason about how to build DNS servers and to understand their behavior. At the same time, in no way are we attempting to claim that this synthetic benchmark demonstrates that DinoDNS is "better than" CoreDNS. There are real tradeoffs in terms of stability, maturity, the robustness of the plugin ecosystem, and indeed the heft of the runtime when considering the Golang-based CoreDNS to DinoDNS.

Still, it is a little remarkable to us that a Node.js-based DNS server put up such competitive numbers, and we look forward to continuing development to make DinoDNS a reasonable choice for organizations and teams that want to lower the barrier to entry to authoring and administering DNS servers.

-------

### Some additional notes/caveats

In general, our CoreDNS tests were significantly different than the benchmarks we based our methodology on &mdash; on average across all CPU tiers, for example, this benchmark of CoreDNS saw a 27% increase in query throughput. This may be explainable by different versions of CoreDNS used, the performance of the `template` plugin as opposed to whichever storage method was used in the reference benchmark, different deployment configurations (with CoreDNS embedded into K8s for service discovery), etc.

Because of this experimental setup, these results also come with an important caveat: not very often do DNS servers simply answer pre-configured, static records that are bundled in the configuration. For a more realistic production workload, another CoreDNS plugin would have to be chosen, such as the internal [auto plugin](https://coredns.io/plugins/auto/), and likewise an analogous database or file-based plugin would need to be tested in DinoDNS. Still, we aimed throughout the course of this benchmark to establish an upper bound of query performance, not to establish real-world performance, so additional testing will be needed to make commentary on that.