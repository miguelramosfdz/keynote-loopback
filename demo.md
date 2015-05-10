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

Add angular-resource and lbServices to `package.json`

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


### change home.html
```html
<ion-view>
    <ion-header-bar align-title="center" class="bar-positive">

        <h1 class="title">{{vm.message}}</h1>

        <div class="buttons">
            <button class="button"><i class="icon ion-camera"></i></button>
        </div>
    </ion-header-bar>
    <ion-content>
        <h1>Content goes here</h1>

        <div ng-repeat="image in vm.images">{{image.title}}</div>
    </ion-content>
</ion-view>

```

### Copy lbservices
```
cp client/lbServices.js ../instagram-app/client/scripts/

```
Launch
```
gulp browsersync
```

