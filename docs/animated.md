---
id: animated
title: Animated
sidebar_label: Animated
---

## Example of use

`Animated` allows to create declarative animations that are fluid, powerful and easy to build.

### basic

The simplest animation starts with creating an animated value and using one of the built-in animations to change its value over time.

The following example demonstrates use of `Animated.Timing` in order to animate an animated value throughout given period of time.

```reason
open Rebolt;

let animatedValue = Animated.Value.create(0.0);

let animation =
  Animated.Timing.animate(
    ~value=animatedValue,
    ~toValue=`raw(1.0),
    ~duration=100.0,
    (),
  );

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

### multiple

Animations can be also combined together in complex ways using composition functions.

The following example demonstrates use of `Animated.sequence` in order to run animations in a sequence, one by one.

```reason
open Rebolt;

let animatedValue = Animated.Value.create(0.0);

let animation =
  Animated.sequence([|
    Animated.Timing.animate(
      ~value=animatedValue,
      ~toValue=`raw(1.0),
      ~duration=100.0,
      (),
    ),
    Animated.Timing.animate(
      ~value=animatedValue,
      ~toValue=`raw(0.0),
      ~duration=100.0,
      (),
    ),
  |]);

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

### calculation

You can combine two animated values via addition, multiplication, division, or modulo to make a new animated value.

The following example demonstrates use of `Animated.multiply` in order to reverse the value of the `animatedValue`.

```reason
open Rebolt;

let animatedValue = Animated.Value.create(0.0);

let newAnimatedValue = Animated.multiply(animatedValue, Animated.Value.create(-1.0));
```

Keep in mind that calculated values (such as `newAnimatedValue` from the snippet above) cannot be animated. Trying to pass them to any of the animated functions will result in a type error.

### interpolation

You can interpolate an animated value in order to bind to its value and change the output.

The following example demonstrates interpolation in order to map values of an animated value to the opacity of a container.

```reason
let animatedValue = Animated.Value.create(100.0);

let animatedOpacity =
  Animated.Value.interpolate(
    animatedValue,
    ~inputRange=[0.0, 100.0],
    ~outputRange=`float([0.0, 1.0]),
    ~extrapolate=Animated.Interpolation.Clamp,
    (),
  );
```

### styling

Animated values can be passed to an animated component in order to change its apperance as the animated value changes.

The example below demonstrates animating opacity of a component.

```reason
open Rebolt;

let animatedValue = Animated.Value.create(0.0);

let component = ReasonReact.statelessComponent("MyComponent");

let containerStyle = Style.(
  style([
    opacity(Animated(animatedValue))
    flex(1.0)
  ])
);

let make = _children => {
  ...component,
  didMount: _self => {
    Animated.CompositeAnimation.start(
      Animated.Timing.animate(
        ~value=animatedValue,
        ~toValue=`raw(1.0),
        ~duration=100.0,
        (),
      ),
      ~callback=_didFinish => (),
      ()
    );
  },
  render: _self => <Animated.View style=containerStyle />,
};
```

## Animations

Animated provides three types of animation types. Each animation type provides a particular animation curve that controls how your values animate from their initial value to the final value.

### Configuration

Below is the list of common configuration options applicable to all below animations.

#### toValue

```reason
~toValue: [ | `raw(float) | `animated(Animated.value))]
```

#### value

```reason
~value: Animated.value
```

#### useNativeDriver

```reason
~useNativeDriver: bool=?
```

#### iterations

```reason
~iterations: int=?
```

#### isInteraction

```reason
~isInteraction: bool=?
```

### Spring

Provides a simple spring physics model.

```reason
let animatedValue = Animated.Value.create(0.0);

let animation = Animated.Spring.animate(
  ~value=animatedValue,
  ~toValue=`raw(1.0),
  ~bounciness=5.0,
  (),
);

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

See available configuration below:

#### restDisplacementThreshold

```reason
~restDisplacementThreshold: float=?
```

#### overshootClamping

```reason
~overshootClamping: bool=?
```

#### restSpeedThreshold

```reason
~restSpeedThreshold: float=?
```

#### velocity

```reason
~velocity: float=?
```

#### bounciness

```reason
~bounciness: float=?
```

#### speed

```reason
~speed: float=?
```

#### tension

```reason
~tension: float=?
```

#### friction

```reason
~friction: float=?
```

#### stiffness

```reason
~stiffness: float=?
```

#### mass

```reason
~mass: float=?
```

#### damping

```reason
~damping: float=?
```

### Timing

Animates a value over time using easing functions.

```reason
let animatedValue = Animated.Value.create(0.0);

let animation = Animated.Timing.animate(
  ~value=animatedValue,
  ~toValue=`raw(1.0),
  ~duration=100.0,
  (),
);

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

See available configuration below:

#### easing

```reason
~easing: Easing.t=?
```

Easing function. See [Easing](/docs/easing.html) for available options.

#### duration

```reason
~duration: float=?
```

#### delay

```reason
~delay: float=?
```

### Decay

Starts with an initial velocity and gradually slows to a stop.

```reason
let animatedValue = Animated.Value.create(0.0);

let animation = Animated.Decay.animate(
  ~value=animatedValue,
  ~toValue=`raw(1.0),
  ~velocity=100.0,
  (),
);

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

See available configuration below:

#### velocity

```reason
~velocity: float
```

#### deceleration

```reason
~deceleration: float=?
```

## Composition

Animations presented in the previous section can be combined all together in many complex ways using the following API.

### parallel

```reason
let parallel:
  (array(CompositeAnimation.t), {. "stopTogether": bool}) =>
  CompositeAnimation.t;
```

Runs an array of animations in parallel.

#### Example

```reason
let fooValue = Animated.Value.create(0.0);
let barValue = Animated.Value.create(0.0);

let animation =
  Animated.parallel(
    [|
      Animated.Timing.animate(
        ~value=fooValue,
        ~toValue=`raw(1.0),
        ~duration=100.0,
        (),
      ),
      Animated.Timing.animate(
        ~value=barValue,
        ~toValue=`raw(0.0),
        ~duration=100.0,
        (),
      ),
    |],
    {"stopTogether": false},
  );

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

When `stopTogether` is set to `true`, `callback` passed to `CompositeAnimation.start` will get executed only once, after all animations within the array have finished. Otherwise, it may get executed many times. You should check for the value of `didFinish` boolean that is the first argument to the callback function.

### stagger

```reason
let stagger: (float, array(CompositeAnimation.t)) => CompositeAnimation.t;
```

Array of animations may run in parallel (overlap), but are started in sequence with successive delays.

#### Example

```reason
let fooValue = Animated.Value.create(0.0);
let barValue = Animated.Value.create(0.0);

let animation =
  Animated.stagger(
    50.0,
    [|
      Animated.Timing.animate(
        ~value=fooValue,
        ~toValue=`raw(1.0),
        ~duration=100.0,
        (),
      ),
      Animated.Timing.animate(
        ~value=barValue,
        ~toValue=`raw(0.0),
        ~duration=100.0,
        (),
      ),
    |],
  );

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

### delay

```reason
let delay: float => CompositeAnimation.t;
```

Helper function to delay execution of the animation. To be used with other Animated functions, as demonstrated at the below example.

#### Example

```reason
let barValue = Animated.Value.create(0.0);

let animation =
  Animated.sequence(
    [|
      Animated.delay(500),
      Animated.Timing.animate(
        ~value=barValue,
        ~toValue=`raw(0.0),
        ~duration=100.0,
        (),
      ),
    |],
    {"stopTogether": false},
  );

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

The above example will delay the `barValue` animation by 500 milliseconds.

### sequence

```reason
let sequence: array(CompositeAnimation.t) => CompositeAnimation.t;
```

Starts an array of animations in order, waiting for each to complete before starting the next. If the current running animation is stopped, no following animations will be started.

#### Example

See the example [in the previous section](#delay).

### loop

```reason
let loop:
  (~iterations: int=?, ~animation: CompositeAnimation.t, unit) =>
  CompositeAnimation.t;
```

Loops a given animation continuously, so that each time it reaches the end, it resets and begins again from the start.

You can specify the number of interations explicitly here or use [iterations](#iterations) property when defining animation.

#### Example

```reason
let fooValue = Animated.Value.create(0.0);

let animation =
  Animated.loop(
    ~animation=Animated.Timing.animate(
      ~value=fooValue,
      ~toValue=`raw(0.0),
      ~iterations=4,
      ~duration=100.0,
      (),
    ),
    (),
  );

Animated.CompositeAnimation.start(animation, ~callback=_didFinish => (), ());
```

## Animated.Value

### create

### add

### divide

### multiply

### diffClamp

### modulo

### interpolate

### setValue

### setOffset

### flattenOffset

### extractOffset

### addListener

### removeListener

### removeAllListeners

### stopTracking

### track

## CompositeAnimation

All animations created with [Animations](#animations) or [Composition](#composition) APIs are an instance of `CompositeAnimation`. This module can be used to start, stop and reset such created objects.

### start

```reason
let start = (CompositeAnimation.t, ~callback: Animation.endCallback=?, unit) => unit;
```

Starts an animation

### stop

```reason
let stop = (CompositeAnimation.t) => unit;
```

Stops an animation

### reset

```reason
let reset = (CompositeAnimation.t) => unit;
```

Resets an animation

## Easing

This module is exposed under `Animated` for historical reasons. Please see [`Easing`](/docs/easing.html) module instead.

```

```
