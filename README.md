# useSound

### A React Hook for Sound Effects

The web needs more (tasteful) sounds!

- 🔊 Lets your website communicate using 2 human senses instead of 1
- 🔥 Declarative Hooks API
- ⚡️ <10kb gzip total, <800 bytes in your bundle!
- ✨ Built with Typescript
- 🗣 Uses a powerful, battle-tested audio utility: **Howler.js**

[![Minified file size](https://img.badgesize.io/https://www.unpkg.com/use-sound@0.3.0/dist/use-sound.esm.js.svg)](https://bundlephobia.com/result?p=use-sound@0.3.0) [![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT) [![NPM version](https://img.shields.io/npm/v/use-sound)](https://www.npmjs.com/package/use-sound) [![Code of Conduct](https://img.shields.io/badge/Code%20of-conduct-d03eaf?style=flat)](https://github.com/tessel/project/blob/master/CONDUCT.md)

---

## Installation

Package can be added using **yarn**:

```bash
yarn add use-sound
```

Or, use NPM:

```bash
npm install use-sound
```

UMD build available on [unpkg](https://www.unpkg.com/browse/use-sound@0.3.0/dist/use-sound.cjs.production.min.js).

---

## Demo

See the library in action on Storybook:

// link

## Examples

### Play sound on click

```js
import useSound from 'use-sound';

import boopSfx from '../../sounds/boop.mp3';

const BoopButton = () => {
  const [play] = useSound(boopSfx);

  return <button onClick={play}>Boop!</button>;
};
```

### Playing on hover

This demo only plays the sound while hovering over an element. The sound pauses when the mouse leaves the element:

> NOTE: Many browsers disable sounds until the user has clicked somewhere on the page. If you're not hearing anything with this example, try clicking anywhere and trying again.

```js
import useSound from 'use-sound';

import fanfareSfx from '../../sounds/fanfare.mp3';

const FanfareButton = () => {
  const [play, { stop }] = useSound(fanfareSfx);

  return (
    <button onMouseEnter={play} onMouseLeave={stop}>
      <span role="img" aria-label="trumpet">
        🎺
      </span>
    </button>
  );
};
```

### Increase pitch on every click

With the `playbackRate` option, you can change the speed/pitch of the sample. This example plays a sound and makes it 10% faster each time:

```js
import useSound from 'use-sound';

import glugSfx from '../../sounds/glug.mp3';

export const RisingPitch = () => {
  const [playbackRate, setPlaybackRate] = React.useState(0.75);

  const [play] = useSound(glugSfx, {
    playbackRate,
    // `interrupt` ensures that if the sound starts again before it's
    // ended, it will truncate it. Otherwise, the sound can overlap.
    interrupt: true,
  });

  const handleClick = () => {
    setPlaybackRate(playbackRate + 0.1);
    play();
  };

  return (
    <Button onClick={handleClick}>
      <span role="img" aria-label="Person with lines near mouth">
        🗣
      </span>
    </Button>
  );
};
```

---

## Usage Notes

### No sounds immediately after load

For the user's sake, browsers don't allow websites to produce sound until the user has interacted with them (eg. by clicking on something). No sound will be produced until the user clicks, taps, or triggers something.

`useSound` takes advantage of this: because we know that sounds won't be needed immediately on-load, we can lazy-load a third-party dependency.

`useSound` will add about 800 bytes gzip to your bundle, and will asynchronously fetch an additional package after load, which clocks in around 9kb gzip.

If the user does happen to click with something that makes noise before this dependency has been loaded and fetched, it will be a no-op (everything will still work, but no sound effect will play). In my experience this is exceedingly rare.

### Respects changes to values

Consider the following snippet of code:

```js
const [playbackRate, setPlaybackRate] = React.useState(0.75);

const [play] = useSound('/path/to/sound', { playbackRate });
```

`playbackRate` doesn't just serve as an _initial_ value for the sound effect. If `playbackRate` changes, the sound will immediately begin playing at a new rate. This is true for all options passed to the `useSound` hook.

---

## API Documentation

The `useSound` hook takes two arguments:

- A URL to the sound that it wil load
- A config object (`HookOptions`)

It produces an array with two values:

- A function you can call to trigger the sound
- An object with additional data and controls (`ExposedData`)

When calling the function to play the sound, you can pass it a set of options (`PlayOptions`).

Let's go through each of these in turn.

### HookOptions

When calling `useSound`, you can pass it a variety of options:

| Name         | Value     |
| ------------ | --------- |
| volume       | number    |
| playbackRate | number    |
| interrupt    | boolean   |
| soundEnabled | boolean   |
| sprite       | SpriteMap |
| [delegated]  | —         |

- `interrupt` specifies whether or not the sound should be able to "overlap" if the `play` function is called again before the sound has ended.
- `soundEnabled` allows you to pass a value (typically from context or redux or something) to mute all sounds. Note that this can be overridden in the `PlayOptions`, see below
- `sprite` allows you to use a single `useSound` hook for multiple sound effects. See "Advanced" below.

`[delegated]` refers to the fact that any additional argument you pass in `HookOptions` will be forwarded to the `Howl` constructor. See "Escape hatches" below for more information.

### The `play` function

When calling the hook, you get back a play function as the first item in the tuple:

```js
const [play] = useSound('/meow.mp3');
//      ^ What we're talking about
```

You can call this function without any arguments when you want to trigger the sound. You can also call it with a `PlayOptions` object:

| Name              | Value   |
| ----------------- | ------- |
| id                | string  |
| forceSoundEnabled | boolean |
| playbackRate      | number  |

- `id` is used for sprite identification. See "Advanced" below.
- `forceSoundEnabled` allows you to override the `soundEnabled` boolean passed to `HookOptions`. You generally never want to do this. The only exception I've found: playing a sound when you click the "Disable sound" button. I like to play a little “powering off” sound effect, and I don't want it to be muted.
- `playbackRate` is another way you can set a new playback rate, same as in `HookOptions`. In general you should prefer to do it through `HookOptions`, this is an escape hatch.

### ExposedData

The hook produces a tuple with 2 options, the play function and an `ExposedData` object:

```js
const [play, exposedData] = useSound('/meow.mp3');
//                ^ What we're talking about
```

| Name      | Value                            |
| --------- | -------------------------------- |
| stop      | function ((id?: string) => void) |
| pause     | function ((id?: string) => void) |
| isPlaying | boolean                          |
| duration  | number (or null)                 |
| sound     | Howl (or null)                   |

- `stop` is a function you can use to pre-emptively halt the sound.
- `pause` is like `stop`, except it can be resumed from the same point. Unless you know you'll want to resume, you should use `stop`; `pause` hogs resources, since it expects to be resumed at some point.
- `isPlaying` lets you know whether this sound is currently playing or not. When the sound reaches the end, or it's interrupted with `stop` or `paused`, this value will flip back to `false`. You can use this to show some UI only while the sound is playing.
- `duration` is the length of the sample, in milliseconds. It will be `null` until the sample has been loaded. Note that for sprites, it's the length of the entire file.
- `sound` is an escape hatch. It grants you access to the underlying `Howl` instance. See the [Howler documentation](https://github.com/goldfire/howler.js) to learn more about how to use it. Note that this will be `null` for the first few moments after the component mounts.

---

## Advanced

### Sprites

An audio sprite is a single audio file that holds multiple samples. If a page on your site uses 20 different audio files, it can be a performance win to combine them all into a single file, and to slice up that file into individually-playable sounds.

> You probably don't need to use sprites! The only real benefit to this is performance, and technologies like HTTP/2 are making this a moot point.

For sprites, we'll need to define a `SpriteMap`. It looks like this:

```js
const spriteMap = {
  laser: [0, 300],
  explosion: [1000, 300],
  meow: [2000, 75],
};
```

`SpriteMap` is an object. The keys are the `id`s for individual sounds. The value is a tuple (array of fixed length) with 2 items:

- The starting time of the sample, in milliseconds, counted from the very beginning of the sample
- The length of the sample, in milliseconds.

This visualization might make it clearer:

![Waveform visualization showing how each sprite occupies a chunk of time, and is labeled by its start time and duration](./docs/sprite-explanation.png)

We can pass our SpriteMap as one of our HookOptions:

```js
const [play] = useSound('/path/to/sprite.mp3', {
  sprite: {
    laser: [0, 300],
    explosion: [1000, 300],
    meow: [2000, 75],
  },
});
```

To play a specific sprite, we'll pass its `id` when calling the `play` function:

```js
<button
  onClick={() => play('laser')}
>
```

### Escape hatches

Howler is a very powerful library, and we've only exposed a tiny slice of what it can do in `useSound`. We expose two escape hatches to give you more control.

First, any unrecognized option you pass to `HookOptions` will be delegated to `Howl`. You can see the [full list](https://github.com/goldfire/howler.js#options) of options in the Howler docs. Here's an example of how we can use `onend` to fire a function when our sound stops playing:

```js
const [play] = useSound('/thing.mp3', {
  onend: () => {
    console.info('Sound ended!');
  },
});
```

If you need more control, you should be able to use the `sound` object directly, which is an instance of Howler.

For example: Howler [exposes a `fade` method](https://github.com/goldfire/howler.js#fadefrom-to-duration-id), which lets you fade a sound in or out. You can call this method directly on the `sound` object:

```js
const Arcade = () => {
  const [play, { sound }] = useSound('/win-theme.mp3');

  return (
    <button
      onClick={() => {
        // You win! Fade in the victory theme
        sound.fade(0, 1, 1000);
      }}
    >
      Click to win
    </button>
  );
};
```
