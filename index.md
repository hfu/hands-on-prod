# Vector tiles production with pnd.js and tippecanoe
## What is in this hands-on?
1. Extract data from [PostGIS](https://postgis.net/) to [NDJSON](http://ndjson.org/) while appending tippecanoe properties to features.
2. Obtain mbtiles from NDJSON using [tippecanoe](https://github.com/mapbox/tippecanoe).
3. Confirm the resulting mbtiles using [tileserver-gl-light](https://github.com/klokantech/tileserver-gl/blob/master/README_light.md).

## Recommended baseline software
- ubuntu Linux
- Google Chrome - you may want to add Chrome Secure Shell extension for ssh access to the server.
## Required software
### tippecanoe
Install from [GitHub](https://github.com/mapbox/tippecanoe).

### Node.js
Install from [nodejs.org](https://nodejs.org/ja/download/package-manager/#debian-and-ubuntu-based-linux-distributions-debian-ubuntu-linux)

Or you may want to use [n](https://github.com/tj/n):
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
[git](https://git-scm.com/) is installed in Linux in many cases.

## Hands-on steps
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