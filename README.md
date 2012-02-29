Messenger.js - Fast Node.js Communication Library
============
Installation

    npm install messenger

What is Messenger.js?
------------------
Messenger.js is a library that makes network communication via JSON dead simple and insanely fast!

Example:
  
    var messenger = require('messenger');
  
    client = messenger.createSpeaker(8000);
    server = messenger.createListener(8000);
  
    server.on('give it to me', function(message, data){
      message.reply({'you':'got it'})
    });
  
    setInterval(function(){
      client.request('give it to me', {hello:'world'}, function(data){
        console.log(data);
      });
    }, 1000);

Output:
  
    > {'you':'got it'}
    > {'you':'got it'}
    > ...etc...

Features
--------
Messenger.js is very flexible and can handle everything you need.

- Supports Request / Reply Communication using round robin
- Supports Publish / Subscribe (fanout) Communication
- Supports Fire and Forget Communication
- Supports middleware plugin for messenger Listeners (servers)
- Extremely fast (disables TCP Nagle's algorithm)
- Fault tolerant: clients will reconnect to servers even if server goes down and comes back later
- Elegant API
- Easily involves multiple servers

Pub Sub Example
-------------

Example
  
    var messenger = require('messenger');
  
    // here we have 4 servers listening on 4 different ports
    var server1 = messenger.createListener(8001);
    var server2 = messenger.createListener(8002);
    var server3 = messenger.createListener(8003);
    var server4 = messenger.createListener('127.0.0.1:8004');

    server1.on('a message came', function(m, data){
      // note that m.data and data are equivalent
      console.log('server 1 got data', data);
    });
  
    server2.on('a message came', function(m, data){
      console.log('server 2 got data', data);
    });
  
    server3.on('a message came', function(m, data){
      console.log('server 3 got data', data);
    });
  
    server4.on('a message came', function(m, data){
      console.log('server 4 got data', data);
    });
  
    // a client that can be used to emit to all the servers
    var client = messenger.createSpeaker(8001, 8002, 8003, 8004);
  
    setInterval(function(){
      client.shout('a message came', {some: 'data});
    }, 1000);
  

Output

    > server 1 got some data
    > server 2 got some data
    > server 3 got some data
    > server 4 got some data
    > ... repeat ....

Multi-Server Request Reply
-------------

Example

    var messenger = require('messenger');

    // here we have 4 servers listening on 4 different ports
    var server1 = messenger.createListener(8001);
    var server2 = messenger.createListener(8002);
    var server3 = messenger.createListener(8003);
    var server4 = messenger.createListener('127.0.0.1:8004');

    server1.on('a message came', function(m, data){
      m.reply({greetings:'server 1 got some data'});
    });

    server2.on('a message came', function(){
      m.reply({greetings:'server 2 got some data'});
    });

    server3.on('a message came', function(){
      m.reply({greetings:'server 3 got some data'});
    });

    server4.on('a message came', function(){
      m.reply({greetings:'server 4 got some data'});
    });

    // a client that can be used to emit to all the servers
    var client = messenger.createSpeaker(8001, 8002, 8003, 8004);

    setTimeout(function(){
      // request here instead of shout to only access 1 server
      client.request('a message came', {some: 'data}, function(data){
        console.log(data.greetings);
      });
    }, 2000);


Output

    > server 1 got some data
    
    
Plugin (Middleware) Example
-------------

Example
    
    var messenger = require('messenger');
    
    var server = messenger.createListener(8000);
    var client = messenger.createClient(8000);
    
    function authRequired(m, data) {
      if (data.authorized) {
        m.next(); // continuation passing
        return;
      }
      m.reply({error:'not authorized'});
    }
    
    server.on('protected request', authRequired, function(m, data){
      m.reply({you:'got past security'})
    });
    
    var auth = false;
    setInterval(function(){
      
      client.request('protected request', {authorized:auth}, function(data){
        console.log(data);
      })
      
      if (auth === false) {
        auth = true;
      } else {
        auth = false;
      }
    }, 2000);
    
Output
    
    > {error: 'not authorized'}
    > {you:'got past security'}
    > {error: 'not authorized'}
    > {you:'got past security'}
    > ... etc ...