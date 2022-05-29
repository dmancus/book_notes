# Designing Data-Inteinsive Applications

# Part 1 - Foundations of Data Systems

## Chapter 1 - Reliable, Scalable, and Maintainable Applications
- Many applications are `data-intensive` instead of `compute-intensive`
- Standard building blocks for a data-intensive application:
  - database
  - cache - speed up reads
  - search index - support user search by key word or filter
  - stream processing - send messages to another process
  - batch processing
### Thinking About Data Systems
- Reliability - System should work correctly in the face of `adversity`
- Scalability - There should be ways for the system to grow
- Maintainability - Many different people will work on the system both to maintain behavior and to add new behavior.  They should be able to do so productively.
### Reliability
- Things that can go wrong are called `faults`.  We want `fault-tolerant` or `resilient` systems
- `fault` is different from `failure`.  Fault is when one piece misbehaves, failure is when the system misbehaves.  Cope with faults to minimize failures.
- Increasing `faults` may be desirable to reduce `failures`, a la Chaos Monkey
#### Hardware Faults
- In modern eco system, we want to move toward architecture that can tolerate the loss of entire machines
#### Software Errors
- No simple solution here, need lots of things
- testing, process isolation, allow crash/restart, measure, monitor, analyze behavior
- Example: If a system should guarantee something like equal number of incoming messages and outgoing messages, it can monitor itself and alert when a discrepancy arises
#### Human Errors
- Try to make it hard to "do the wrong thing"
- Decouple the place where humans make mistakes from the place where they cause `failure`. e.g., create sandbox environments
- Test thoroughly, from unit test to system integration test, to manual test
- Allow for Easy Recovery from errors
- Set up good monitoring, aka `telemetry`
- Have good management practices
### Scalability
#### Describing Load
- `load parameters` describe load
- Actual parameters depend on architecture
- Twitter Example 
  - 2 key operations: `post tweet` and `home timeline`
  - 12k writes per second is easy, but scaling challenge comes from `fan-out`.  Each user follows many people, and each user is followed by many people.
  - Two options to handle this:
    1. Posting tweet just inserts one entry.  Then when querying home timeline, you query for all the tweets from everyone they follow
    2. Maintain a cache of each user's home timeline, and update everyone's timeline when new tweet gets created.  Then lookups are much faster
  - Twitter went from approach 1 to 2, and is now moving toward a hybrid approach
#### Describing Performance
- Two ways to think about it:
  - If you increase a `load parameter` and dont change system resources, how does performance change
  - If you increase a `load parameter`, how much do you need to increase resources to keep performance unchanged
- batch processing systems like Hadoop - `throughput`
- Online systems - `response time`
  - `response times` vary, so think of them as a `distribution` of times to measure
- Rather than just use `average response time`, its better to look at `percentiles`
- High percentiles are called `tail latencies` - Important beause they actually show users' experience
- Example - Amazon measures `99.9th percentile` because that is likely to reflect the power user's experience, aka the account with lots of data
- Percentiles are often used in `Service Level Objectives` and `Service Level Agreements`
- We can look at some percentiles to get an idea of performance: `p50`, `p95`, `p99`, `p999`
#### Approaches for Coping with Load
- `Scale Up` vs `Scale Out`
- In most cases, some mix of both can be good.  Using several fairly powerful machines can be simpler and cheaper than a lot of small virtual machines
- Stateless services are easy to distribute, but stateful ones are not
- Conventional wisdom is to try to keep db on a single node, but this may change, lots of new products to try to solve this
### Maintainability
- `Operability` - Make it easy for operations teams to keep system running well
- `Simplicity` - Remove complexity of the system.  Different from creating simple UI
- `Evolvability` - Make it easy to change the system in the future
#### Operability
Operations teams do:
- Monitoring health and restore service
- Track the cause of problems
- Keep software up to date, including security patches
- Be aware of system interactions to avoid problematic changes ahead of time
- Capacity Planning
- Establish good practices for deployment, config management, etc
- Perform complex maintenance tasks like app migrations
- Maintain security of the system as config changes are made
- Define processes that make operations predictable and keep things stable
- Preserve organizational knowledge/institutional knowledge
#### Simplicity: Managing Complexity
- Use `abstraction` to remove accidental complexity.  Hide a bunch of details behind a clean facade
- Finding good abstractions is hard
#### Evolvability: Making Change Easy
- `Agile` and `TDD` try to provide tools for dealing with changes
- What can we do with larger scale changes, like Twitter Architecture

# Part 2 - Distributed Data

# Part 3 - Derived Data