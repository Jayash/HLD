# Reliable, Scalable and Maintainable Application

non-functional requirements for any application

## Reliability
The system should continue to work correctly even in the face of adversity (hardware or software faults and human error)
- Hardware faults
- Software faults
- Human Errors

## Scalability
As the system grows there should be reasonable ways of dealing with the growth
- Calculate Load: How much load an application has to handle
- Calculate Performance: how performant must be the application
- Approaches for coping with load

  ---

Twitter example
Twitter main operations

- Post tweet: a user can publish a new message to their followers (4.6k req/sec, over 12k req/sec peak)
- Home timeline: a user can view tweets posted by the people they follow (300k req/sec)
  
Two ways of implementing those operations:

- Posting a tweet simply inserts the new tweet into a global collection of tweets. When a user requests their home timeline, look up all the people they follow, find all the tweets for those users, and merge them (sorted by time). This could be done with a SQL JOIN.
- Maintain a cache for each user's home timeline. When a user posts a tweet, look up all the people who follow that user, and insert the new tweet into each of their home timeline caches.
Approach 1, systems struggle to keep up with the load of home timeline queries. So the company switched to approach 2. The average rate of published tweets is almost two orders of magnitude lower than the rate of home timeline reads.

Downside of approach 2 is that posting a tweet now requires a lot of extra work. Some users have over 30 million followers. A single tweet may result in over 30 million writes to home timelines.

Twitter moved to an hybrid of both approaches. Tweets continue to be fanned out to home timelines but a small number of users with a very large number of followers are fetched separately and merged with that user's home timeline when it is read, like in approach 1.

---
Describing performance
What happens when the load increases:

- How is the performance affected?
- How much do you need to increase your resources?
In a batch processing system such as Hadoop, we usually care about throughput, or the number of records we can process per second.

> Latency and response time
>> The response time is what the client sees. Latency is the duration that a request is waiting to be handled.

## Maintainability:
Over time, many different people will work on the system, and they should all be able to work on it productively
- Operability: Make it easy for operations team to keep the system running smoothly
- Simplicity: Managing Complexity
- Evolvability: Making change easy
