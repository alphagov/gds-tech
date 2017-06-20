---
title: Logging
layout: recommendation
expires: 2017-12-01
---

# Short term queryable logs

For the purposes of incident management and debugging, it's useful to
have a system for storing and querying the most recent logs produced
by your application and infrastructure.

In general, tools for querying logs are not a good fit for long-term
archival, and vice versa.

## Shared Elasticsearch, Logstash and Kibana (ELK) stack

GDS has a shared account with logit, which offers on-demand ELK stacks
as a service.

This is suitable for a short-term place to store logs in order that
they are available and queryable.  This should not be considered a
durable store of logs; it is *not* suitable for long-term archival of
logs.

## Sumo Logic

Some teams have used [Sumo Logic][] successfully for storing and
querying logs.

## Self-hosted Elasticsearch, Logstash and Kibana (ELK)

Each team should not operate its own ELK stack for logging.
In the past we've found that ELK is relatively easy to set up
but teams tend to not prioritise keeping it updated.

ELK isn't appropriate for long term archival of logs. There
are services which are less expensive and more reliable.

# Long term archival of logs

You should have a way of storing logs which focuses on long-term
durability.  Some teams archive logs to S3 at the same time as sending
them to their ELK or Sumo Logic store.

## Querying from the archives

It's a good idea to be able to take logs from the long-term archive
and load them into a queryable tool (for example, a one-time-use ELK
stack).

If you are using Amazon Web Services, CloudWatch can run rudimentary
queries on older logs which may be good enough.

# Shipping logs

When implementing your mechanism to ship logs to your stores, here are
some questions to ask:

  * How do I ensure my logs get to both my short-term and long-term
    stores?
  * What happens if one of these stores is temporarily unavailable?
    What happens when it comes back online?

The [beats][] tools will detect periods of unavailability.  They will
remember how much they have sent and continue where they left off.
They also use a protocol which is backpressure-aware, so they will not
overload logstash when it comes back online.

One pattern is to have one process to ship logs to your long-term
archive, and a second process to fill the short-term query tool from
the long-term archive.  This way you can have confidence that
everything has been correctly archived.

# Retention periods

This section recommends some sensible default choices for retention
periods for logs.  These are a starting point for a discussion, or a
default choice in the absence of any other constraints.

Your product may have requirements which affect your choice of log
retention periods: for example, [PCI-DSS][] mandates 3 months of
easily-accessible logs, and a year (no more, no less) of archived
logs.

## Short-term

You shouldn't need to store a long period of logs in your short-term
queryable store.  Sensible retention periods for short-term queryable
logs are:

  * non-production environments: no more than 7 days
  * production environments: no more than 30 days

## Long-term

It's sensible to keep logs in long-term archives for at least a year.

[beats]: https://www.elastic.co/products/beats
[PCI-DSS]: https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard
[Sumo Logic]: https://www.sumologic.com/
