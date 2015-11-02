---
layout:     post
title:      "NodeJS Cluster"
subtitle:   "make NodeJS support multi-CPUs"
date:       2015-11-02 20:23:00
author:     "luchenjie"
header-img: "img/post-bg-04.jpg"
---

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
			if(worker.process.pid %2 ===0)
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
	var msg = 'default';
	http.createServer(function(req, res) {
		res.writeHead(200);
		res.end('process '+process.pid+' say hello:'+msg);
	}).listen(8000);

	process.on('message', function(message) {
		console.log('process on message:'+message);
		msg = message;
	});

	process.send('hello from '+process.pid);
}
</pre>

