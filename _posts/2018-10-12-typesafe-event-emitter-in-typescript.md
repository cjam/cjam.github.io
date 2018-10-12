---
layout: article
title: Typesafe Event Emitter with Typescript
comments: true
description: My approach to creating a Typesafe Event Emitter in Typescript
---

I was recently working on a project that leveraged the `event-emitter` pattern.  The project used the implementation given by [`npm events`](https://www.npmjs.com/package/events) which basically implements the nodejs event-emitter in all its glory in vanilla javascript for everyone to use.

I do like the event emitter, but I also like using Typescript and getting type-safety.  I naturally went to use the [`npm @types/events`](https://www.npmjs.com/package/@types/events) package to get the typings, but it doesn't give type-safety around the types of events that you are using.  So I made my own declaration which gives this ability.  This has likely been done already, but I just put it in a gist for using later:

<script src="https://gist.github.com/cjam/fe79fb0f10f91d9bbea62d565c10efae.js"></script>

Using it is simple:

```ts
export default interface MyEvents {
    "stateChanged": {state:any,isChanged:boolean};
    "remoteDisconnected": void;
    "remoteConnected": void;
}

export interface MyEventEmitter extends TypedEventEmitter<MyEvents>{

}

// You'll actually have to create a class that inherits from EventEmitter at some point and cast it to a MyEventEmitter
```

Then when your using your event emitter in code, you'll get intellisense.  Here's an example from the project I'm currently working on:

![Typesafe Event Emitter](/images/event-type-safety.png);


Happy coding!