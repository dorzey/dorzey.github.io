---
layout: post
title: "How Miro migrated its analytics event tracking system"
canonical_url: "https://medium.com/miro-engineering/how-miro-migrated-its-analytics-event-tracking-system-46a6b1d2622e"
date: 2023-11-14
image: /assets/images/birds.png
---

In this post, we’ll take you behind the scenes of how we migrated our analytics event system — responsible for handling ~3.5 billion events per day — without losing a single event. We’ll go into the technical details and the strategy we used that allowed us to leverage Kubernetes, Kafka, Apache Spark, and Airflow to safeguard the events that power some of Miro’s key business decisions.

# In the beginning: 2017–2021
By 2017 Miro had reached 1M users. In September of that year, Miro put its in-house analytics event tracking system, called Event Hub, into production. On its first day it received its first 1.5 million events. Event Hub consists of the following:

* A Spring Boot API that collects events from various backend and frontend clients; the events are written to a Kafka topic;
* A Kafka that is used as a durable message broker;
* An Apache Spark Data Pipeline, in Java, that runs once a day. It conforms the events into a common format, applies enrichments, and makes the events available for downstream use cases.
The initial deployment of Event Hub was done as a raw deployment on EC2. Only some parts of the deployment were done as infrastructure-as-code, and it turned out to be remarkably stable with very few incidents.

# During Hypergrowth: 2021–2022
During 2021–2022, Miro experienced dramatic growth, reaching 30M registered users in 2021. In turn, our Event Hub instance was receiving around 2.5 billion events per day.

Miro also grew its data team during this time, which helped us improve our data capabilities. This led to the data pipeline being simplified, re-implemented in Scala Spark, migrated to Databricks, and being orchestrated via Airflow. This allowed the team to increase the frequency the pipeline ran to hourly. This allowed us to both provide fresher data to downstream analytics use cases and reduce our mean time to recover if there was a failure.

In this period, Miro also adopted Kubernetes and outlined a golden path for teams to deploy their services.

# Decision time: 2023
By the beginning of 2023 we were processing around 3 billion events per day. At the same time, a few circumstances coincided which altogether led us to the decision of completing the migration:

1. Event quality was beginning to suffer. This negatively impacted Miro’s ability to make reliable data-informed decisions. At the beginning, Miro deliberately chose a permissive model for events, but once the organization scaled this became unsustainable. We wanted to introduce Event Validation to improve the quality, but this would have been risky to do on the existing infrastructure.
2. We were asked to bring the Event Hub deployment in line with Miro best practices and leverage all the internal tooling around Kubernetes, Istio, and Terraform.
3. While Event Hub had very few incidents during its lifespan, maintenance and upgrades were a delicate matter. It was now running on several EC2 instances. We knew that losing even just one of them would have meant hours of downtime. The number of moving parts was destined to increase with the growth in Miro users. And so was the probability of a catastrophic failure.

# The migration: 2023
We decided to migrate, but now we needed to understand how to make it happen.

> “Plans are of little importance, but planning is essential” — Winston Churchill.

![Birds migrating
]({{ site.baseurl }}/assets/images/birds.png)

## 1. Define the plan
We started the process knowing that we needed a plan. But we all agreed that we would not have all the knowledge we needed at the beginning, so we should be adaptable to changing the plan as we progressed. We had a regular sync on progress to understand if we needed to update the plan.

## 2. Create infrastructure
We needed somewhere to deploy Event Hub, so we first created the infrastructure needed for our deployment. We could reuse some parts of the existing infrastructure, like Kubernetes, but other parts, like Kafka, had to be created.

## 3. Observability
The previous iteration of Event Hub had been running for years. The engineering team that cared lovingly for this system grew intimately familiar with its usage profile. But the significant design changes we were about to make were a source of uncertainty. Observability proved to be crucial to proceed with confidence. Luckily, a lot of instrumentation at the API Gateway, Kubernetes, and Infrastructure level was provided out of the box by Miro internal tooling. Moreover, the experience with the previous system informed us where to focus our attention first.

## 4. Migrate Event Hub to staging
Once we had the infrastructure necessary in the staging environment, we could deploy Event Hub to it. At this point we were able to go end to end. We could post an event into the API, move it through Kafka, and process it through the Spark Data Pipeline.

## 5. Load test
We were now running Event Hub in staging, but it was new infrastructure. We were unsure how it would perform, and if the infrastructure was correctly sized. Paired with the effort put toward observability highlighted above, we were able to run a load test that provided a realistic production volume of events. This required several iterations to correctly size the infrastructure to achieve the necessary performance. This also provided valuable insight into how we should define our auto-scaling policies.

## 6. Deploy to production
We were now confident that we could deploy Event Hub to production and begin the process of switching the event tracking over to it.

## 7. Prepare the data pipelines
Before we could switch the traffic, we had to make some changes to the Spark Data Pipeline so it could process the events from the old Event Hub and the new Event Hub deployments.

## 8. Switch Clients
Now we could switch over the event tracking. We had identified ~40 different services/clients sending events to Event Hub. Each one needed to be updated to point to the new infrastructure. We started with the clients with the lowest priority and/or least critical events first. This allowed us to validate the full end-to-end flow while minimizing the risk if we did hit a problem. It took about 6 weeks to switch over all the clients and lots of coordination with different engineering teams.

## 9. Success
By August 2023 we had migrated to the new deployment and were processing ~3.5 billion events per day. We had successfully switched to the new Event Hub deployment and not lost a single event or caused any delays to the data we deliver to the downstream use cases.

# The lessons we learned
During the migration we learned a few valuable lessons:

* *Make your work visible*

When many teams, streams, and stakeholders are involved, you must be transparent about the work in progress and what has been completed.

* *Individuals reaching out made a big difference*

While we aligned on expectations and priorities, we hit the inevitable problem that teams had conflicting priorities, deadlines, etc. We found out that individuals reaching out to people made a big difference. We were able to provide the context and specific help to each of the teams to help the work to be completed faster.

* *Bake in event quality from the beginning*

If you’re starting from scratch, then make event quality a first class concern from the beginning. Going back and adding it was much more difficult and carried more risk.

* *Lean on internal tooling*

A lot of the machinery that allowed us to pull this off was available because of the excellent work other teams in Miro did. Event Hub evolved from being directly exposed to the public internet through a simple load balancer to benefitting from state of the art scalable infrastructure and DDoS protection.