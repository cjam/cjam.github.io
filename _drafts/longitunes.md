---
title: Creating Geo-Referenced Playlist app
date: 2018-10-12 10:05:24.075000000 -07:00
layout: article
description: Notes on creating my Geo Referenced playlist app code-named Longitunes
---

## Creating my new react app

Ran into a bunch of issus with the new `create-react-native-app`, somethings still need to be ironed out I think.  Anyways, here is some of the issues I faced.

### Troubleshooting

#### `Problems with glog (config.h missing) and gflags/gflags.h on Xcode 10`

Turned out be caused by the new build system in X Code 10.  It was fixed using this comment: 

https://github.com/facebook/react-native/issues/19774#issuecomment-425897085


#### Bunch of bundler errors

```sh
error: bundling failed: Error: Unable to resolve module `schedule/tracking`from `/Users/roughdraft/Projects/_craplets/longitunes/node_modules/react-native/Libraries/Renderer/oss/ReactNativeRenderer-dev.js`: Module `schedule/tracking` does not exist in the Haste module map
```

<https://github.com/facebook/react-native/issues/21150#issuecomment-426496581>

You can see from the link above what I ended up having to do to fix this.

Ok moving on.


### Setting up React-Native with Typescript

```sh
yarn add packages here
```

```json
package.json here
```

```
babelrc
```

```json
tsconfig
```

```js
rn-cli.config.js here
```

## Directory Structure


## Loading Images
In order for typescript to allow you to import images, we have to let it know that our images are actually modules.  To do this, we simply add a `images.d.ts` file (could be named anything) within your source root (`src` for me).

```ts
declare module '*.png';
declare module '*.svg';
declare module '*.jpg';
declare module '*.jpeg';
```

Then in your component/module you can import it like so:

```ts
import robotDev from '../../assets/images/robot-dev.png';
```

## Loading Fonts

Source Article: <https://medium.com/react-native-training/react-native-custom-fonts-ccc9aacf9e5e>

I actually only had to do the first step of adding in the assets to the `rnpm` section of my `package.json` like so:

```json
  "rnpm":{
    "assets":[
      "./assets/fonts/"
    ]
  },
```

then run `react-native link`
Oops, I forgot I had already installed the `rn-spotify-sdk` package.. and it tried to link itself in there and broke my build.  Ok lets try and fix that.

## React-Native-SDK

So right now I'm doing everything from the perspective of IOS.  So lets try and just link the `rn-spotify-sdk` package according to their [Readme](https://github.com/lufinkey/react-native-spotify)

```sh
react-native link rn-spofity-sdk
react native link react-native-events
```

and 🥁 .... `** BUILD FAILED **` damnit.  Alright time to open up the `xcodeproject` and figure out what's going on in there.





