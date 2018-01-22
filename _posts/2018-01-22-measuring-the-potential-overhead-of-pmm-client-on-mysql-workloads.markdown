---
layout: post
redirect_from:
  - 2018/01/22/measuring-the-potential-overhead-of-pmm-client-on-mysql-workloads/
  - 2018/01/22/measuring-the-potential-overhead-of-pmm-client-on-mysql-workloads.html

author: Fernando Ipar
status: publish
published: true
title: Measuring the potential overhead of pmm-client on MySQL workloads 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- mysql
- notes
tags:
- mysql
- notes
- pythian
---

*This article is a cross post from [Pythian's blog](https://blog.pythian.com/measuring-potential-overhead-pmm-client-mysql-workloads/)*

Having good historial metrics monitoring in place is critical for properly operating, maintaining and troubleshooting database systems, and [Percona Monitoring and Management](https://www.percona.com/doc/percona-monitoring-and-management/index.html) is one of the options we recommend to our clients for this.

One common concern among potential users is how using this may impact their database’s performance. As I could not find any conclusive information about this, I set out to do some basic tests and this post shows my results.

To begin, let me describe my setup. I used the following Google Cloud instances:

- One 4 vCPU instance for the MySQL server
- One 2 vCPU instance for the sysbench client
- One 1 vCPU instance for the PMM server

I used Percona Server 5.7 and PMM 1.5.3 installed via Docker. Slow query log was enabled with long_query_time set to 0 for all tests.

I ran sysbench with 200k, 1M and 10M rows using the legacy oltp script with the pareto and special distributions, and up to 64 client threads. After several runs and reviews of the results, I settled on 1M rows with pareto for extended tests, as other combinations showed minor variations on the results from this one.

I am well aware a synthetic workload is not representative but I think the results are still useful, though I would love to measure this on a real life workload (do let me know in the comments if you have done this already).

In a nutshell, I found some impact in performance (measured as throughput in transactions per second) when running sysbench with the PMM exporters which in my case was eliminated when I configured them to serve their metrics by HTTP instead of HTTPS.

The following graph shows box plots for throughput for pmm enabled or disabled, for a different number of threads, with and without ssl:

![throughput with pmm enabled or disabled, per threads, with and without ssl](/assets/images/Boxplot-pmm-client-overhead.png)

We can see that with SSL enabled there is a noticeable drop in throughput when the exporters are running, while this is not the case when SSL is disabled.

I arrived at the conclusion that it was worth repeating the tests with SSL disabled after creating [Flame Graphs](https://github.com/brendangregg/FlameGraph) from perf captures during sample runs. On them, the only significant increases were due to the exporters (mysqld_exporter and node_exporter, the qan exporter did not have any noticeable impact during my tests). The results from the tests show that this analysis pointed me in the right direction so while they are worth of separate blog posts, it is worth to at least recommend our readers to get familiar with this performance analysis tool.

Next is a scatter plot of throughput over time with ssl enabled:

![throughput per threads, pmm enabled or disabled, with ssl](/assets/images/with-ssl-pareto-1000000-plot.png)

On it we get a more clear picture of the impact of having the exporters running during the test.

Next is the same graphs but with SSL disabled:

![throughput per threads, pmm enabled or disabled, without ssl](/assets/images/without-ssl-pareto-1000000-plot.png)

Now it is much more difficult to differentiate the runs.

This is confirmed if we look at the 99 percentile of throughput for each case (here for 32 threads):

<table style="width: 90%; cell-padding: 2px; border: 2px solid black; text-align: center;">
<tbody>
<tr bgcolor="%9b9b9b">
<td style="border: 0px solid black;"><strong>PMM</strong></td>
<td style="border: 0px solid black;"><strong>SSL</strong></td>
<td style="border: 0px solid black;"><strong>tps (p99)</strong></td>
</tr>
<tr bgcolor="%efefef">
<td style="border: 0px solid black;">enabled</td>
<td style="border: 0px solid black;">enabled</td>
<td style="border: 0px solid black;">1167.518</td>
</tr>
<tr>
<td style="border: 0px solid black;">enabled</td>
<td style="border: 0px solid black;">disabled</td>
<td style="border: 0px solid black;">1397.397</td>
</tr>
<tr bgcolor="%efefef">
<td style="border: 0px solid black;">disabled</td>
<td style="border: 0px solid black;">disabled</td>
<td style="border: 0px solid black;">1429.097</td>
</tr>
</tbody>
</table>

Conclusion
PMM is a very good Open Source option for monitoring but as every instrumentation and monitoring layer you add to your stack, it won’t come for free. My very simple tests show that its impact may be significant under some scenarios, yet if it’s bad enough it may be mitigated by using HTTP instead of HTTPS for the exporters. Given the events that are unfolding in IT security as I type this, it may seem reckless to recommend disabling SSL as an “optimization”, but I think good engineering is all about informed tradeoffs and if you’re running this on a secure private network, how risky is it to expose monitoring metrics over HTTP instead of HTTPS? I would love to read answers to this question in the comments!

Finally, I think a similar cost is probably paid for the TLS layer on the pmm-server end. It would be very interesting to see an experiment like this repeated but on a different scenario: one pmm-server with several monitored clients.