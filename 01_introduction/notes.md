# Introduction

## Performance activities:  

1. Setting performance objectives and performance modeling (how well do we want the system to perform)
2. Performance characterization of prototype software or hardware
3. Performance analysis of development code, pre-integration 
4. Performing non-regression testing of software builds, pre- or post-release (confirm that the software update doesn't cause a regression in performance)
5. Benchmarking/Benchmarketing for software releases
6. Proof-of-concept testing in the target environment
7. Configuration optimization for production deployment
8. Monitoring of running production software
9. Performance analysis of issues

Steps 1-5: Software Dev Timeline  
Steps 6-9: Customer Environment Timeline

## Case Studies

### Slow Disks

**Problem:** "slow disks" on database server  

**Questions asked:**  

* Is there currently a database performance issue? How is it measured?
* How long has this issue been present?
* Has anything changed with the database recently?
* Why were the disks suspected?

**Extra information:**  

* log for queries slower than 1000 ms, dozens per hour recently
* AcmeMon reported 
    * disk util is at 80% (high) using a minute interval
    * CPU util has been steady
    * no saturation or error statistics for disks
* database load did not increase (rate of queries has been steady)

**Independently found information:**  

* `/proc` yields 0 disk error counters
* iostat yields 100% disk util at times
* dynamic tracing used to show that database blocks for many milliseconds during a filesystem read, during a query
* disk I/O is caused by fs cache misses, so when checked, it turns out that the fs cache has a 91% hit rate, and is smaller than the fs cache of other servers that serve similar workloads.

**Final verdict:** a separate service was causing this issue by taking up a growing amount of memory

### Software Change

**Problem:** software devs unsure of if new core feature could hurt performance  

**Information learned:**  

* client simulator applies an average production workload but doesn't account for variance
* requests level off at 700, with the server resources and threads largely idle
* requests level off at 700 for the new version as well
* old software was at 20% CPU, new software was at 30% CPU
* client sim was single threaded, and one thread is spending 100% of its time executing on-CPU

**Action Taken:** 

* non-regression testing of new application version (non-regression testing is for confirming if a software/hardware change does *not* regress performance, hence, *non*-regression testing).  
* started at 100 requests, and then upped the number by 100 every minute until it hit 1000
* ran 700 rps and checked resource utilization during that time
* client sim was run in parallel on different client systems

**Final Verdict:** there *is* a regression with the new software, and the client sim is single-threaded, which can be a bottleneck
