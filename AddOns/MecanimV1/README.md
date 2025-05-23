# Mecanim V1

This is the Mecanim runtime that was formerly included as part of the Latios
Framework as part of the Mimic module.

Note: The Mecanim module has known issues and is not considered suitable for
production. It is merely provided as a quick-start to get familiar with
Kinemation and also serves as a potential learning resource for how to leverage
Kinemation’s APIs. If you desire a production-ready Mecanim implementation and
are willing to invest in its existence, please consider helping develop V2!

## Getting Started

**Scripting Define:** LATIOS_ADDON_MECANIM_V1

**Requirements:**

-   Requires Latios Framework 0.10.0 or newer

**Main Author(s):** Sovogal

**Additional Contributors:**

-   Dreaming I’m Latios

**Support:** This add-on is legacy and is not officially supported. You are
still welcome to contribute to it, but you are highly encouraged to contribute
to Mecanim V2 instead.

### Installing

Add the following installer lines to your bootstrap after Kinemation:

```csharp
// In LatiosBakingBootstrap
Latios.Mecanim.MecanimBakingBootstrap.InstallMecanim(ref context);

// In LatiosBootstrap
Latios.Mecanim.MecanimBootstrap.InstallMecanim(world);
```

The Mecanim runtime will only take effect when an Animator is baked with a valid
*Animator Controller*. If this field is left null, then it is assumed the
character is a static pose or is being driven by usage of the low-level
Kinemation APIs.

The Mecanim runtime supports both QVVS and Unity Transforms and supports both
exposed and optimized skeletons.

**Important:** Currently, the runtime only supports one animation layer.

## MecanimAspect

The `MecanimAspect` is an `IAspect` which allows you to modify the runtime
parameters to drive the state machine. The parameter manipulation API mimics the
classical API, but with a few twists.

The easiest to use API is the one where you pass in a string:

```csharp
mecanimAspect.SetFloat(“strafe”, strafeValue);
```

While the easiest, it is also the slowest. It may be sufficient in many cases,
but you may prefer instead to use one of the hash APIs.

The hash APIs diverge slightly from the classical Unity APIs because they add an
extra bool argument. This argument specifies whether to use the classical Unity
generated hash, or to use a hash of the parameter name computed by
`FixedString64Bytes.GetHashCode()`.

This method is slightly faster, but for the best performance, you want to bake
the parameter index and use that directly.

### Baking the Parameter Index

Inside a Baker, when using the `Latios.Kinemation.Authoring` namespace, use the
`IBaker` extension method `FindAnimatorController()` and pass in the Animator
component’s `RuntimeAnimatorController`. You must wrap your baker in an `#if
UNITY_EDITOR` block.

That method will return an Editor-exclusive representation of the
`AnimatorController`, from which you can retrieve the `parameters`.

Another extension method is provided for parameters called `TryGetParameter()`
which will search through the parameters for the specified name, and if found,
return the parameter index as an out parameter. You can save this index inside
an `IComponentData` or blob asset for use at runtime.

### Manual Crossfades

If you know the name, hash, or index of a state in the state machine, you can
cross-fade into that state with the `CrossfadeInFixedTime()` method. This new
state will immediately become the state machine’s current state, and an inertial
blend will be started for the specified duration. Crossfades always use inertial
blending.

## Events

The Mecanim runtime is able to detect clip events and broadcast them to a
`DynamicBuffer<MecanimActiveClipEvent>`. Each event contains a `nameHash` which
can be computed by `FixedString64Bytes.GetHashCode()` of the event name. It also
contains the `parameter` which is the integer parameter from the clip’s import
settings. If you need the full event details, the `clipIndex` and `eventIndex`
can be used to index into the `MecanimController` component’s `clips` blob
asset.

## Other Mecanim Runtime Quirks

The Mecanim runtime does not aim to be a perfect recreation of Unity’s classical
implementation. In some cases, there may be frame or blending differences. For
example, when classical Unity interrupts a transition, it saves the pose at the
point of interruption and blends between that static pose and the target
interrupting state. In contrast, Mimic elects to immediately transition to the
target interrupting state and trigger a new inertial blend to smooth out the
motion.

## Blend Shape Animation

Blend shape animation is a highly experimental feature which can be enabled via
LATIOS_ADDON_MECANIM_EXPERIMENTAL_BLENDSHAPES scripting define. There are known
issues with the animations not matching classical GameObjects, though the
reasons remain unknown at the time of writing this. We would appreciate any help
investigating the cause of these issues!
