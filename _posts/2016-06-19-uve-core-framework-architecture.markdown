---
layout: post
title:  "UVE Core Framework Architecture"
author: <a href="https://github.com/fwso">James Tang</a>
date:   2016-06-19 05:01:01
categories:
  - Architecture
tags:
  - "Web Framework"
---

## Introduction

Three years ago, I joined Weibo Recommendation department. At the time, the
recommendation system was at the very early stage, our services was evolving very
quickly. One year later, advertising system has also been integrated with the
recommendation system. Consequently, a new layer called Unified Value Evaluation
(UVE) emerged. UVE became the unified endpoint for recommendation and advertising
system.

The first 5 versions(yes, iterated very quickly) of UVE were implemented with Nginx/PHP-FPM.
Unfortunately, UVE service depends on many internal resources such as Redis, RPC, etc.
But PHP is not good at handling TCP connections in persistent way.

After some investigation and tests, we decided to choose [OpenResty](https://openresty.org/en/)(Nginx/Lua) as the solution, for
it can work with many existing PHP services seamlessly. And finally, the [UVE Core Framework](https://github.com/uveio/uve_core)
in Lua.

## Features

Main features compared to the previous PHP version:

- high performance
- less resource(CPU, RAM) usage
- configurable
- easy to deploy
- unified log format

## Architecture

The architecture was very specialized for our requirements, so it's not a general
purpose web framework.

![UVE Core Architecture](/images/uve-core-architecture.png)

The "UVE Core" is main procedure which strings together all other components:

- The common components, such as Logger, Redis Utils, etc.
- The business components, such as GlobalStrategy, Service Modules, etc.

The main procedure workflow:

![UVE Core Workflow](/images/uve-core-workflow.png)

Each stage of the workflow is modularized and can be configured as need.

The Service Modules and scheduling is the most important part of the workflow.
Although the module scheduler can be implemented in different way, the following
diagram is our ad hoc implementation:

![UVE Core Service Modules Scheduler](/images/uve-core-workflow-module-schduler.png)

## Log Format

A semi-structured log format was introduced with this framework. The purpose of
the log format is easy to process with good extensibility.

The log format defines three level, but more sub-level can be implemented as needed.

![UVE Core Log Format](/images/uve-core-log-format-diagram.png)

The log is constructed from `key:value` basic fields, each level can have as many `key:value`
fields as possible. Fields are separated with separator of different level, the
separators of the three level are defined below.

### Level 1 separator

Group Separator: `0x1A`.
Field Separator: `0x1C`.

### Level 2 separator

Group Separator: `0x01`.
Field Separator: `0x1D`.

### Level 3 separator

Group Separator: `0x1E`.
Field Separator: `0x1F`.

Every level has a Group Separator and a Field Separator. In practice, the Level 1 only
has one group, so the Group Separator is only used to normalize the log format which
makes it easier to process the log recursively.

Following is an example log has been parsed to JSON:

```
{
   "feedsnum": "2",
   "from": "1051513010",
   "platform": "iphone",
   "reqtime": "1427178973",
   "service_name": "article",
   "tmeta_l2": [
       {
           "authoruid": "1633520291",
           "bgid": "1234567890123456",
           "category": "nr",
           "key": "article_app",
           "oid": "1022:1001603821084397949239",
           "tmeta_l3": [
               {
                   "itemid": "1234567890",
                   "loc": "1",
                   "recotype": "arrarticle",
                   "ruletype": "a"
               },
               {
                   "itemid": "1234567891",
                   "loc": "2",
                   "recotype": "arrarticle",
                   "ruletype": "a"
               }
           ]
       },
       {
           "authoruid": "1496913734",
           "bgid": "2234567890123456",
           "category": "ad",
           "key": "article_app",
           "oid": "1022:1001603821496853214821",
           "tmeta_l3": [
               {
                   "itemid": "3334567890",
                   "loc": "3",
                   "recotype": "arrarticle",
                   "ruletype": "b"
               },
               {
                   "itemid": "6634567891",
                   "loc": "4",
                   "recotype": "arrarticle",
                   "ruletype": "b"
               }
           ]
       }
   ],
   "uid": "1716016202",
   "version": "5.1.5"
}
```

## Service Endpoint

The default endpoint looks like this:

```
http://example.com/uve/service/SERVICE_NAME
```

The `SERVICE_NAME` will be used as the service identifier to load corresponding configuration.

The URL pattern can be customized according to your requirements.

## Configuration

Configuration is implemented as a Lua module. The out most attribute is the same
as the the `SERVICE_NAME` described in preceding section. Here is an example:

```lua
module(..., package.seeall)

profile = {
   modules = {
       [25] = 'uve.sm.profile_s1',
       [26] = 'uve.sm.profile_s2',
   },
   scheduler = 'uve.scheduler.profile',
   init = 'uve.init.profile',
   strategy = 'uve.strategy.profile',
   prerender = 'uve.prerender.profile',
   render = 'uve.render.localrender',
   postrender = 'uve.postrender.profile',
   postrequest = 'uve.postrequest.profile',
}
```

Notice the attributes of the configuration are corresponding to the workflow stages.
Only the `modules` and `render` are required, others attributes are optional, depends
on your requirements.

## Applications

As described in preceding sections, the UVE Core Framework is dedicated for [Weibo](http://weibo.com) advertising and recommendation services, so it's not
expected to be a general purpose web framework. But it may be useful for some guys
have similar requirements with us, or it can be an example of [OpenResty](https://openresty.org/en/).
