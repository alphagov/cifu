# ADR038: SolidQueue and ActiveJob for managing background processing

Date: 2024-11-25

## Status

Approved

## Context

Forms will be using AWS SES to deliver emails to form processors with files attached to the emails.

For our team to successfully process submission emails we must have:
* a way to retry the email on failed submissions, for example if an email bounces.
* a queuing mechanism to send emails asynchronously

For this we looked at Sidekiq and Redis for managing a queue and to give us the ability to retry failed submissions - such as bounced emails. However with Redis, the details of a submission are ephemeral once a job is taken to be processed.

Due to the way Sidekiq uses Redis in the free version is it possible for it to lose jobs if the process crashes. It is very important that we do not lose submissions, so this risk isn't acceptable to us.

We considered just using SQS, but there is a message size limit of 256kb. We cannot be sure that there won't be form submissions that exceed this size.

So if the processing fails for whatever reason after the details have been obtained from the queue, we are unable to retry the submission because that data is gone.

Sidekiq can retain that data, but only on their Pro offering which costs money.

Solid Queue, which is a queue backed by a SQL database, is the default for ActiveJob in Rails 8. This provides us a guarantee that jobs will not be lost, as it works by locking a database row for reads while it is processing a job and only deleting the database row after the job has successfully completed.

 We can use SolidQueue with a PostgresSQL database that we set up in AWS, which we already have running for other parts of the system.

There is some prior art in that GOV.UK do a similar thing with a combination of Sidekiq and storing details in a database, which then gets scrubbed once a job is processed (this was all built before SolidQueue became available).

The implication here for our architecture is that we will need to add a PostgresSQL database to the Forms Runner for processing submissions.

In terms of performance and additional infrastructure, we have plans for the future to store form filling data in something more persistent than Redis as part of things like Save and Return. Given that we will need that resource anyway, setting up an additional database for Forms Runner isn't a problem. Another ADR will be forthcoming separately to record that we will need an additional Aurora cluster, but worth mentioning here as it's relevant.

## Decision

We will use SolidQueue and ActiveJob to manage the background processing of emails. We will use PostgresSQL/RDS for the database on the Forms Runner.

## Consequences

We will have a reliable way to deliver emails to form processors with files attached to the emails.

We must encrypt the data in the database, as this will be user-submitted data.

We have an additional infrastructure dependency on PostgresSQL on Forms Runner.
