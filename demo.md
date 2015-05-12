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
        <div class="button button-small button-positive button-outline" image-capture handler="vm.test" ng-click="vm.mycapture()">
            <i class="icon ion-camera"></i>
        </div>
    </ion-header-bar>

    <ion-content class="has-subheader">

        <div ng-repeat="image in vm.images">

            <div class="list card">

                <div class="item item-avatar">
                    <img ng-src="{{image.src}}">
                    <h2>{{image.title}}</h2>
                    <p>November 05, 1955</p>
                </div>

                <div class="item item-body">
                    <img class="full-image" ng-src="{{image.src}}">
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
'use strict';
var controllername = 'home';

module.exports = function(app) {
    /*jshint validthis: true */

    var angular = require('angular');

    var deps = [app.name + '.loopbackConstant', 'Image', 'ImageContainer', app.namespace.yoobicUI + '.fileUploadLoopback'];

    function controller(loopbackConstant, Image, ImageContainer, fileUploadLoopback) {
        var vm = this;
        vm.title = 'Share some moments';
        var activate = function() {
            Image.find().$promise.then(function(images) {
                images.forEach(function(image) {
                    image.src = loopbackConstant.baseUrl + '/' + 'ImageContainers' + '/' + 'instagram' + '/download/' + image.filename;
                });
                vm.images = images;
                console.log(images);
            });
        };
        activate();

        vm.mycapture = function() {
            vm.test()
                .then(function(res) {
                    var filedata = res.filedata;
                    var filename = res.filename;
                    return fileUploadLoopback.upload(loopbackConstant.baseUrl, 'ImageContainers', loopbackConstant.container, filename, filedata);

                }, function(err) {
                    alert('err:' + err);
                })
                .then(function(uploadedFile) {
                    return Image.create({
                        filename: uploadedFile.name,
                        title: vm.pictureTitle
                    }).$promise;
                })
                .then(function() {
                    activate();
                });
        };

    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};


```

