- Feature Name: Caprine Operator
- Start Date: 2022-05-07
- DIP PR: [goatcorp/DIPs#29](https://github.com/goatcorp/DIPs/pull/29)
- Repo-Relevant Issue: [goatcorp/DIPs#25](https://github.com/goatcorp/DIPs/issues/25)

# Summary

[summary]: #summary

Caprine Operator is an email reporting service to be used for streamlining the human review process over repositories in the
core Dalamud ecosystem. The Operator sends emails with aggregated repository updates to subscribers to reduce the need for
manually checking a repository and figuring out what has been updated and what can be ignored. After the Operator has sent
its first email to a subscriber, subsequent emails will only include updates since that point. If there have been no updates
since the previous email, no additional emails will be sent to the subscriber.

# Motivation

[motivation]: #motivation

The introduction of Caprine Operator should reduce the cognitive load on participating reviewers, allowing them to spend more
energy focusing on other important projects. As an email-based system, it supports the processes of reviewers who have a robust
email setup for directing their attention.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Currently (as a prototype), Caprine Operator supports reporting updates for the plugin repository. The plugin repository review
process requires repository moderators to perform the following steps:
1. Identify that a pull request has been opened.
2. Determine if the pull request is an update to an existing plugin, or a new plugin.
3. If the pull request is for a new plugin, add the `new plugin` label to the pull request.
4. Perform a code review of the plugin, and add the `code reviewed` label to the pull request.
5. Perform a rules review of the plugin, and add the `rules reviewed` label to the pull request.
6. A core review team member will merge the plugin, provided that a majority of the core review team agrees to its addition.

Through its emails (henceforth referred to as reports), Caprine Operator streamlines steps 1, 4, 5, and 6 of this process.
Caprine Operator will show the pull request in an update report, along with its labels, so that subscribers (henceforth
referred to as readers) can see at a glance what the state of the pull request is. When a label is added, this will update
the `updated` timestamp on the pull request in the GitHub API, which will cause it to show up in the Operator's reports,
signalling to readers that it should be checked again.

A sample email at the time of writing is shown below:
![](https://raw.githubusercontent.com/goatcorp/operator/3c20cf7c45c7bd9180a8245acef8b62bd993f898/assets/email.png "Sample email from Caprine Operator")

Disregarding the egregious contrast between the `code reviewed` label and the Mac OS Mail dark background, this should show what
reports will look like. The title column is linked to the pull request for quick navigation, and colored labels are also shown to
signal what review step each pull request is at.

This example is provided as just a sample of what can be done with the Operator; review processes in other core Dalamud ecosystem
repositories can likely be streamlined in a similar manner.

## Usage

[usage]: #usage

To subscribe to updates, send an email to [caprine.operator@outlook.com](mailto:caprine.operator@outlook.com) with the subject: `[op] subscribe`. In the message body, include the following:
Field|Description
---|---
`github`|(Optional) Your GitHub username, which will (eventually) be used for highlighting sections of the report emails.
`interval`|How often Caprine Operator should send you updates when they are available. This field expects a value that can be parsed as a [`time.Duration`](https://pkg.go.dev/time#ParseDuration).

```
github: yourgithub
interval: 6h30m
```

If your email was well-formed, you will receive a confirmation email from Caprine Operator. Be sure to add the Operator to your Safe Senders list, or its updates will probably be flagged as spam.

To unsubscribe from Operator updates, send an email with the subject line `[op] unsubscribe`. You will receive a confirmation email once the operation completes.

Upon unsubscribing, all of your associated data in the database will be deleted, meaning you will need to resubmit it if you subscribe again in the future.

If you want to update your reader information, send an email to the Operator with the subject line `[op] update`. The body of the email follows the same conventions as subscribing. If you specify a field but leave it empty, that will be interpreted as deleting the value (unless you try to delete the reporting interval).

```
github:
interval: 3h
```

The above email body would delete any stored GitHub username, and adjust your report interval to 3 hours.

```
interval: 30m
```

The above email body would adjust your report interval to 30 minutes, without affecting your stored GitHub username.

After sending the update request, you will receive a confirmation email.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the DIP. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

This section is based on [goatcorp/operator](https://github.com/goatcorp/operator) at commit [3c20cf7c45c7bd9180a8245acef8b62bd993f898](https://github.com/goatcorp/operator/tree/3c20cf7c45c7bd9180a8245acef8b62bd993f898).

Caprine Operator has its source code stored in the [goatcorp/operator](https://github.com/goatcorp/operator/tree/3c20cf7c45c7bd9180a8245acef8b62bd993f898) repository. It is written in
the Go programming language, and uses PostgreSQL as its database. The repository contains Docker configuration files for portable execution.
These Docker files are for development and deployment only; readers need only subscribe to the service to use it. Currently, Caprine Operator
uses Outlook as its email provider; this decision has been made for several reasons:
* It doesn't seem to have any restrictions on automated usage.
* It exposes an accessible SMTP server and an IMAP server for external access.
* It doesn't require a specialized API client.
  * In contrast, Gmail is blocking access using basic authentication starting on [May 30, 2022](https://support.google.com/accounts/answer/6010255).
* It's free.

Prebuilt Docker images are accessible via goatcorp's [Docker image repository](https://github.com/goatcorp/operator/pkgs/container/operator).

## Data

[data]: #data

In order to track what has been updated since the previous report to a reader in a maintainable manner, the system requires that both readers
and report logs be stored. Upon unsubscribing from the service, all of the reader's data stored in the database is deleted.

### Reader

[reader]: #reader

The current Reader table schema is as follows:

|Constraints|Column|Type|Description|
|---|---|---|---|
|PK|`id`|integer|The record key, used for indexing and references.|
|U, N|`github`|varchar(40)|The reader's GitHub username, used for (eventually) highlighting relevant updates.|
|U|`email`|varchar(40)|The reader's email address.|
||`report_interval`|interval|The frequency at which the reader should be emailed, if there are updates.|
||`active`|bool|Whether or not emails should be sent to this reader.|

### Report

[report]: #report

The current Report table schema is as follows:

|Constraints|Column|Type|Description|
|---|---|---|---|
|PK|`id`|integer|The record key, used for indexing and references.|
||`sent_time`|datetime with time zone|The time at which this report was sent.|
|FK(Reader.id)|`reader_id`|integer|The reader that this report was sent to.|
||`skipped`|bool|Whether or not this report was skipped for the current interval, as a result of there being no updates.|

# Drawbacks

[drawbacks]: #drawbacks

* Data storage of any kind is a liability.
* This adds complexity to the core Dalamud ecosystem, both as a result of being an additional tool and through its terminology.
* Server resources are required to host the Operator.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

* As a plugin reviewer myself, it's annoying to have to check the plugin repository for updates, since the Discord webhook only shows opened PRs and PR comments, and it's combined with updates for other core Dalamud ecosystem repositories. First and foremost, I'm making this for myself.
* I considered using C# to write this, but I figured that it would have taken longer to set up the database code that way, just based on having worked with databases in both languages before.
* I also considered using Python to write this, but I didn't feel like dealing with virtual environments and duck typing this week.
* Finally, I considered using Rust, but I'm not familiar enough with Rust to write something that doesn't use escape hatches everywhere.
* For databases, I just felt like using PostgreSQL. I have no excuses.

# Prior art

[prior-art]: #prior-art

* Our Discord webhook is nice, but all of our repository updates are sent to a single channel, which makes it easy to miss things for specific repositories. Additionally, it can't update when labels are added to pull requests and issues.
* If I set desktop notifications on the webhook channel, they get filtered into the notification manager on Windows. However, that's too cramped for at-a-glance reading, and is subject to the other restrictions of Discord webhooks, too.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

* Bikeshedding: Should the email prefix for incoming emails be `[op]`, or something else?
* What other repositories should Caprine Operator support, and how should updates be formatted for them?

# Future possibilities

[future-possibilities]: #future-possibilities

* Expansion to other repositories, as noted above.
* Discord integration, if we can figure out how to do that reliably given how unstructured Discord is.
* Report channels, so that once further repository integration is implemented, we can opt into specific repositories for updates.
