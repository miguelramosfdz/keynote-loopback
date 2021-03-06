## Preparation

# Server
## Create the project
```bash
ulimit -n 2048
npm install -g strongloop
slc loopback instagram-loopback
cd instagram-loopback

git init

slc loopback:datasource mongo
slc loopback:datasource filestorage

npm install --save loopback-connector-mongodb loopback-component-storage
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

Add Procfile
```
echo "web: node server/server.js" > Procfile
```

Create the configuration for heroku with mongo   
Add a new file `datasources.heroku.js`
```
touch server/datasources.heroku.js
```

```js
'use strict';

module.exports = {
    mongo: {
        connector: 'mongodb',
        url: process.env.MONGOHQ_URL
    }
};
```


and create directory storage

```bash
mkdir -p storage/ngconf && touch storage/ngconf/README.md

slc loopback:model Image title, geolocation, filename, created_at, updated_at  -> MAKE SURE DATABASE is mongo
slc loopback:model ImageContainer based on Model -> MAKE SURE DATABASE is filestorage
```

Change the datasource to mongo


Add operation hook to Image
```js
'use strict';

module.exports = function(Image) {

    Image.observe('before save', function(ctx, next) {
        var data;
        if(ctx.instance) {
            data = ctx.instance;
        } else {
            data = ctx.data;
        }
        var currDate = new Date();
        if(ctx.isNewInstance) {
            data.created_at = currDate;
        }
        data.updated_at = currDate;
        next();
    });
};
```

Show Arc
```
slc arc
```


Create a mongolab database 
thaiat/***   
test/test   




navigate to the heroku app `heroku open`

Generate client side
```
lb-ng server/server.js ./client/lbServices.js
```

Publish to heroku
```bash
git cm "feat(app): Initial repo"
git remote add heroku https://git.heroku.com/ngconf.git
heroku create ngconf -> START WITH 1 DYNO
heroku config:set MONGOHQ_URL=mongodb://test:test@ds031792.mongolab.com:31792/ngconf
heroku config:set NODE_ENV=heroku
git push heroku
```


#Client

## Scaffold the project
```bash
yo mcfly --mobile
yo mcfly:module common
yo mcfly:constant common loopbackConstant
yo mcfly:controller common home
npm install --save mcfly-angular angular-moment
```

### Copy lbServices.js
```
cp ../ngconf-loopback/client/lbServices.js client/scripts
```

### Show the project
```
gulp help
gulp lint
gulp karma
````


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

        baseUrl : 'https://ngconf.herokuapp.com/api',
        container : 'ngconf'
    });
};
```

### Change index.js
```js

require('lbServices');
require('angular-moment');
var mcflyUI = require('mcfly-angular').ui;

['lbServices', mcflyUI.name, 'angularMoment']


controller : fullname + '.home as vm'

app.config(['LoopBackResourceProvider', fullname + '.loopbackConstant', function(LoopBackResourceProvider, loopbackConstant) {
        // Change the URL where to access the LoopBack REST API server
        LoopBackResourceProvider.setUrlBase(loopbackConstant.baseUrl);
    }]);
```


### Change home.html
```html
<ion-view>

    <ion-header-bar class="bar-positive" align-title="center">
        <h1 class="title">{{vm.title}}
            <span class="badge badge-assertive">{{vm.count}}</span>
        </h1>
    </ion-header-bar>

    <ion-header-bar class="bar-subheader bar-stable item-input-inset" align-title="center">
        <form style="width:90%">
            <label class="item-input-wrapper">
                <input type="text" name="pictureTitle" placeholder="Your picture title" ng-model="vm.pictureTitle" required>
            </label>
        </form>
        <div style="margin-left:10px" ng-disabled="!vm.pictureTitle" class="button button-small button-positive button-outline" mcf-image-capture handler="vm.captureHandler" ng-click="vm.doCapture()">
            <i class="icon ion-camera"></i>
        </div>
    </ion-header-bar>

    <ion-content class="has-subheader">
        <yoo-refresher on-refresh="vm.loadImages()"></yoo-refresher>
        <div ng-repeat="image in vm.images">

            <div class="list card">

                <div class="item item-avatar">
                    <img ng-src="{{image.src}}">
                    <h2>{{image.title}}</h2>
                    <p><span am-time-ago="image.created_at"></span></p>
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

main.scss
```css
form.ng-invalid > .item-input-wrapper {
    border: 2px red dashed;
}
```





### Modify home.js
```js
'use strict';
var controllername = 'home';

module.exports = function(app) {
    /*jshint validthis: true */

    var deps = [app.name + '.loopbackConstant', 'Image', 'ImageContainer', 'mcfly.angular.ui.fileStorageLoopback'];

    function controller(loopbackConstant, Image, ImageContainer, fileStorageLoopback) {
        var vm = this;
        vm.title = 'Share some moments';

        var activate = function() {
            vm.loadImages();
        };

        vm.loadImages = function() {
            return Image.find({
                    filter: {
                        order: 'created_at DESC'
                    }
                })
                .$promise
                .then(function(images) {
                    images.forEach(function(image) {
                        image.src = fileStorageLoopback.download(loopbackConstant.baseUrl, 'ImageContainers', loopbackConstant.container, image.filename);
                    });
                    return images;
                })
                .then(function(images) {
                    vm.images = images;
                    vm.count = images.length;
                });
        };
        vm.doCapture = function() {
            vm.captureHandler()
                .then(function(res) {
                    var filedata = res.filedata;
                    var filename = res.filename;
                    return fileStorageLoopback.upload(loopbackConstant.baseUrl, 'ImageContainers', loopbackConstant.container, filename, filedata);

                })
                .then(function(uploadedFile) {
                    return Image.create({
                        filename: uploadedFile.name,
                        title: vm.pictureTitle
                    }).$promise;
                })
                .then(function() {
                    vm.loadImages();
                });
        };

        activate();

    }

    controller.$inject = deps;
    app.controller(app.name + '.' + controllername, controller);
};

```


### Build ios and android
```bash
gulp cordova:all 
```


## TestFairy
Add api upload key : `7422cf17c862ecca807a817e2c1f2c06567cf62a` to `gulp_tasks/common/constants.js`

Add plugins in hook
```
'org.apache.cordova.device',
'org.apache.cordova.device-motion',
'org.apache.cordova.device-orientation',
'org.apache.cordova.geolocation',
'org.apache.cordova.console',
'org.apache.cordova.file-transfer',
'org.apache.cordova.camera',
'org.apache.cordova.media-capture',
'org.apache.cordova.geolocation',
'org.apache.cordova.splashscreen',
'org.apache.cordova.statusbar',
'org.apache.cordova.globalization',
'com.ionic.keyboard',
'https://github.com/testfairy/testfairy-cordova-plugin'

```

Add testfairy initialization in main.js
```js
if($window.TestFairy) {
    $window.TestFairy.begin('9d85ea005720f0b65a824114d53b0bce5a958581');
}
```

gulp cordova:all

gulp cordova:testfairy


