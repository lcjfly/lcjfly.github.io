---
layout:     post
title:      "NodeJS Cluster"
subtitle:   "make NodeJS support multi-CPUs"
date:       2015-11-02 20:23:00
author:     "luchenjie"
header-img: "img/post-bg-04.jpg"
---

<h2>talk is cheap, show me the code.</h2>
<pre class="prettyprint linenums">
var cluster = require('cluster');
var http = require('http');
if (cluster.isMaster) {
	var numWorkers = require('os').cpus().length;
	console.log('Master cluster setting up '+numWorkers+' workers...');
	for(var i=0; i<numWorkers;i++) {
		cluster.fork();
	}

	cluster.on('online', function(worker) {
		console.log('Worker '+worker.process.pid+' is online');
		setTimeout(function() {
			worker.send('hello from the master '+worker.process.pid);
		}, 3000);
		worker.on('message', function(message) {
			console.log('worker on message:'+message);
		});
	});

	cluster.on('exit', function(worker, code, signal) {
		console.log('Worker '+worker.process.pid+' died with code:'+code+', and signal:'+signal);
		console.log('starting a new worker');
		cluster.fork();
	});
} else {
	http.createServer(function(req, res) {
		res.writeHead(200);
		res.end('process '+process.pid+' say hello');
	}).listen(8000);

	process.on('message', function(message) {
		console.log('process on message:'+message);
		msg = message;
	});
}
</pre>

<h2>start node cluster server</h2>
<img src="{{ "/img/nodejs-cluster-img-1.jpg" | prepend: site.baseurl }}">

<h2>CURL test</h2>
<img src="{{ "/img/nodejs-cluster-img-2.jpg" | prepend: site.baseurl }}">

<h2>Apache Benchmark test</h2>
<img src="{{ "/img/nodejs-cluster-img-3.jpg" | prepend: site.baseurl }}">

NodeJS Cluster API: <a href="https://nodejs.org/api/cluster.html#cluster_cluster">https://nodejs.org/api/cluster.html#cluster_cluster</a>