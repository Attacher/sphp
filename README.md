##Snappy PHP for node js
A new snappy PHP execution module / middleware 

Features
* Middleware for node Express
* Made for fast response time on PHP requests
* Supports Websocket requests served by PHP scripts
* Transfer of node session to PHP session
* No dependencies (except for example)
* Mimic of Apache mod_php population of $_SERVER
* Highly configurable.

Note:
* File upload disabled at present.

####Install

    npm install sphp


####Use with express

    var express = require('express');
    var sphp = require('sphp');
    var app = express();
    var server = app.listen(8080,'0.0.0.0');
    
    // Attach PHP and static file server
    app.use(sphp.express('public/'));
    app.use(express.static('public/'));

####Use with ws (Websockets)

    var express = require('express');
    var sessionStore = new expressSession.MemoryStore();
    var sphp = require('sphp');
    
    var app = express();
    var server = app.listen(8080);
    var ws = new require('ws').Server({server: server});
    
    // Attach PHP, websocket and static file server
    app.use(sphp.express('public/'));
    ws.on('connection',sphp.websocket(sessionOptions));
    app.use(express.static('public/'));
    
####Use with express-session

    var express = require('express');
    var expressSession = require('express-session');
    var bodyParser = require('body-parser');
    var sessionStore = new expressSession.MemoryStore();
    var sphp = require('sphp');
    
    var app = express();
    var server = app.listen(8080);
    var sessionOptions={
         store: sessionStore
        ,secret:'yes :c)'
        ,resave:false
        ,saveUninitialized:false
        ,rolling: true
        ,name: 'SID'
    }

    app.use(expressSession(sessionOptions));
    // Save session 
    app.use(function(request, response, next){ 
      request.session.ip=request.client.remoteAddress;
      next();
    });
    app.use(bodyParser.json());       // to support JSON-encoded bodies
    app.use(bodyParser.urlencoded({extended: true})); // to support URL-encoded bodies

    // Attach PHP, websocket and static file server
    app.use(sphp.express('example/'));
    ws.on('connection',sphp.websocket(sessionOptions));
    app.use(express.static('example/'));

####Configuration
#####cgiEngine (Default: php-cgi)
Specify wich binary file to use to execute PHP script. The executable must be in the enviroment PATH or use a full path to the executable file.

    sphp.cgiEngine='php';

#####docRoot (default: ./public)
Where to serve script files from. Might be relative or an absolute path. This is the variable set, when sphp.express is called with a parameter.

    sphp.docRoot='./my_files';

#####minSpareWorkers (Default: 2)
Define the minimum number of workers kept ready. 
Note that when caling PHP scripts through websockets, an aditionan concurent worker is used. 

    sphp.minSpareWorkers=4;

#####maxWorkers (Default: 10)
The maximum number of workers allowed to start. This number will never be exceded. Request will be rejected.
Set this to limit the amount of RAM the server can use, when load is high. The foodprint is about 20MB / worker as of php-cgi 5.4 php-gci

    sphp.maxWorkers=20;

#####stepDowntime (Default: 360 seconds)
The number of worker is increased dynamicaly, When the need arises. This is time it takes before the number of workers, are reduced by one.

    sphp.stepDowntime=600;


####Notes
The goal of this project is to serve PHP scripts with the best response time possible, while using nodes session control. 
It is achieved by pre-emptively spawning and loading of the PHP-CGI engine and holdning it there, until nedded.

