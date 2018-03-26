---
layout: post
redirect_from:
  - 2018/03/26/running-sysbench-based-benchmarks-against-cassandra/
  - 2018/03/26/running-sysbench-based-benchmarks-against-cassandra.html

author: Fernando Ipar
status: publish
published: true
title: Running sysbench-based benchmarks against Cassandra 
author: Fernando Ipar
author_login: fernando
author_email:
categories:
- cassandra
- notes
tags:
- cassandra
- notes
- pythian
---

*This article is a cross post from [Pythian's blog](https://blog.pythian.com/running-sysbench-based-benchmarks-against-cassandra/)*

I was recently discussing benchmarking options for Cassandra with some colleagues and given my background with MySQL, sysbench was the first tool I thought of.

Sysbench is a high performance and flexible benchmark tool that can be used to run both Database and Operating System based experiments. In case you’re not familiar with it, there’s an excellent introduction to it by my colleague Martin Arrieta here.

One interesting thing about this tool is that while it includes out of the box support for MySQL and PostgreSQL, it uses Lua for scripting so other databases can be supported provided there’s a Lua driver for it.

Naturally, that means my quest for a way to run sysbench against Cassandra started with the search for a Cassandra Lua driver, which led me to https://github.com/thibaultcha/lua-cassandra

The next task was looking for a simple way to deploy benchmark clients without the need to install dependencies on the host OS. These days my answer to that involves Docker and what I found for this was a handy image from Severalnines: https://hub.docker.com/r/severalnines/sysbench/

This image was a good starting point but did not fully support my use case as I need to install custom Lua modules on the container, which requires installing some additional packages on it.

Given my full stack for this is Open Source I went ahead and modified the Dockerfile for this image to add what I needed.
I felt this was a change that could benefit others too as I’m probably not the only one using sysbench to run experiments against databases that don’t have a driver bundled with it, so I submitted the following PR which has been merged already: https://github.com/ashraf-s9s/sysbench-docker/pull/1

Putting it all together, I can now launch sysbench against a Cassandra cluster to test the performance of different schemas and workloads.

To give a simple example let’s consider the following Lua script:

	    #!/usr/bin/env sysbench

	    function event()
	    local cassandra = require "cassandra"

	    local peer= assert(cassandra.new {
		host = "172.17.0.2",
		port = 9042,
		keyspace = "test"
	    })

	    assert(peer:connect())

	    assert(peer:execute("select * from user"))
	    peer:close()
	    end
 

A couple of comments about the script:

172.17.0.2 is the IP address of the Cassandra node I want to connect to. In my case, this is another container, but be sure to change this as needed if you want to reproduce this test (or refer to this gist to see how I ran mine).
For the script to work, the Cluster must have a ‘test’ keyspace with a ‘user’ table on it (as you can see from the query, the table structure does not matter here).
We can use sysbench to execute it via docker like so:

	    telecaster:sysbench-docker fipar$ docker run -v ~/src/:/src/ --name=sb -it severalnines/sysbench bash -c 'luarocks install lua-cassandra --local; luarocks install luasocket --local; /src/tmp/benchmark1.lua run'
	    Warning: The directory '/root/.cache/luarocks' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing /usr/local/lib/luarocks/rocks/luarocks/2.4.3-1/bin/luarocks with sudo, you may want sudo's -H flag.
	    Installing https://luarocks.org/lua-cassandra-1.2.3-0.rockspec
	    ...snip...
	    Installing https://luarocks.org/luasocket-3.0rc1-2.src.rock
	    ...snip...
	    sysbench 1.0.13 (using bundled LuaJIT 2.1.0-beta2)

	    Running the test with following options:
	    Number of threads: 1
	    Initializing random number generator from current time


	    Initializing worker threads...

	    Threads started!


	    General statistics:
		total time:                          10.0017s
		total number of events:              2792

	    Latency (ms):
		    min:                                    2.17
		    avg:                                    3.57
		    max:                                   48.12
		    95th percentile:                        4.65
		    sum:                                 9958.60

	    Threads fairness:
		events (avg/stddev):           2792.0000/0.00
		execution time (avg/stddev):   9.9586/0.00
You can see I am installing the required Lua modules when starting the container. If a benchmark will be executed several times (which is usually the case) a better approach would be to further customize the Dockerfile to include the necessary modules. I have not done that in the PR though because I think that would bloat the existing image. You can also see that I’m making my machine’s src directory available on the container via the -v ~/src/:/src/. That’s why I can then execute the script from /src/ on the container. Be sure to adjust this as needed to point to a directory tree where the lua script can be found on your machine.

In conclusion, if you have benchmarking needs and have not considered sysbench, don’t be put off if your database of choice is not listed as supported: as long as there’s a Lua driver for it there’s a good chance that you will be able to use sysbench for the task!