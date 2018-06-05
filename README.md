# Vector tiles production
## What do we do in this hands-on?

1. Extract data from [PostGIS](https://postgis.net/) to [NDJSON](http://ndjson.org/) of GeoJSON. Modify features to remove unnecessary properties. Add production control properties.
2. Get vector tiles from NDJSON using [tippecanoe](https://github.com/mapbox/tippecanoe).
3. Check resulting vector tiles using [mc](https://github.com/hfu/mc/) and [tileserver-gl-light](https://github.com/klokantech/tileserver-gl/blob/master/README_light.md). Checking is an important part of optimization. 

After this hands-on, vector tiles are ready for hosting and consumption.

# Software

## Recommended baseline software

### Linux
This hands-on assumes Linux such as ubuntu or RedHat as the operating system. 

Yet, we also used macOS. Cygwin or Windows Subsystem for Linux might also be good.

### Google Chrome
Vector tiles works well with all modern web browsers. But Google Chrome has Chrome Secure Shell extension. This extension provides ssh access to host.

## Required software

### tippecanoe

You can install from [GitHub](https://github.com/mapbox/tippecanoe).

### Node.js

You can install from [nodejs.org](https://nodejs.org/ja/download/package-manager/#debian-and-ubuntu-based-linux-distributions-debian-ubuntu-linux)

Or you may want to use [n](https://github.com/tj/n) like this:
```
npm install -y nodejs
npm cache clean
npm install -g n
npm cache clean
n stable
ln -sf /usr/local/bin/node /usr/bin/node
apt-get -y purge npm nodejs
```

### git

Linux often contains [git](https://git-scm.com/) from the beginning.

# Hands-on steps

## Software check
### check node

```console
$ node -v
v8.11.2
```

### check tileserver-gl-light

```console
$ tileserver-gl-light -v
tileserver-gl-light v2.3.1
```

### check tippecanoe

```console
$ tippecanoe -v
tippecanoe v1.28.1
```
## Produce vector tiles
### clone pnd.js using git

```console
$ git clone git@github.com:hfu/pnd.git
Initialized empty Git repository in /home/fhidenori/pnd/.git/
remote: Counting objects: 52, done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 52 (delta 29), reused 35 (delta 14), pack-reused 0
Receiving objects: 100% (52/52), 21.06 KiB, done.
Resolving deltas: 100% (29/29), done.
```

### install npm modules

```console
$ cd pnd
$ npm install
added 166 packages from 81 contributors in 5.574s
```

### prepare a config file (config/default.hjson)

```console
$ mkdir config
$ vi config/default.hjson
```

You can also use nano or emacs. You need to enter configuration to config/default.hjson like below.  [hjson](http://hjson.org/) is human interface to JSON. You can skip quotes and commas, and also can comment out.  If you want to write the configuration in JSON, you can create config/default.json. 
```hjson
{
  host: postgis.host.name
  user: username
  password: secretpassword
  modules: [[5, 18, 15]]
  geom: { // geometric field name for each database
    database1: geom
    database2: geom
  }
  data: {
    database1: [
      custom_planet_ocean
      // ...
    ]
    database2: [
      osm_planet_water
      osm_planet_major_roads
      osm_planet_places
      // ...
    ]
  }
}
```
Your hands-on tutor may provide the file.

_modules_ contains [z, x, y] of area you want to create vector tiles. [module-locate](https://hfu.github.io/module-locate/#5/8.000/25.000) is a small site to show which module is where. 

_data_ contains the names of databases and relations. If you want to check databases and relations of your source DB, 
 [schema-check](https://github.com/hfu/schema-check) might be helpful.

### prepare modify.js
modify.js contains a function to filter a GeoJSON feature. We documented the specification of modify.js at [modify-spec](https://hfu.github.io/modify-spec/)

```console
$ vi modify.js
```

Below is an example of modify.js

```javascript
module.exports = (f) => {
  switch (f.properties._layer) {
    case 'custom_planet_ocean_l08':
    case 'custom_planet_land_a_l08':
      f.tippecanoe.minzoom = 0
      f.tippecanoe.maxzoom = 6
      f.properties = {}
      break
    // ...
  }
  return f
}
```

 Your hands-on tutor may provide the file. 

We will not cover the detailed meaning of this file. You can see [RFC 7946: The GeoJSON Format](https://tools.ietf.org/html/rfc7946) and [tippecanoe's GeoJSON Extension](https://github.com/mapbox/tippecanoe#geojson-extension). 

### create vector tiles
You prepared config/default.hjson and modify.js for your data. Next, you can create vector tiles!
```console
 $ node pnd.js
importing #1 8-150-124
started a tippecanoe process: 1 of 1 active, 0 in queue.
For layer 0, using name "8150124ndjson"
288854 features, 11519390 bytes of geometry, 1280 bytes of separate metadata, 5806 bytes of string pool
  99.9%  16/38517/31873  
8-150-124.ndjson finished. 0 active, 0 in queue.
```
