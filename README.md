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
$ git clone git@github.com:hfu/mbq.git
Initialized empty Git repository in /home/fhidenori/mbq/.git/
remote: Counting objects: 52, done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 52 (delta 16), reused 23 (delta 9), pack-reused 21
Receiving objects: 100% (52/52), 8.85 KiB, done.
Resolving deltas: 100% (24/24), done.
$ cd mbq
$ npm install

> integer@1.0.3 install /home/fhidenori/mbq/node_modules/integer
> node tools/install

make: Entering directory `/home/fhidenori/mbq/node_modules/integer/build'
  CXX(target) Release/obj.target/integer/src/integer.o
  SOLINK_MODULE(target) Release/obj.target/integer.node
  COPY Release/integer.node
make: Leaving directory `/home/fhidenori/mbq/node_modules/integer/build'

> better-sqlite3@4.1.1 install /home/fhidenori/mbq/node_modules/better-sqlite3
> node deps/install

==> cwd: /home/fhidenori/mbq/node_modules/better-sqlite3
==> /home/fhidenori/mbq/node_modules/lzz-gyp/lzz-compiled/linux -hx hpp -sx cpp -k BETTER_SQLITE3 -d -hl -sl -e ./src/better_sqlite3.lzz
==> cwd: /home/fhidenori/mbq/node_modules/better-sqlite3
==> node-gyp rebuild
make: Entering directory `/home/fhidenori/mbq/node_modules/better-sqlite3/build'
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
make: Leaving directory `/home/fhidenori/mbq/node_modules/better-sqlite3/build'
npm notice created a lockfile as package-lock.json. You should commit this file.
added 7 packages from 4 contributors in 71.922s
[+] no known vulnerabilities found [8 packages audited]
$ npm link
up to date in 1.525s
[+] no known vulnerabilities found [8 packages audited]

/home/fhidenori/local/bin/mbq -> /home/fhidenori/local/lib/node_modules/mbq/mbq.js
/home/fhidenori/local/lib/node_modules/mbq -> /home/fhidenori/mbq


   ╭────────────────────────────────────────────────────────────────╮
   │                                                                │
   │       New minor version of npm available! 6.0.1 → 6.1.0        │
   │   Changelog: https://github.com/npm/npm/releases/tag/v6.1.0:   │
   │               Run npm install -g npm to update!                │
   │                                                                │
   ╰────────────────────────────────────────────────────────────────╯
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

### check the vector tiles

FIXME
