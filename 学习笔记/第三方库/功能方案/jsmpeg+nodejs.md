## jsmpeg

1、编译

[jsmpeg]()

    查看build.sh文件显示需要emsdk版本
    # ./emsdk install 1.38.47
    # ./emsdk activate 1.38.47

## emsdk

[emsdk]()

按照安装方法下载更新指定版本。（必须安装指定版本，最新版本的emsdk在web端加载时会报错）

## 使用

  - 1. 运行emsdk环境变量： source emsdk_env.sh
  - 2. 启动websocket服务：  node websocket-relay.js supersecret 8081 8082
  - 3. 启动ffmpeg转发： ffmpeg -i "rtsp://192.168.1.20:8550/streamch0" -q 0 -f mpegts -codec:v mpeg1video -s 1366x768 http://127.0.0.1:8081/supersecret

## websocket 说明
<details>
<summary>websocket-reply.sj 修改</summary>
~~~js
// Use the websocket-relay to serve a raw MPEG-TS over WebSockets. You can use
// ffmpeg to feed the relay. ffmpeg -> websocket-relay -> browser
// Example:
// node websocket-relay yoursecret 8081 8082
// ffmpeg -i <some input> -f mpegts http://localhost:8081/yoursecret

var fs = require('fs'),
	http = require('http'),
	WebSocket = require('ws');

if (process.argv.length < 5) {
	console.log(
		'Usage: \n' +
		'node websocket-relay.js <secret> [<stream-port> <infrared-stream-port> <websocket-port> <websocket-port>]'
	);
	process.exit();
}

var STREAM_SECRET = process.argv[2],
	STREAM_PORT = process.argv[3] || 8081,
	INFRARED_STREAM_PORT = process.argv[4] || 8082,
	WEBSOCKET_PORT = process.argv[5] || 8083,
	INFRARED_WEBSOCKET_PORT = process.argv[6] || 8084,
	RECORD_STREAM = false;

// Websocket Server
var socketServer = new WebSocket.Server({port: WEBSOCKET_PORT, perMessageDeflate: false});
socketServer.connectionCount = 0;
socketServer.on('connection', function(socket, upgradeReq) {
	socketServer.connectionCount++;
	console.log(
		'New WebSocket Connection: ',
		(upgradeReq || socket.upgradeReq).socket.remoteAddress,
		(upgradeReq || socket.upgradeReq).headers['user-agent'],
		'('+socketServer.connectionCount+' total)'
	);
	socket.on('close', function(code, message){
		socketServer.connectionCount--;
		console.log(
			'Disconnected WebSocket ('+socketServer.connectionCount+' total)'
		);
	});
});
socketServer.broadcast = function(data) {
	socketServer.clients.forEach(function each(client) {
		if (client.readyState === WebSocket.OPEN) {
			client.send(data);
		}
	});
};

// HTTP Server to accept incomming MPEG-TS Stream from ffmpeg
var streamServer = http.createServer( function(request, response) {
	var params = request.url.substr(1).split('/');

	if (params[0] !== STREAM_SECRET) {
		console.log(
			'Failed Stream Connection: '+ request.socket.remoteAddress + ':' +
			request.socket.remotePort + ' - wrong secret.'
		);
		response.end();
	}

	response.connection.setTimeout(0);
	console.log(
		'Stream Connected: ' +
		request.socket.remoteAddress + ':' +
		request.socket.remotePort
	);
	request.on('data', function(data){
		socketServer.broadcast(data);
		if (request.socket.recording) {
			request.socket.recording.write(data);
		}
	});
	request.on('end',function(){
		console.log('close');
		if (request.socket.recording) {
			request.socket.recording.close();
		}
	});

	// Record the stream to a local file?
	if (RECORD_STREAM) {
		var path = 'recordings/' + Date.now() + '.ts';
		request.socket.recording = fs.createWriteStream(path);
	}
})
// Keep the socket open for streaming
streamServer.headersTimeout = 0;
streamServer.listen(STREAM_PORT);

/*
 * for infrared streamch1
 */
// Websocket Server
var socketServerInfrared = new WebSocket.Server({port: INFRARED_WEBSOCKET_PORT, perMessageDeflate: false});
socketServerInfrared.connectionCount = 0;
socketServerInfrared.on('connection', function(socket, upgradeReq) {
	socketServerInfrared.connectionCount++;
	console.log(
		'[Infrared]New WebSocket Connection: ',
		(upgradeReq || socket.upgradeReq).socket.remoteAddress,
		(upgradeReq || socket.upgradeReq).headers['user-agent'],
		'('+socketServerInfrared.connectionCount+' total)'
	);
	socket.on('close', function(code, message){
		socketServerInfrared.connectionCount--;
		console.log(
			'[Infrared]Disconnected WebSocket ('+socketServerInfrared.connectionCount+' total)'
		);
	});
});
socketServerInfrared.broadcast = function(data) {
	socketServerInfrared.clients.forEach(function each(client) {
		if (client.readyState === WebSocket.OPEN) {
			client.send(data);
		}
	});
};

// HTTP Server to accept incomming MPEG-TS Stream from ffmpeg
var streamServerInfrared = http.createServer( function(request, response) {
	var params = request.url.substr(1).split('/');

	if (params[0] !== STREAM_SECRET) {
		console.log(
			'Failed Stream Connection: '+ request.socket.remoteAddress + ':' +
			request.socket.remotePort + ' - wrong secret.'
		);
		response.end();
	}

	response.connection.setTimeout(0);
	console.log(
		'[Infrared]Stream Connected: ' +
		request.socket.remoteAddress + ':' +
		request.socket.remotePort
	);
	request.on('data', function(data){
		socketServerInfrared.broadcast(data);
		if (request.socket.recording) {
			request.socket.recording.write(data);
		}
	});
	request.on('end',function(){
		console.log('[Infrared]close');
		if (request.socket.recording) {
			request.socket.recording.close();
		}
	});

	// Record the stream to a local file?
	if (RECORD_STREAM) {
		var path = 'recordings/' + Date.now() + '.ts';
		request.socket.recording = fs.createWriteStream(path);
	}
})

</details>

// Keep the socket open for streaming
streamServerInfrared.headersTimeout = 0;
streamServerInfrared.listen(INFRARED_STREAM_PORT);

console.log('Listening for incomming MPEG-TS Stream on http://127.0.0.1:'+STREAM_PORT+'/<secret>');
console.log('Listening for incomming infrared MPEG-TS Stream on http://127.0.0.1:'+INFRARED_STREAM_PORT+'/<secret>');
console.log('Awaiting WebSocket connections on ws://127.0.0.1:'+WEBSOCKET_PORT+'/');
console.log('Awaiting Infrared WebSocket connections on ws://127.0.0.1:'+INFRARED_WEBSOCKET_PORT+'/');
~~~
因为需要同时支持两路rtsp视频，所有参考原来的代码，增加了一个红外的通信和数据流端口

## 移植到his3559a
因为jsmpeg编译后是一个js文件，所以不需要交叉编译就能运行。唯一需要移植的是nodejs解析器。参考jsmpeg下载的nodejs版本，下载对应的源码进行编译，这里是node-14.15.5

### nodejs安装
url： []()

**1、configure**

导入环境变量
~~~shell
#!/bin/sh
# Native Compiler
export AR_host="ar"
export CC_host="gcc"
export CXX_host="g++"
export LINK_host="g++"


# Allwinner H8 CQA83t cross compiler
export ARCH=arm64
export CROSS_COMPILE=aarch64-himix100-linux-
export CC=aarch64-himix100-linux-gcc   
export CXX=aarch64-himix100-linux-g++    
export LD=aarch64-himix100-linux-ld
export AR=aarch64-himix100-linux-ar
export AS=aarch64-himix100-linux-as
export RANLIB=aarch64-himix100-linux-ranlib
~~~

**<font color=red>ARCH</font>** 变量需要制定 arm64， 而不是arm。否则编译会出现
~~~
#error target architecture arm is only supported on arm and ia32 host
~~~

**2、编译**

~~~
CC=aarch64-himix100-linux-gcc CXX=aarch64-himix100-linux-g++ CC_host="gcc" CXX_host="g++" ./configure --prefix=$PWD/../node-14.15.5 --dest-cpu=arm64 --cross-compiling --dest-os=linux --with-arm-float-abi=hard
make -j8
make install
~~~

编译需要很久，如果编译环境内存较小，可能会报内存不够的错误。建议调整到8G内存

### 运行环境

~~~shell
1、将jsmpeg-master拷贝到嵌入式环境中。目录结构如下
    jsmpeg.min.js
    node                       
    node_modules               
    view-stream-infrared.html  view-stream.html           
    websocket-relay.js
    其中node_modules里面包含ws的库
2、运行环境变量
    export EMSDK_NODE=/app/private/jsmpeg-master
~~~
