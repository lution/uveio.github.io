---
layout: post
title:  "PHP Memcached Timeout Setting"
author: James Tang
date:   2015-07-08 04:15:30
categories:
  - Code
  - Notes
tags:
  - PHP
  - Memcached
---

PHP Memcached extension does not provides direct method for timeout setting, but can be implemented with [memcached.setoption], e.g.

{% highlight php %}
$mc = new Memcached();
$mc->addServer(MC_HOST, MC_PORT);
$mc->setOption(Memcached::OPT_NO_BLOCK, true);
$mc->setOption(Memcached::OPT_CONNECT_TIMEOUT,  5);
$mc->setOption(Memcached::OPT_POLL_TIMEOUT, 5);
$mc->setOption(Memcached::OPT_SEND_TIMEOUT, 5);
$mc->setOption(Memcached::OPT_RECV_TIMEOUT, 5);
$mc->setOption(Memcached::OPT_RETRY_TIMEOUT, 3);
{% endhighlight %}

For more information, see [memcached.setoption].

[memcached.setoption]:      http://php.net/manual/en/memcached.setoption.php

