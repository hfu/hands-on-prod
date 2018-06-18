# Vector tiles production
## What do we do in this hands-on?

First, we extract geospatial data from [PostGIS](https://postgis.net/) to [NDJSON](http://ndjson.org/) of GeoJSON. Embedded in this process, we modify features to remove unnecessary properties and add production control properties such as vector tile layer name and minimum/maximum zoom levels. We also split data into modules each of which convers the spatial extent of the tile at zoom level 5. We call it z=5 modules. Embedding these steps in streaming geospatial data from PostGIS is a way to keep the production time short and reduce the amount of storage to be used.

Second, we get vector tiles from NDJSON using [tippecanoe](https://github.com/mapbox/tippecanoe). We borrow the power of tippecanoe, an open-source C++ implementation actively developed for a long time.

Last, check produced vector tiles. We check the size distribution of vector tiles using [mbq](https://github.com/hfu/mbq/). This is a quantitative indicator to examine the performance of vector tiles. In the lean vector tiles project, we mainly use this qmatrix to optimize the size distrubution of vector tiles. In addition, we shortly introduce [mbstat](https://github.com/hfu/mbstat) which checks statistics of filtered vector tiles. At last, we visual-check produced vector tiles using [tileserver-gl-light](https://github.com/klokantech/tileserver-gl/blob/master/README_light.md).

After this hands-on, vector tiles are ready for [hosting](https://github.com/hfu/hands-on-host) and consumption.

# Software

## Recommended baseline software

### Operating System: Linux
As of now, his hands-on assumes Linux such as ubuntu or RedHat as the operating system. 

Yet, we also used macOS for vector tile production and it was successful. Cygwin or Windows Subsystem for Linux might also be good, but we haven't tested yet.

### Web Browser: Google Chrome
Vector tiles works well with all modern web browsers. But if you still have options to choose from, we would like to recommend Google Chrome.

The advantage of using Google Chrome in this hands-on is not only it is a good modern browser. Google Chrome has [Chrome Secure Shell extension](https://chrome.google.com/webstore/detail/secure-shell-extension/iodihamcpbpeioajjeobimgagajmlibd). This extension provides ssh access to host.

## Required software

### tippecanoe
Tippecanoe is the software to build vector tilesets from large collections of GeoJSON features.

If you need to install tippecanoe, you can install it from [GitHub](https://github.com/mapbox/tippecanoe). 

### Node.js
Node.js is an asynchronous event driven JavaScript runtime. 

You can install  from [nodejs.org](https://nodejs.org/ja/download/package-manager/#debian-and-ubuntu-based-linux-distributions-debian-ubuntu-linux). Or you may want to use [n](https://github.com/tj/n) like this:
```
npm install -y nodejs
npm cache clean
npm install -g n
npm cache clean
n stable
ln -sf /usr/local/bin/node /usr/bin/node
apt-get -y purge npm nodejs
```

### tileserver-gl-light
We use a Node.js module called tileserver-gl-light to test the vector tiles. If it is not yet installed, you can install it like this:
```
npm install -g tileserver-gl-light
```

### git

Linux often contains [git](https://git-scm.com/) from the beginning.

# Hands-on steps

## Software check

### check git

```console
$ git --version
git version 1.7.1
```

### check node

```console
$ node -v
v9.9.0
```

### check tileserver-gl-light

```console
$ tileserver-gl-light -v
tileserver-gl-light v2.3.1
```

### check tippecanoe

```console
$ tippecanoe -v
tippecanoe v1.27.16
```

## Produce vector tiles
### clone pnd.js using git

```console
$ git clone https://github.com/hfu/pnd.git
Initialized empty Git repository in /home/user1/pnd/.git/
remote: Counting objects: 79, done.
remote: Compressing objects: 100% (58/58), done.
remote: Total 79 (delta 45), reused 52 (delta 20), pack-reused 0
Unpacking objects: 100% (79/79), done.
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
As you might see, pnd.js called tippecanoe internally. 

Now you have vector tiles in .mbtiles!
```console
$ ls *.mbtiles
8-150-124.mbtiles
```

## Check vector tiles

### check the source NDJSON
Any pager or good editor can see the content of a .ndjson file.
```console
$ more 8-150-124.ndjson
{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[32.34375,5.615985819155343],[32.34375,4.2
14943141390654],[30.9375,4.214943141390654],[30.9375,5.615985819155343],[32.34375,5.615985819155343]]]},"
tippecanoe":{"layer":"polygon","minzoom":7,"maxzoom":10},"properties":{}}
{"type":"Feature","geometry":{"type":"MultiPolygon","coordinates":[[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]]},"tippecanoe":{"layer":"polygon","minzoom":11,"ma
xzoom":16},"properties":{}}
{"type":"Feature","geometry":{"type":"MultiPolygon","coordinates":[[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]]},"tippecanoe":{"layer":"polygon","min
zoom":11,"maxzoom":16},"properties":{}}
{"type":"Feature","geometry":{"type":"MultiPolygon","coordinates":[[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]
,[],[],[],[],[],[]]},"tippecanoe":{"layer":"polygon","minzoom":11,"maxzoom":16},"properties":{}}
{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[30.995972670602328,5.520289328489753],[30
.983090496529258,5.546084477516672],[30.97657363126814,5.597520556228744],[30.995972670602328,5.597520556
228744],[31.042670554202743,5.558906209054328],[30.995972670602328,5.520289328489753]]]},"tippecanoe":{"l
ayer":"polygon","minzoom":7,"maxzoom":10},"properties":{}}
{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[30.9375,5.541978380873107],[30.9431936432
7208,5.53377158239266],[30.995972670602328,5.520289328489753],[30.949502120326656,5.435164943607333],[30.
9375,5.437489479260407],[30.9375,5.541978380873107]]]},"tippecanoe":{"layer":"polygon","minzoom":7,"maxzo
om":10},"properties":{}}
{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[31.442150580684824,5.404423662721911],[31
.394202368744345,5.447074410955338],[31.466285714314324,5.471373517973518],[31.442150580684824,5.40442366
2721911]]]},"tippecanoe":{"layer":"polygon","minzoom":7,"maxzoom":10},"properties":{}}
```
These are features extracted from the database, filtered by the rectangle of the module. 

Features with empty coordinates are not actually valid. We keep them because they does not do harm to tippecanoe.

If you want to modify _properties_ or _tippecanoe_, you can modify _modify.js_. You might edit _modify.js_ to change design or optimize vector tiles.

### check the qmatrix
It is too easy to create over-sized vector tiles. Over-sized vector tiles lead to slow performance. We need to get rid of over-sized vector tiles because they often cover important areas. 

Thus, we need to measure the size distribution of vector tiles.

#### install mbq - *c++11 required*

First, we install [mbq](https://github.com/hfu/mbq). We need a compiler for c++11.

```console
$ cd
$ git clone https://github.com/hfu/mbq.git
Initialized empty Git repository in /home/user1/mbq/.git/
remote: Counting objects: 52, done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 52 (delta 16), reused 23 (delta 9), pack-reused 21
Unpacking objects: 100% (52/52), done.
$ cd mbq
$ npm install

> integer@1.0.5 install /home/user1/mbq/node_modules/integer
> node-gyp rebuild

make: Entering directory `/home/user1/mbq/node_modules/integer/build'
  CXX(target) Release/obj.target/integer/src/integer.o
  SOLINK_MODULE(target) Release/obj.target/integer.node
  COPY Release/integer.node
make: Leaving directory `/home/user1/mbq/node_modules/integer/build'

> better-sqlite3@4.1.4 install /home/user1/mbq/node_modules/better-sqlite3
> node-gyp rebuild

make: Entering directory `/home/user1/mbq/node_modules/better-sqlite3/build'
  ACTION deps_sqlite3_gyp_action_before_build_target_unpack_sqlite_dep Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c

  TOUCH Release/obj.target/deps/action_before_build.stamp
  CC(target) Release/obj.target/sqlite3/gen/sqlite-autoconf-3240000/sqlite3.o
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c: In function ‘fts5MultiIterNew’:
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c:198438: warning: dereferencing pointer ‘z.6263’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c:198434: warning: dereferencing pointer ‘z.6263’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c:200904: note: initialized from here
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c: In function ‘fts5ApiQueryPhrase’:
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c:205600: warning: dereferencing pointer ‘pNew.6631’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3240000/sqlite3.c:207105: note: initialized from here
  AR(target) Release/obj.target/deps/sqlite3.a
  COPY Release/sqlite3.a
  CXX(target) Release/obj.target/better_sqlite3/src/better_sqlite3.o
  SOLINK_MODULE(target) Release/obj.target/better_sqlite3.node
  COPY Release/better_sqlite3.node
  CC(target) Release/obj.target/test_extension/deps/test_extension.o
  SOLINK_MODULE(target) Release/obj.target/test_extension.node
  COPY Release/test_extension.node
make: Leaving directory `/home/user1/mbq/node_modules/better-sqlite3/build'
npm notice created a lockfile as package-lock.json. You should commit this file.
added 4 packages from 3 contributors in 71.113s
[+] no known vulnerabilities found [5 packages audited]
```

#### check qmatrix
```console
$ cd ../pnd
$ mbq 8-150-124.mbtiles 
8-150-124.mbtiles: 50000
qz  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18
-1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 0 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 2 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 3 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 4 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 5 -1 -1 -1 -1 -1  0 -1 -1  1  1  2  3  5  8 11 13 14 -1 -1
 6 -1 -1 -1 -1 -1 -1  0  0  1  1  1  3  5  7  9 10 11 -1 -1
 7 -1 -1 -1 -1 -1 -1 -1  0  0  1  3  4  5  7  9 10 11 -1 -1
 8 -1 -1 -1 -1 -1 -1  0  0  1  2  2  3  5  7  8 10  9 -1 -1
 9 -1 -1 -1 -1 -1 -1 -1  0 -1 -1  1  4  5  7  8  8  7 -1 -1
10 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1  2  4  5  6  6  7  7 -1 -1
11 -1 -1 -1 -1 -1 -1 -1 -1 -1  1  2  3  4  4  5  6  5 -1 -1
12 -1 -1 -1 -1 -1 -1 -1 -1 -1  0  1  2  3  2  5  3  5 -1 -1
13 -1 -1 -1 -1 -1 -1 -1 -1  0 -1  1  0  1 -1  4 -1  5 -1 -1
14 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1  4 -1 -1
15 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
16 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
17 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
18 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
19 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
20 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
21 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
22 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
23 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
final result for 89468 tiles
qz  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18
-1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 0 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 2 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 3 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 4 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
 5 -1 -1 -1 -1 -1  0 -1 -1  1  1  2  3  5  8 11 13 15 -1 -1
 6 -1 -1 -1 -1 -1 -1  0  0  1  1  1  3  5  7  9 10 12 -1 -1
 7 -1 -1 -1 -1 -1 -1 -1  0  0  1  3  4  5  7  9 10 12 -1 -1
 8 -1 -1 -1 -1 -1 -1  0  0  1  2  2  3  5  7  8 10 11 -1 -1
 9 -1 -1 -1 -1 -1 -1 -1  0 -1 -1  1  4  5  7  8  8  9 -1 -1
10 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1  2  4  5  6  6  7  8 -1 -1
11 -1 -1 -1 -1 -1 -1 -1 -1 -1  1  2  3  4  4  5  6  7 -1 -1
12 -1 -1 -1 -1 -1 -1 -1 -1 -1  0  1  2  3  2  5  3  6 -1 -1
13 -1 -1 -1 -1 -1 -1 -1 -1  0 -1  1  0  1 -1  4 -1  6 -1 -1
14 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1  6 -1 -1
15 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
16 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
17 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
18 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
19 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
20 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
21 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
22 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
23 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1
```
This matrix shows the distribution of tile size in the mbtiles file. Each column is for each zoom level. Each row is for each size bin described in what I call [quantity level (q)](https://github.com/hfu/q). For example, at zoom level 6, there is 64 - 127 (q=6) files which takes 16K - 32K (16384 - 32767; q=14) bytes.

I have [a table for q](https://github.com/hfu/qtable).

#### install mbstat - *c++11 required*
(This tool is subject to change.)

```console
$ cd
$ git clone git@github.com:hfu/mbstat.git
Initialized empty Git repository in /home/fhidenori/mbstat/.git/
remote: Counting objects: 22, done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 22 (delta 10), reused 12 (delta 5), pack-reused 0
Receiving objects: 100% (22/22), 5.28 KiB, done.
Resolving deltas: 100% (10/10), done.
$ cd mbstat
$ npm install

> integer@1.0.3 install /home/fhidenori/mbstat/node_modules/integer
> node tools/install

make: Entering directory `/home/fhidenori/mbstat/node_modules/integer/build'
  CXX(target) Release/obj.target/integer/src/integer.o
  SOLINK_MODULE(target) Release/obj.target/integer.node
  COPY Release/integer.node
make: Leaving directory `/home/fhidenori/mbstat/node_modules/integer/build'

> better-sqlite3@4.1.1 install /home/fhidenori/mbstat/node_modules/better-sqlite3
> node deps/install

==> cwd: /home/fhidenori/mbstat/node_modules/better-sqlite3
==> /home/fhidenori/mbstat/node_modules/lzz-gyp/lzz-compiled/linux -hx hpp -sx cpp -k BETTER_SQLITE3 -d -hl -sl -e ./src/better_sqlite3.lzz
==> cwd: /home/fhidenori/mbstat/node_modules/better-sqlite3
==> node-gyp rebuild
make: Entering directory `/home/fhidenori/mbstat/node_modules/better-sqlite3/build'
  ACTION deps_sqlite3_gyp_action_before_build_target_unpack_sqlite_dep Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c
  TOUCH Release/obj.target/deps/action_before_build.stamp
  CC(target) Release/obj.target/sqlite3/gen/sqlite-autoconf-3210000/sqlite3.o
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c: In function ‘exprAnalyze’:
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:131262: warning: ‘eOp2’ may be used uninitialized in this function
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:131506: warning: ‘pRight’ may be used uninitialized in this function
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:131506: warning: ‘pLeft’ may be used uninitialized in this function
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c: In function ‘fts5MultiIterNew’:
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:191945: warning: dereferencing pointer ‘z.4988’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:191941: warning: dereferencing pointer ‘z.4988’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:194409: note: initialized from here
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c: In function ‘fts5ApiQueryPhrase’:
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:199090: warning: dereferencing pointer ‘pNew.5356’ does break strict-aliasing rules
Release/obj/gen/sqlite-autoconf-3210000/sqlite3.c:200595: note: initialized from here
  AR(target) Release/obj.target/deps/sqlite3.a
  COPY Release/sqlite3.a
  CXX(target) Release/obj.target/better_sqlite3/src/better_sqlite3.o
  SOLINK_MODULE(target) Release/obj.target/better_sqlite3.node
  COPY Release/better_sqlite3.node
  CC(target) Release/obj.target/test_extension/deps/test_extension.o
  SOLINK_MODULE(target) Release/obj.target/test_extension.node
  COPY Release/test_extension.node
make: Leaving directory `/home/fhidenori/mbstat/node_modules/better-sqlite3/build'
npm notice created a lockfile as package-lock.json. You should commit this file.
added 14 packages from 11 contributors in 71.739s
[+] no known vulnerabilities found [15 packages audited]
```

#### check statistics on mbtiles
```console
$ node mbstat.js ../pnd/8-150-124.mbtiles 
working with ../pnd/8-150-124.mbtiles...
```
We get no output because there is no over-sized vector tiles.

Let's say we had very strict size constraint that q=14 is not acceptable. In that case we edit mbstat.js like this:
```js
    //if (q <= 16) continue
    if (q < 14) continue
```

Then we can get statistics as below. For example, this shows the number of features in a single tile. If there is any layer which has too many features, say 10000 per tile, then it would be good idea to update modify.js to filter some features or to simply push these layer to larger zoom level.

```
$ node mbstat.js ../pnd/8-150-124.mbtiles 
working with ../pnd/8-150-124.mbtiles...
16/38520/31884(14) {"linestring":52,"point":2,"polygon":1197} {"undefined":1197}
16/38520/31885(14) {"linestring":49,"point":3,"polygon":1507} {"undefined":1507}
16/38516/31884(14) {"linestring":26,"point":7,"polygon":1393} {"undefined":1393}
16/38517/31885(14) {"linestring":42,"point":7,"polygon":1683} {"undefined":1683}
16/38516/31885(14) {"linestring":49,"point":2,"polygon":1518} {"undefined":1518}
16/38516/31889(14) {"linestring":32,"polygon":1575} {"undefined":1575}
16/38515/31886(14) {"linestring":60,"point":1,"polygon":1569} {"undefined":1569}
16/38515/31887(14) {"linestring":62,"point":4,"polygon":1819} {"undefined":1819}
16/38519/31882(14) {"linestring":27,"point":9,"polygon":1795} {"undefined":1795}
16/38507/31879(14) {"linestring":54,"point":1,"polygon":1373} {"undefined":1373}
16/38518/31882(14) {"linestring":36,"point":8,"polygon":1418} {"undefined":1418}
16/38515/31879(14) {"linestring":41,"polygon":1559} {"undefined":1559}
16/38514/31879(14) {"linestring":47,"polygon":1916} {"undefined":1916}
16/38514/31878(14) {"linestring":41,"point":2,"polygon":1571} {"undefined":1571}
16/38519/31887(14) {"linestring":57,"point":2,"polygon":1892} {"undefined":1892}
16/38519/31886(14) {"linestring":36,"point":1,"polygon":1416} {"undefined":1416}
16/38518/31887(14) {"linestring":63,"polygon":1629} {"undefined":1629}
16/38518/31886(14) {"linestring":47,"point":1,"polygon":1298} {"undefined":1298}
16/38519/31890(14) {"linestring":60,"polygon":1487} {"undefined":1487}
16/38515/31883(14) {"linestring":49,"point":2,"polygon":1594} {"undefined":1594}
16/38515/31882(14) {"linestring":49,"point":5,"polygon":1749} {"undefined":1749}
16/38514/31883(14) {"linestring":39,"point":1,"polygon":1415} {"undefined":1415}
16/38506/31882(14) {"linestring":49,"point":1,"polygon":1434} {"undefined":1434}
16/38515/31885(14) {"linestring":57,"point":2,"polygon":2094} {"undefined":2094}
16/38515/31884(14) {"linestring":34,"point":2,"polygon":1958} {"undefined":1958}
16/38514/31885(14) {"linestring":64,"point":4,"polygon":1305} {"undefined":1305}
16/38514/31884(14) {"linestring":43,"point":2,"polygon":1338} {"undefined":1338}
16/38521/31887(14) {"linestring":63,"point":1,"polygon":2454} {"undefined":2454}
16/38517/31882(14) {"linestring":40,"point":1,"polygon":1432} {"undefined":1432}
16/38516/31883(14) {"linestring":43,"point":7,"polygon":1879} {"undefined":1879}
16/38513/31879(14) {"linestring":41,"polygon":1452} {"undefined":1452}
16/38521/31886(14) {"linestring":63,"point":5,"polygon":1658} {"undefined":1658}
16/38520/31887(14) {"linestring":45,"point":2,"polygon":2101} {"undefined":2101}
16/38516/31882(14) {"linestring":52,"point":7,"polygon":1371} {"undefined":1371}
16/38520/31886(14) {"linestring":49,"point":5,"polygon":1805} {"undefined":1805}
16/38527/31893(14) {"linestring":56,"point":1,"polygon":1193} {"undefined":1193}
16/38519/31884(14) {"linestring":67,"point":4,"polygon":1259} {"undefined":1259}
16/38519/31889(14) {"linestring":63,"point":3,"polygon":2031} {"undefined":2031}
16/38526/31892(14) {"linestring":134,"polygon":1540} {"undefined":1540}
16/38519/31888(14) {"linestring":82,"point":3,"polygon":2114} {"undefined":2114}
16/38515/31881(14) {"linestring":33,"point":2,"polygon":1964} {"undefined":1964}
16/38514/31881(14) {"linestring":45,"point":4,"polygon":1682} {"undefined":1682}
16/38515/31880(14) {"linestring":29,"point":3,"polygon":1658} {"undefined":1658}
16/38507/31880(14) {"linestring":31,"polygon":1463} {"undefined":1463}
16/38514/31880(14) {"linestring":35,"point":1,"polygon":1456} {"undefined":1456}
16/38525/31891(14) {"linestring":101,"polygon":1047} {"undefined":1047}
16/38525/31890(14) {"linestring":31,"point":8,"polygon":1124} {"undefined":1124}
16/38511/31880(14) {"linestring":31,"point":1,"polygon":1517} {"undefined":1517}
16/38510/31881(14) {"linestring":53,"polygon":1417} {"undefined":1417}
16/38510/31880(14) {"linestring":57,"point":1,"polygon":1401} {"undefined":1401}
16/38514/31877(14) {"linestring":42,"polygon":1343} {"undefined":1343}
16/38511/31881(14) {"linestring":37,"point":2,"polygon":1215} {"undefined":1215}
16/38517/31886(14) {"linestring":68,"polygon":2309} {"undefined":2309}
16/38516/31887(14) {"linestring":61,"point":3,"polygon":1959} {"undefined":1959}
16/38516/31886(14) {"linestring":73,"point":4,"polygon":1908} {"undefined":1908}
16/38512/31881(14) {"linestring":49,"point":3,"polygon":1666} {"undefined":1666}
16/38513/31880(14) {"linestring":61,"polygon":2081} {"undefined":2081}
16/38521/31888(14) {"linestring":49,"point":1,"polygon":2063} {"undefined":2063}
16/38512/31880(14) {"linestring":33,"point":2,"polygon":1674} {"undefined":1674}
16/38513/31881(14) {"linestring":58,"point":1,"polygon":1604} {"undefined":1604}
16/38513/31884(14) {"linestring":104,"point":8,"polygon":1435} {"undefined":1435}
```


### visual check the vector tiles
We visual-check vector tiles before we host them. We use [tileserver-gl-light](https://github.com/klokantech/tileserver-gl#get-started) for that purpose. 

If you hands-on in a shared computer, you need to use different port number than other participant's port number. The example below uses port 9001.

```console
$ $ tileserver-gl-light 8-150-124.mbtiles -p 9001
Starting tileserver-gl-light v2.3.1
Automatically creating config file for 8-150-124.mbtiles
WARN: MBTiles not in "openmaptiles" format. Serving raw data only...
Run with --verbose to see the config file here.
Starting server
Listening at http://[::]:9001/
Startup complete
```

If you access port 9001 of your server on your browser. You will see TileServer GL light running. You can visual-check the data by clicking _inspect_ from there.

You can stop tileserver-gl-light by entering Ctrl-C on the console.

You can move on to [the hands-on vector tile hosting](https://hfu.github.io/hands-on-host).
