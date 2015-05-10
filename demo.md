## Preparation

# Server
## Create the project
```bash
npm install -g strongloop
slc loopback instagram-loopback
cd instagram-loopback
slc loopback:datasource mongo
slc loopback:datasource filestorage

npm install --save loopback-connector-mongodb
npm install --save loopback-component-storage

slc loopback:model Image title, geolocation, filename, date
slc loopback:model ImageContainer based on Model
```

Change filestorage to
```json
"filestorage": {
    "name": "filestorage",
    "connector": "loopback-component-storage",
    "provider": "filesystem",
    "root": "./storage"
  }
```
and create directory storage
```bash
mkdir storage
mkdir storage/instagram
```


Show Arc
```
slc arc
```

Generate client side
```
lb-ng server/server.js ./client/lbServices.js
```


#Client

## Scaffold the project
```bash
yo angular-famous-ionic --mobile
yo angular-famous-ionic:module common
yo angular-famous-ionic:contant loopbackConstant


bower install --save angular-resource
```

Update bower.json > GENERATOR
```json
"dependencies": {
    "angular": "1.3.15",
    "angular-mocks": "1.3.15",
    "angular-animate": "1.3.15",
    "angular-sanitize": "1.3.15",
    "angular-ui-router": "0.2.13",
    "collide": "1.0.0-beta.3",
    "ionic": "1.0.0-rc.5",
    "ngCordova": "0.1.15-alpha",
    "font-awesome": "4.2.0",
    "angular-resource": "1.3.15"
  },
  "resolutions": {
    "angular": "1.3.15",
    "angular-animate": "1.3.15",
    "angular-sanitize": "1.3.15"
  }
```

Add angular-resource and lbServices to `package.json`  GENERATOR

```json
"lbServices": "./client/scripts/lbServices.js"
"angular-resource": "./bower_components/angular-resource/angular-resource.js",
   
...

"angular-resource": {
      "exports": "angular",
      "depends": [
        "angular"
      ]
    },
    "lbServices": {
      "depends": [
        "angular-resource"
      ]
    },

```



### Show the project
```
gulp help
gulp lint
gulp karma
````


### Change home.html
```html
<ion-view>

    <ion-header-bar class="bar-positive" align-title="center">
        <h1 class="title">{{vm.title}}</h1>
    </ion-header-bar>

    <ion-header-bar class="bar-subheader bar-stable item-input-inset" align-title="center">
        <label class="item-input-wrapper">
            <input type="text" name="pictureTitle" placeholder="Your picture title" ng-model="vm.pictureTitle" required>
        </label>
        <button class="button button-small button-positive button-outline" ng-click="vm.takePicture($event)">
            <i class="icon ion-camera"></i>
        </button>
    </ion-header-bar>

    <ion-content class="has-subheader">

        <div ng-repeat="image in vm.images">

            <div class="list card">

                <div class="item item-avatar">
                    <img src="mcfly.jpg">
                    <h2>Marty McFly</h2>
                    <p>November 05, 1955</p>
                </div>

                <div class="item item-body">
                    <img class="full-image" src="delorean.jpg">
                    <p>
                    </p>
                    <p>
                        <a href="#" class="subdued">1 Like</a>
                        <a href="#" class="subdued">5 Comments</a>
                    </p>
                </div>
            </div>

        </div>

    </ion-content>
</ion-view>

```

### Copy lbServices.js
```
cp client/lbServices.js ../instagram-app/client/scripts/

```
Launch
```
gulp browsersync
```

### Change loopbackConstants.js
```js
'use strict';
var constantname = 'loopbackConstant';

module.exports = function(app) {
    app.constant(app.name + '.' + constantname, {

        baseUrl : 'http://localhost:3000/api',
        container : 'instagram'
    });
};
```


### Add home controller and modify config
```js
app.config(['LoopBackResourceProvider', fullname + '.loopbackConstant', function(LoopBackResourceProvider, loopbackConstant) {
        // Change the URL where to access the LoopBack REST API server
        LoopBackResourceProvider.setUrlBase(loopbackConstant.baseUrl);
    }]);
```

### Build ios and android
```bash
gulp cordova:all  MODIFY GENERATOR generator-sublime
```

### Modify home.js
```js
title

```

