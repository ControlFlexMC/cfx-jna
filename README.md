# ControlFlex JNA

Bundles [JNA](https://github.com/java-native-access/jna) for **Forge 1.7.10**, the
one Minecraft version that ships no JNA of its own. Drop the jar in `mods/` and
anything that reaches a native library through JNA works without editing the
launcher's class path.

This mod contains no gameplay content and exposes no API. It exists so that other
mods do not each have to bundle their own copy.

## Why this exists

Forge adds every jar in `mods/` to `LaunchClassLoader`, so a jar that carries
`com.sun.jna` is enough to satisfy mods that need it. That is all this mod is: JNA
plus an empty `@Mod` class, so Forge registers it cleanly instead of warning about
a jar with no mod class.

The alternative — every mod embedding its own JNA — does not produce duplicate
classes (the class loader caches the first one it finds and keeps using it), but it
does make *which version wins* depend on jar ordering. A single shared provider
removes that ambiguity.

## Which versions

| Minecraft | Needs this mod? |
|---|---|
| 1.7.10 | **Yes** — no JNA in the version JSON |
| 1.8.9 – 1.16.5 | No — ships JNA 3.4.0 / 4.4.0 |
| 1.17 and later | No — ships JNA 5.8.0 or newer |

`acceptedMinecraftVersions` is pinned to `[1.7.10]`, so Forge refuses to load it
anywhere else. That is deliberate: on 1.8.9+ the game already provides JNA, and a
second copy only makes the winning version depend on file ordering.

## Install

1. Put `cfx-jna-<version>-mc1.7.10.jar` in `.minecraft/mods/`.
2. Start the game.

Nothing to configure — no config file, no keybind, no command.

## Verifying it works

The mod logs one line during pre-init:

```
[cfxjna] ControlFlex JNA 1.0.0 - bundles JNA 5.12.1; com.sun.jna.Native visible: true
```

`com.sun.jna.Native visible: false` means the classes are not reaching the class
loader, which would be a bug in this mod rather than in whatever is trying to use
JNA.

## What it bundles

JNA 5.12.1, unmodified, including its native `jnidispatch` for every platform JNA
supports. Only the core artifact is included; `jna-platform` is not, because
ControlFlex does not use it.

`jna.nosys`, `jna.boot.library.path` and any class-transformer exclusions are
**not** set here. Those belong to whoever loads the native library, and setting
them globally would change the bootstrap order for every other JNA consumer in the
pack.

## Building

Requires JDK 8 or newer (the jar targets Java 8) and network access on the first
build.

```bash
./gradlew build          # jar + verifyJar
./gradlew jar            # build/libs/cfx-jna-<version>-mc1.7.10.jar
```

`verifyJar` fails the build if the jar is missing `mcmod.info`, the mod class, the
JNA classes or the `jnidispatch` natives, or if any Forge/Minecraft class leaked
into it.

## License

This repository's own code is Apache-2.0 — see [LICENSE](LICENSE).

JNA is © the java-native-access project, dual-licensed Apache-2.0 OR LGPL-2.1. It is
redistributed here unmodified under Apache-2.0; the full text is in
[LICENSE_JNA](LICENSE_JNA) and travels inside the jar as `META-INF/AL2.0`.
