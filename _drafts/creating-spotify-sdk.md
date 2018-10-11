---
layout: article
title: (Re)Creating React Native Spotify SDK
description: Some notes on my work on upgrading the Spotify SDK used within rn-spotify-sdk
tags: [objective-c, react-native, spotify, ios, android, java]
---

Many thanks to [@lufinkey](https://github.com/lufinkey) on his great work setting up the [rn-spotify-sdk](https://github.com/lufinkey/react-native-spotify) project.


## Goal

To upgrade the `rn-spotify-sdk` project to use the latest Spotify SDKs:
- [IOS Spotify SDK](https://github.com/spotify/ios-sdk)
- [Android Spotify SDK](https://github.com/spotify/android-sdk)


# Day 1

## Linked my rn-spotify-sdk within a project using it

After talking with `lufinkey` he recommended for local development to replace the `rn-spotify-sdk` within the `node_modules` of a project with the git clone.  Instead I decided to try and use `yarn link` to do this.

First within the directory for my `rn-spofity-sdk` repo I ran:

```sh
yarn link
```

then within the project directory that was using the `rn-spotify-sdk` I ran:

```sh
yarn link rn-spotify-sdk
```

Which essentially sim links my local package into this project.  

Lets begin...

annnd that doesn't work.

### Metro Packager Doesn't Support Symlinks

In order to work on this Native Module, the project needs to be situated within the `node_modules` of a project that is using it.  This has to do with how the *xcode project* references are setup, unfortunately you can't just build the project on its own (at least not that I'm aware of).  So generally this is what you can use `yarn link` for.  It allows you to take a package on your local machine and use it within another project.  How it does this is by using symlinks...

Unfortunately after some time I realized that the react-native packager doesn't seem to support symlinks which is what `yarn link` uses under the hood.. so that's too bad.  I found this article discussing a possible solution:

<https://www.bram.us/2018/03/10/working-with-symlinked-packages-in-react-native/>

However the `wml` utility just didn't work for me.  It could be that I just recently upgraded my mac os to `Mojave`.  Unsure, but it looks like I'll need to copy over my project into my `node_modules` folder, which is a bit scary because I'll have to remember not to delete my `node_modules` folder.  Anyways, lets try it out...

Yea, so that didn't work out well.   I accidentally installed a new npm package which in turn overwrote my work.  Luckily I hadn't closed XCode and was able to recover most of my work.  I'm now being more careful to commit my changes as they happen. 

`#Learning`

## Change Spotify SDK Submodule 

The `rn-spotify-sdk` project has a submodule within the `/ios/external/spotifysdk` which points to the SpotifySDK github repo.  I need to switch this submodule to the new SDK.

Changed the sub module within `ios/external/SpotifySDK` to point to the new [Spotify IOS SDK](git@github.com:spotify/ios-sdk.git)

did this by doing the following:

```sh
git rm ios/external/SpotifySDK
git submodule add git@github.com:spotify/ios-sdk.git ios/external/SpotifySDK/
```

Replaced reference to previous Spotify SDK projects and added `SpotifyiOS.framework` to the `RNSpotify.xcodeproject`.

Changed iOS Deployment Target to `9.0` to support new Spotify SDK.

## Setup the iOS SDK

Instructions from <https://developer.spotify.com/documentation/ios/quick-start/#setup-the-ios-sdk>


Now that I'm inspecting the previous incarnation of the API, it appears that many of the underlying Spotify API's may have changed.  The difference between this new api and previous is that this new one represents the Spotify Remote Api, (i.e. control spotify on the same device remotely) versus the streaming api which handles, you guessed it, streaming from Spotify Services.  I think these changes will likely end up in a new package entirely, but we shall see.



## Day 2

Ok so yesterday was a big day for learning some Objective-C.  I've got some things working but I am still quite a ways away from something useful.  That being said, I have managed to get some rudimentary authentication happening on my phone and have managed to authorize and play a [Justin Beiber track](https://open.spotify.com/track/69bp2EbF7Q2rqc5N3ylezZ).  This was taking straight from the examples found [here](https://github.com/spotify/ios-sdk/tree/v1.0.0#using-the-built-in-app-remote-control-authorization-flow-recommended)

Very Cool.  All done right?  Let's implement `skipToNext`...

*Wrong*

So my `appRemote` instance can't connect to my spotify app.  Turns out the authAndPlay works because the track is sent in the payload so spotify just grabs it and plays it.  The reason my application can't connect is because I'm not grabbing the url parameters being passed back from spotify. Which can be seen in *Step 4* of the example linked to above.  Darn, what's the pattern for solving this. 

## Day 3 

Day 2 was pretty good, however, we're at the stage where we need to get the url parameters back into our native module.  Looking for a nice pattern.   Luckily, I remembered using a `react-native` library in the past that required some url support.. right, [`react-native-google-signin`](https://github.com/react-native-community/react-native-google-signin/blob/a19261081783d2f202dfa185b1f2654e8945f43e/ios-guide.md).  It makes sense that any auth flow type library requires this type of thing because tokens are generally handed back to your application via url parameters. 

Lets see how they did it in their library.
Hmm.. so the `GoogleSigninSDK` has a shared instance that appears to be static.. we may have to follow the same pattern for our SpotifySDK.

Ok, so I ended up setting up a singleton pattern for my `RNSpotify` module.  The trick with it (which may be bad practice) came with the fact that this class instance is actually instantiated by the React Native Bridge when the application starts up.  It automatically calls `alloc` and `init` on our class.  However, within our application we need to be able to access the singleton instance through a static variable like `[RNSpotify sharedInstance]`.  In order to get around this, I did what seems like a bit of a hack, but here it is (Objective-C experts go easy on me):

```obj-c

// Static singleton instance
static RNSpotify *sharedInstance = nil;

// interface / implementation stuff in here 
@implementation RNSpotify

+ (instancetype)sharedInstance {
    // Hopefully ReactNative can take care of allocating and initializing our instance
    // otherwise we'll need to check here
    return sharedInstance;
}

-(id)init
{
    // This is to hopefully maintain the singleton pattern within our React App.
    // Since ReactNative is the one allocating and initializing our instance,
    // we need to store the instance within the sharedInstance otherwise we'll
    // end up with a different one when calling shared instance statically
    if(sharedInstance == nil){
        if(self = [super init])
        {
            NSLog(@"RNSpotify Initialized");
            _initialized = NO;
            _sessionManagerCallbacks = [NSMutableArray array];
        }
        
        static dispatch_once_t once;
        dispatch_once(&once, ^
                      {
                          sharedInstance = self;
                      });
    }else{
        NSLog(@"Returning shared instance");
    }
    return sharedInstance;
}
@end
```

Seems to work so far.

## Day 4

Alright, I was heads down for quite a bit and am happy to say that I've finally got some Spotify Remoting working.  I had finally gotten the authentication flow setup and ran into an issue where the `AppRemote` would never connect.  I found an [issue](https://github.com/spotify/ios-sdk/issues/26) that sounded similar and figured out that the reason none of my delegate handlers were being called was due to the code not being called on the main thread.  So it just needed to be dispatched:

```obj-c
- (void)initializeAppRemote:(SPTSession*)session completionCallback:(RNSpotifyCompletion*)completion{
    _appRemote = [[SPTAppRemote alloc] initWithConfiguration:_apiConfiguration logLevel:SPTAppRemoteLogLevelDebug];
    _appRemote.connectionParameters.accessToken = session.accessToken;
    _appRemote.delegate = self;
    // Add our callback before we connect
    [_appRemoteCallbacks addObject:completion];
    dispatch_async(dispatch_get_main_queue(), ^(void){
        [self->_appRemote connect];
    });
}
```

This also applied to using the `AppRemote` apis and needed to be used for the methods that are exposed by the api:

```obj-c
RCT_EXPORT_METHOD(pause:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject){
    dispatch_async(dispatch_get_main_queue(), ^(void){
        [self->_appRemote.playerAPI pause:^(id  _Nullable result, NSError * _Nullable error) {
            if(error != nil){
                [[RNSpotifyError errorWithNSError:error] reject:reject];
            }else{
                resolve([NSNull null]);
            }
        }];
    });
}
```

**Edit**: *Was looking through React-Natives source code and found a nice helper method to make this simpler* 

```obj-c
    // Pause Method
    RCT_EXPORT_METHOD(pause:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject){
        RCTExecuteOnMainQueue(^{
            [self->_appRemote.playerAPI pause:^(id  _Nullable result, NSError * _Nullable error) {
                if(error != nil){
                    [[RNSpotifyError errorWithNSError:error] reject:reject];
                }else{
                    resolve([NSNull null]);
                }
            }];
        });
    }
```

Also figured out how to make a function that returns a callback so I was able to clean this code up further to:

```obj-c
// Default callback handler
+(void (^)(id _Nullable, NSError * _Nullable))defaultSpotifyRemoteCallback:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject{
    return ^(id  _Nullable result, NSError * _Nullable error) {
        if(error != nil){
            [[RNSpotifyError errorWithNSError:error] reject:reject];
        }else{
            resolve([NSNull null]);
        }
    };
}

// Pause Method
RCT_EXPORT_METHOD(pause:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject){
    RCTExecuteOnMainQueue(^{
        [self->_appRemote.playerAPI pause:[RNSpotify defaultSpotifyRemoteCallback:resolve reject:reject]];
    });
}
```

The abstractions are reducing a lot of duplicate code 👍.

## Day 5

Alright, today I'm going to try and work on getting the rest of the exposed apis fleshed out.  This will need to include some conversion from native objects to javascript objects.


```obj-c
RCT_EXPORT_METHOD(getPlayerState:(RCTPromiseResolveBlock)resolve reject:(RCTPromiseRejectBlock)reject){
    RCTExecuteOnMainQueue(^{
        [self->_appRemote.playerAPI getPlayerState:^(id _Nullable result, NSError * _Nullable error) {
            if(error != nil){
                [[RNSpotifyError errorWithNSError:error] reject:reject];
            }else{
                if([result conformsToProtocol:@protocol(SPTAppRemotePlayerState)]){
                    resolve([RNSpotifyConvert SPTAppRemotePlayerState:result]);
                }else{
                    [[RNSpotifyError errorWithCodeObj:RNSpotifyErrorCode.BadResponse message:@"Couldn't parse returned player state"] reject:reject];
                }
            }
        }];
    });
}
```

Ok so above I'm using our `RNSpotifyConvert` class which returns us `JSON` for various object types within the system (like `PlayerState`,`Track`,`Album` etc), these functions are pretty simple with one catch:

```obj-c
+(id)SPTAppRemotePlayerState:(NSObject<SPTAppRemotePlayerState>*) state{
    if(state == nil)
    {
        return [NSNull null];
    }
    return @{
        @"track": [RNSpotifyConvert SPTAppRemoteTrack:state.track],
        @"playbackPosition": [NSNumber numberWithInteger:state.playbackPosition],
        @"playbackSpeed": [NSNumber numberWithFloat:state.playbackSpeed],
        @"paused": [NSNumber numberWithBool:state.isPaused],
        @"playbackRestrictions": [RNSpotifyConvert SPTAppRemotePlaybackRestrictions:state.playbackRestrictions],
        @"playbackOptions": [RNSpotifyConvert SPTAppRemotePlaybackOptions:state.playbackOptions]
    };
}
```

Here we're returning an object literal, which can contain keys and objects.  But since `BOOL`, `float` and `integer` are not objects, we have to use th `NSNumber` class which wraps these primitive types in an object and allows them to be inserted into our JSON object literal.  Also, using nested conversion functions to represent the nested JSON Structure.

### Events

Here's a cool post on the [Gory details of how events are handled in `react-native`](https://levelup.gitconnected.com/react-native-events-in-gory-details-what-happens-on-the-way-to-listeners-2cee6c55940c)

So the Spotify IOS SDK supports events by using `Delegates` and a subscription pattern.  So basically, you need to pass it an object that will receive the events and then you need to tell the API that you would like to start receiving events.  Since we want our API wrapper to be a good citizen we'd like to make sure that we `subscribe`/`unsubscribe` to the api events if there are/aren't any subscribers respectively.

In this project, the eventing is handled by a package called `react-native-events` which essentially adds the [`EventEmitter`](https://github.com/Gozala/events/blob/master/events.js) methods.  This allows our module to be used like so:

```ts
import SpotifyAPI from 'rn-spotify-sdk';
SpotifyAPI.on('someEvent',(arg)=>{
    // handle event
});
```

There are few things that I'd like to do with our implementation.  

1. Type safety on events (since we're using Typescript)
2. Subscription management (i.e. subscribe/unsubscribe when there are/aren't any listeners)

#### Type Safety

I handled type safety by using some nice tricks provided by the Typescript compiler.  Mostly the `keyof`.

First I created an interface that represents all of the events within the system:

```ts
export default interface SpotifyApiEvents {
    "playerStateChanged": SpotifyPlayerState
}
```

Here the key i.e `"playerStateChanged"` is the actualy name of the event.  The value `SpotifyPlayerState` is the arguments of the event.  So how is used?  This part I'm not as proud of because it involved redefining the `EventEmitter` interface, however, I feel that the EventEmitter is pretty stable so I will allow it.

```ts
export interface SpotifyEventEmitter{
    on<K extends keyof SpotifyApiEvents>(name:K,listener:(v:SpotifyApiEvents[K])=>void) : this;
    addListener<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    once<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    prependListener<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    prependOnceListener<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    removeListener<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    off<K extends keyof SpotifyApiEvents>(event: K, listener: (v:SpotifyApiEvents[K])=>void): this;
    removeAllListeners<K extends keyof SpotifyApiEvents>(event?:K): this;
    setMaxListeners(n: number): this;
    getMaxListeners(): number;
    listeners<K extends keyof SpotifyApiEvents>(event: K): (v:SpotifyApiEvents[K])=>void[];
    rawListeners<K extends keyof SpotifyApiEvents>(event: K): (v:SpotifyApiEvents[K])=>void[];
    emit<K extends keyof SpotifyApiEvents>(event: K, args: SpotifyApiEvents[K]): boolean;
    eventNames<K extends keyof SpotifyApiEvents>(): Array<K>;
    listenerCount<K extends keyof SpotifyApiEvents>(type: K): number;
}

export interface SpotifyNativeApi extends SpotifyEventEmitter{
    // Api methods here
}
```

Now when we're within our typescript based project we can do:

![Event Emitter Intellisense](/images/event-type-safety.png)

👌

### Subscription Management

Ok, so we want to subscribe / unsubscribe when there are/aren't listeners to events.  Luckily, the `react-native-events` actually wires in the methods of the `EventEmitter` (other than emit).  This is great, because the `EventEmitter` has it's own meta-events `newListener` and `removeListener` which both have argument types of 
```ts
(eventName:string,listener:Function) 
```
Excellent.  So we should be able to listen to these events and be able to tell when our api events are subscribed to.

```ts
// The events produced by the eventEmitter implementation around 
// when new event listeners are added and removed
const metaEvents = {
    newListener: 'newListener',
    removeListener: 'removeListener'
};

// Want to ignore the metaEvents when sending our subscription events
const ignoredEvents = Object.keys(metaEvents);

(SpotifyNative as any).on(metaEvents.newListener, (type: string) => {
    if (ignoredEvents.indexOf(type) === -1) {
        const listenerCount = SpotifyNative.listenerCount(type as any);
        // If this is the first listener, send an eventSubscribed event
        if (listenerCount == 0) {
            RNEvents.emitNativeEvent(SpotifyNative, "eventSubscribed", type);
        }
    }
}).on(metaEvents.removeListener, (type: string) => {
    if (ignoredEvents.indexOf(type) === -1) {
        const listenerCount = SpotifyNative.listenerCount(type as any);
        if (listenerCount == 0) {
            RNEvents.emitNativeEvent(SpotifyNative, "eventUnsubscribed", type);
        }
    }
});
```

So we just send an event down to the Native module whenever either an event gets it's first listener or loses its last listener.  Then in native land, I maintain a collection of `subscriber` classes which are responsible for subscribing / unsubscribing for a particular event.  Then if one of these special events come in I just run `subscribe` or `unsubscribe` on the Subscriber object for that event if it exists.  💥






[objc-properties]: https://www.ios-blog.com/tutorials/objective-c/objective-c-property-attribute-reference-guide/