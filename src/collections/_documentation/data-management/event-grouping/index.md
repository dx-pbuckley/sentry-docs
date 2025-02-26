---
title: 'Grouping & Fingerprints'
sidebar_order: 0
---

All events have a fingerprint and events with the same fingerprint are grouped together. By default, Sentry will run one of our built-in grouping algorithms to generate a fingerprint based on information available within the event such as `stacktrace`, `exception`, and `message`. To extend the default grouping behavior or change it completely, you can use a combination of the following options:

1. [SDK Fingerprinting]({%- link _documentation/data-management/event-grouping/sdk-fingerprinting.md -%})
2. [Server-side Fingerprinting]({%- link _documentation/data-management/event-grouping/server-side-fingerprinting.md -%})
3. [Custom Grouping Enhancements]({%- link _documentation/data-management/event-grouping/grouping-enhancements.md -%})

## Grouping Algorithms

Each time the default behavior is modified, Sentry releases it as a new version so it does not affect how existing issues are grouped. When you create a Sentry project, the latest and greatest version of the grouping algorithm is automatically selected. This means that the grouping behavior is consistent within a project. If you want to upgrade an existing project to a new grouping algorithm version, you can do so in the project settings. Note that you can only upgrade whhen upgrading you will very likely see new groups being created.

All versions consider the `stacktrace`, `exception` and `message`. 

### Grouping by Stacktrace

When Sentry detects a stack trace in the event data (either directly or as part of an exception), the grouping is effectively based entirely on the stack trace. This grouping is fairly involved but easy enough to understand.

The first and most important part is that Sentry only groups by stack trace frames reported to be from your application. Not all SDKs report this, but if that information is provided, it’s used for grouping. This means that if two stack traces differ only in parts of the stack that are not related to the application, they'll still be grouped together.

Depending on the information available, the following data can be used for each stack trace frame:

- Module name
- Normalized filename (with revision hashes, etc. removed)
- Normalized context line (essentially a cleaned up version of the sourcecode of the affected line, if provided)

This grouping usually works well, but two specific situations can throw it off if not dealt with:

- Minimized JavaScript sourcecode will destroy the grouping in really bad ways. Because of this you should ensure that Sentry can access your [Source Maps]({%- link _documentation/platforms/javascript/index.md -%}#source-maps).
- If you modify your stack trace by introducing a new level through the use of decorators, your stack trace will change and so will the grouping. To handle this, many SDKs support hiding irrelevant stack trace frames. (For example, the Python SDK will skip all stack frames with a local variable called `__traceback_hide__` set to _True_).

### Grouping By Exception

If no stack trace is available but exception info is, then the grouping will consider the `type` and `value` of the exception, as long as both pieces of data are present on the event. This grouping is a lot less reliable because of changing error messages.

### Fallback Grouping

If everything else fails, grouping falls back to messages. In this case the grouping algorithm will try to use the message without any parameters, but if that is not available, it will use the full message attribute.