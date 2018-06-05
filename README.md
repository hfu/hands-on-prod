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
Although vector tiles works well with all modern web browsers. You may want to add Chrome Secure Shell extension for ssh access to the server.

## Required software

### tippecanoe

You can install from [GitHub](https://github.com/mapbox/tippecanoe).

### Node.js

You can install from [nodejs.org](https://nodejs.org/ja/download/package-manager/#debian-and-ubuntu-based-linux-distributions-debian-ubuntu-linux)

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

### prepare a config file (config/default.hjson) for connecting the datasource

```console
$ mkdir config
$ vi config/default.hjson 
```

You need to input the configurations. Your hands-on tutor may provide the configurations. Otherwise, the following is an example of the content of the config file.

```hjson
{
  host: postgis.host.name
  user: username
  password: secretpassword
  modules: [ // array of z=5 modules expressed by [z, x, y]
    [5, 18, 15]
  ]
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

Or you can use other editors such as nano if you are not familiar with vi.

You may want to check the schema and example values of source database by [schema-check](https://github.com/hfu/schema-check).

The syntax of config.hjson is [hjson](https://hjson.org/). You can prepare config.json if you prefer, and the script will automatically recongize the traditional syntax.

You can check the location of z=5 module from [this site](http://maps.gsi.go.jp/#5/3.557283/28.520508/&ls=seamlessphoto&disp=1&lcd=seamlessphoto&vs=c0j0h0k0l0u0t1z0r0s0f1&d=v) for example.

### prepare modify.js

### convert to vector tiles
```console
```
