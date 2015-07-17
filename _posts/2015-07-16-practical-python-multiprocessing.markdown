---
layout: post
title:  "Practical Python: Multiprocessing for log processing"
author: James Tang
date:   2015-07-16 21:50:00
categories:
  - Code
  - Notes
tags:
  - Python
  - Multiprocessing
---

## Question

How to calculate the requests for each services from the access logs efficiently. 

The access log format looks like this:

{% highlight bash %}
172.16.40.10 - - [13/Jul/2015:23:46:07 +0800] "GET /uve/service/myprofile?uid=24696203042323&from=53093010&host_uid=1786844635343&page=1&lang=zh_CN&ip=223.73.254.132 HTTP/1.1" - ^200^ 84 "-" "-" "-" ^"0.003"^ ^"-"^
172.16.40.15 - - [13/Jul/2015:23:54:06 +0800] "GET /uve/service/myprofile?uid=2674542995323&from=51195010&host_uid=2640896377343&page=1&lang=zh_CN&ip=58.22.114.121 HTTP/1.1" - ^200^ 84 "-" "-" "-" ^"0.003"^ ^"-"^
172.16.40.19 - - [13/Jul/2015:23:32:24 +0800] "GET /uve/service/item_page?uid=2559544720&mid=3864252290896703&isRecom=-1&from=53093010224&v_p=21&ip=182.150.150.210&wm=3333_2001&appid=6&source=3439264077&ouid=201849903475 HTTP/1.1" - ^200^ 1287 "-" "-" "-" ^"0.114"^ ^"-"^
172.16.40.25 - - [13/Jul/2015:23:44:42 +0800] "GET /uve/service/hot_tweets?uid=52462459412&from=53095010&host_uid=50551834347994&page=1&lang=zh_CN&ip=222.160.218.74 HTTP/1.1" - ^200^ 84 "-" "-" "-" ^"0.008"^ ^"-"^
{% endhighlight %}

The access logs are stored in multiple files, with around 1GB size for each.

The results will be saved to a file with the following format:

{% highlight bash %}
  /uve/service/myprofile 382388701
  /uve/service/item_page 53227923
  /uve/service/hot_tweets 160399456
{% endhighlight %}

The task should be done on a sigle server with 12Cores/16GB RAM.

## Solution

In order to finish the task as quick as possible, we have to make full use of the multi-cores, [multiprocessing] will be used to achieve this purpose.

First, split all the log files into ten groups:

{% highlight python %}
def get_parts(date):
    jobs = []
    files = glob.glob('/logs/access-' + date + '_*')
    _print('Total files: ' + str(len(files)))
    part_len = len(files) / 10
    last_pos = 0
    indexes = len(files) - 1
    while last_pos < indexes:
        job = files[last_pos:last_pos+part_len]
        jobs.append(job)
        last_pos = last_pos + part_len
    return jobs
{% endhighlight %}

Then run each group parally, and save the results to separate temp files:

{% highlight python %}
def do_process(files):
    stats = {}
    for file in files:
        fp = open(file, 'r')
        #count = 0
        for log in fp:
            #count += 1
            #if count > 10000:
            #    break
            match = regp.search(log)
            if match:
                method_key = match.group(1)
                if stats.has_key(method_key):
                    stats[method_key] += 1
                else:
                    stats[method_key] = 1
        fp.close()
    tmp_fp = open(TMP_DIR + '/tmp-' + multiprocessing.current_process().name, 'w')
    for key in stats.keys():
        tmp_fp.write(key + "\t" + str(stats[key]) + "\n")
    tmp_fp.close()
{% endhighlight %}

And then merge the results:

{% highlight python %}
def merge(date):
    tmp_files = glob.glob(TMP_DIR + '/tmp-*')
    stats = {}
    for tf in tmp_files:
        fp = open(tf, 'r')
        for line in fp:
            parts = line.split("\t")
            if len(parts) >= 2:
                key = parts[0]
                value = int(parts[1])
                if not stats.has_key(key):
                    stats[key] = value
                else:
                    stats[key] += value
        fp.close()
    data_fp = open(DATA_DIR + '/' + date + '.txt', 'w')
    for key in stats.keys():
        data_fp.write(key + "\t" + str(stats[key]) + "\n")
    data_fp.close()
{% endhighlight %}

Last, remember to remove the temp files:

{% highlight python %}
os.system('rm -f ' + TMP_DIR + '/tmp-*')
{% endhighlight %}

This question is simple but typical, this [template] shows the skeleton of this solution:

{% gist fwso/79636c531ddbc3ec1f6e %}

You may noticed the `_print` function, it's just a wrap of the standard print with datetime prefix to the message, but it's very useful in practice. When we are processing large amount of data, the task must be ran in background for a long time, so we have to redirect the standard output to log file, the datetime of the output will help us how long the program actually ran, and it's especially helpful when when runtime error occurred.

[multiprocessing]: https://docs.python.org/2/library/multiprocessing.html
[template]: https://gist.github.com/fwso/79636c531ddbc3ec1f6e

