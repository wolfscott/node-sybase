# node-sybase

A simple node.js wrapper around a Java application that provides easy access to Sybase databases via jconn3. The main goal is to allow easy installation without the requirements of installing and configuring odbc or freetds. You do however have to have java 1.5 or newer installed.

## Requirements

* java 1.5+

``` bash 
  c:\> java -version
```

## Install

### git

To manually build this package, you will of course need node-gyp installed, clone the repo and then perform the build.  If you desire this route of action you should be familiar with linking the result into your project.  

``` bash
git clone git://github.com/rodhoward/node-sybase.git
cd node-sybase
node-gyp configure build

```

### npm (recommended)

``` bash
npm install sybase
```

#### Example (see additional notes below)

``` javascript
var Sybase = require('sybase'),
    db     = new Sybase('host', port, 'dbName', 'username', 'pw');

db.connect(function (err) {
  if (err) return console.log(err);
  
  db.query('select * from user where user_id = 42', function (err, data) {
    if (err) console.log(err);

    console.log(data); // no data if err

    db.disconnect();

  });
});
```

A few items to mention that may be helpful. Some of these tripped me up at first:

* The host is the machine name only - this is NOT a connection string and no special characters required; e.g., NO // prefix  ('server' or '192.168.1.100' NOT '//server')
* port is likely 2638 for Sybase Anywhere 16/17 but I have seen 2640 listed.  Well, for me connecting to Sybase Anywhere 17, it was 2638  (Micros RES 3700)
* dbName is the instance name; for me it was at least
* username/pw well, ya - you got this one
* I had to specifically find the javaJarPath and point it as shown below and it took me a while to figure out that this jar was within THIS module and not some other dependency; I'm guessing other devs just "get it" but I struggled with this one - so now you know
* I received several odd errors while figuring all this out.  It appears the error messages are not very clear between what they are reporting and what I "thought" I was configuring. (understandable since this is just a wrapper layer, right?) For reference, during my search, there is a mention of "removing the lib folder below dist" in some docs out there which I totally didn't understand WHICH lib folder.  Was that dist/lib or was that the lib at the same level of dist.  (I still don't know BUT it started working when all my params above were correct - maybe it's just me but the params took me a few tries to get right until I got it straight in my head)
* While the example above shows connect, fetch, close in one sequence, for a real-world app, it would perhaps be better to connect and remain open until all the work is complete, then close (disconnect) - there is a status flag, isConnected, that helps know the state of your connection
* Kudos to the devs that made all of this happen - it's amazingly fast given what's happening under the hood - Thank YOU - it WORKS! yay!!! :)

### API Usage Example

The api is super simple. It makes use of standard node callbacks so that it can be easily used with promises. The only thing not covered in the example above is the option to print some timing stats to the console as well as to specify the location of the Java application bridge, which shouldn't need to change.

``` javascript
var logTiming = true,
  javaJarPath = './JavaSybaseLink/dist/JavaSybaseLink.jar',
  db = new Sybase('host', port, 'dbName', 'username', 'pw', logTiming, javaJarPath);
```

The java Bridge now optionally looks for a "sybaseConfig.properties" file in which you can configure jconnect properties to be included in the connection. This should allow setting properties like:

``` javascript
ENCRYPT_PASSWORD=true
```

> NOTE: if having an issue with the javaJarPath and receive an error like this:
  __Error: Unable to access jarfile ./JavaSybaseLink/dist/JavaSybaseLink.jar__
  you might consider defining the parameter shown below (note the different path) before calling the Sybase() method - that worked for me but I have a simple config so your mileage may vary;

``` javascript
  let logTiming = true;
  javaJarPath = './node_modules/sybase/JavaSybaseLink/dist/JavaSybaseLink.jar';
  db = new Sybase('host', port, 'dbName', 'username', 'pw', logTiming, javaJarPath);
```

### API Methods

* Sybase( host, port, dbName, uid, pwd, verbose, jarPath )
* connect()
* disconnect()
* isConnected()
* query( sql, callback )

## Revision History

### Version 1.2.2 - wjscott

* Updated some docs for my reference
* Modified the console.log()'s to only show when logTiming = true
* Added a disconnect if calling connect when already connected
* Broke my own rule of not changing code format, but added {} around some implied if's to aid in future updates or other contributors, jic - sorry Ron
* Updated a few typo's
* Not sure if it was just my editor or what but alignments were pretty "wonky" (the technical term) in places so violated my rule one more time and did a reformat. Guessing that was just an oversight before checkin : -(  the justification? Not likely this will be pushed back to the original codebase (ya, not off the hook that easy, eh? but I seriously don't reformat other code - lol -- seriously, I really don't)

### Version 1.2.1

* Original I forked
