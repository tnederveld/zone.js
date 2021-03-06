# Zone.js's support for non standard apis

Zone.js patched most standard APIs so they can be in zone. such as DOM events listener, XMLHttpRequest in Browser
 and EventEmitter, fs API in nodejs. 
  
But there are still a lot of non standard APIs are not patched by default, such as MediaQuery, Notification, 
 WebAudio and so on. We are adding the support to those APIs, and the progress will be updated here.
 
## Currently supported non standard Web APIs 

* MediaQuery
* Notification 

## Currently supported polyfills

* webcomponents

Usage:

```
<script src="webcomponents-lite.js"></script>
<script src="node_modules/zone.js/dist/zone.js"></script>
<script src="node_modules/zone.js/dist/webapis-shadydom.js"></script>
```

## Currently supported non standard node APIs

## Currently supported non standard common APIs

* bluebird promise

Browser Usage: 

```
  <script src="zone.js"></script>
  <script src="bluebird.js"></script>
  <script src="zone-bluebird.js"></script>
  <script>
  Zone[Zone['__symbol__']('bluebird')](Promise);
  </script>
```

After those steps, window.Promise will become a ZoneAware Bluebird Promise.

Node Sample Usage:

```
require('zone.js');
const Bluebird = require('bluebird');
require('zone.js/dist/zone-bluebird');
Zone[Zone['__symbol__']('bluebird')](Bluebird);
Zone.current.fork({
  name: 'bluebird'
}).run(() => {
  Bluebird.resolve(1).then(r => {
    console.log('result ', r, 'Zone', Zone.current.name);
  });
});
```

In NodeJS environment, you can choose to use Bluebird Promise as global.Promise
or use ZoneAwarePromise as global.Promise.

To run the jasmine test cases of bluebird

```
  npm install bluebird
```

then modify test/node_tests.ts
remove the comment of the following line

```
//import './extra/bluebird.spec';
```

## Others

* Cordova 

patch `cordova.exec` API

`cordova.exec(success, error, service, action, args);` 

`success` and `error` will be patched with `Zone.wrap`.

to load the patch, you should load in the following order.

```
<script src="zone.js"></script>
<script src="cordova.js"></script>
<script src="zone-patch-cordova.js"></script>
```

## Usage

By default, those APIs' support will not be loaded in zone.js or zone-node.js,
so if you want to load those API's support, you should load those files by yourself

for example, if you want to add MediaQuery patch, you should do like this. 

```
  <script src="path/zone.js"></script> 
  <script src="path/webapis-media-query.js"></script> 
```  

* rxjs

`zone.js` also provide a `rxjs` patch to make sure rxjs Observable/Subscription/Operator run in correct zone.
for detail please refer to [pull request 843](https://github.com/angular/zone.js/pull/843), the following sample code describe the idea.

```
const constructorZone = Zone.current.fork({name: 'constructor'});
const subscriptionZone = Zone.current.fork({name: 'subscription'});
const operatorZone = Zone.current.fork({name: 'operator'});

let observable;
let subscriber;
constructorZone.run(() => {
  observable = new Observable((_subscriber) => {
    subscriber = _subscriber;
    console.log('current zone when construct observable:', Zone.current.name); // will output constructor.
    return () => {
      console.log('current zone when unsubscribe observable:', Zone.current.name); // will output constructor.
    }
  });
});

subscriptionZone.run(() => {
  observable.subscribe(() => {
    console.log('current zone when subscription next', Zone.current.name); // will output subscription. 
  }, () => {
    console.log('current zone when subscription error', Zone.current.name); // will output subscription. 
  }, () => {
    console.log('current zone when subscription complete', Zone.current.name); // will output subscription. 
  });
});

operatorZone.run(() => {
  observable.map(() => {
    console.log('current zone when map operator', Zone.current.name); // will output operator. 
  });
});
```

currently basically all `rxjs` API include

- Observable
- Subscription
- Subscriber
- Operators 
- Scheduler 

are patched, so they will run in the correct zone.

## Usage.

for example, in angular application, you can load this patch in your `app.module.ts`.

```
import 'zone.js/dist/zone-patch-rxjs';
```

