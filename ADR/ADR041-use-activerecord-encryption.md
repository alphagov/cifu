# ADR041: Use ActiveRecord encryption in Runner

Date: 2025-02-12

## Status

Accepted

## Context

Runner will be storing user submitted data which is likely to include sensitive information, such as PII including document IDs like passport numbers.

Aurora RDS databases will already encrypt when data is saved to disk with AWS KMS, but while data is held in memory it is not. To mitigate against this we can make use [ActiveRecord Encryption](https://guides.rubyonrails.org/active_record_encryption.html).

### Pros

* If there is more access to credentials to query data in the database than there is to your encryption keys. For example, if we allowed devs with the support role in AWS to query the database, we might want to make sure they don’t have access to user submission data. However, if we decide to do this in the future we can start encrypting the data later, as Active Record encryptions supports reading unencrypted data when encryption has been enabled.
* If sensitive data is leaked, for example in the SQL logs, it's encrypted and unreadable.
* An additional layer of defence.

### Cons

* Extra keys to manage, extra configuration in Rails.
* Difficulty accessing the data if we ever need to inspect it. The main reason I can see for doing this is if for some reason we fail to send a submission due to a data error and need to inspect the data itself to diagnose and fix any issues.

## Decision

Use ActiveRecord Encryption.

## Consequences

There will be more keys to manage. It will be harder for us to read the data.
