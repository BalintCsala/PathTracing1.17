# ❗ <ins>Before you try it out, read this!</ins> ❗

This shader is WIP, therefore it lacks many features, including complete support for some of the blocks and all entities. If you want to try it out (although it's not necessary), play in a void or superflat world with the following gamerules:

```
doMobSpawning false
doTileDrops false
```

(You'll want to avoid any lava. If you see random blocks popping into existence and back, it's either lava or entities)

# Vanilla pudding tart - Path tracing shader for vanilla Minecraft 1.17+

## Using this resource pack

Contrary to the name, this pack doesn't have any custom textures (mostly because I'm bad at them), so it uses the vanilla textures. Since those don't have any of the required properties for most of the features (reflections, emission, etc.), they won't show up. To solve this, use [Tart Tin](https://github.com/BalintCsala/TartTin). This requires a LabPBR compatible resource pack, some can be found [here](https://github.com/rre36/lab-pbr/wiki/Resource-Packs).

## Features

- Path traced global illumination
- Shadows
- Reflections
- Emission (= light sources work as they should)
- Configurable F0, metallicity and roughness factors

## How to install

This shader only works if the resolution of your monitors exceeds or matches 1024x768 in both directions.

1. Press the green `⤓ Code` button and select "Download ZIP"
2. Extract the content of the ZIP file to your resource pack folder
3. Start the game and enter into a world 
4. Go into video settings and set Graphics to _Fabulous!_
5. Go into resource packs and enable the path tracing resource pack. Make sure to disable anything else, even if they don't have shaders.

## Adept mode

> It's dangerous to go alone! Take this.

The current view distance is pretty bad (44 blocks in the 3 cardinal directions). This is sadly the limit of the 1024x705 (windowed 768p) minimum resolution I chose.

I can't increase this without removing support for a bunch of people, so if you have a larger screen (which you most likely do) and want better view distance (which you also likely want), you'll need to edit the code a bit.

> Whenever I mention the resolution in the following paragraph, I mean the actual resolution the game is running at. This will only match the monitor resolution when the game is in full screen. You can find the actual resolution on the F3 debug screen in-game on the right side.

The maximum viewable area width (2 x view distance) `v` can be calculated using the following equation, where Sx and Sy are the screen width and height in pixels and `floor` gives you the integer part of a decimal (`floor(5.6) = 5`):

```
floor(Sx / v) * floor(Sy / v) >= v
```

A rough approximation can be found quickly by multiplying the width and height of the screen and taking the third root. This will be the maximum value you could reach in an ideal scenario, but the actual value will probably be less.

For instance:
A 4k screen (3840x2160) has an area of 8294400 pixels, so the theoretical maximum view area width is 202, but the largest value that satisfies the inequality is 196, therefore the actual value is 196.

```
floor(3840 / 196) = 19
floor(2160 / 196) = 11
11 * 19 = 209 >= 196
```

Some values for common screen resolutions (all of these are for fullscreen):

- **resolution: v**
- 1366x768 (768p): 98
- 1600x900: 112 (This is a case where actual and max values match)
- 1920x1080 (1080p): 120
- 2560x1080 (ultrawide 1080p): 135
- 2560x1440 (1440p): 150
- 3840x2160 (4k): 196
- 7680x4320 (8k): 312 (Maybe the 2 fps you'll be getting at this resolution isn't worth it)

Once you have `v`, you need to edit some files. Go into the folder of the resource pack (`%appdata%/.minecraft/resourcepacks/<shader name>/`) and find the files

- `assets/minecraft/shaders/include/utils.glsl`
- `assets/minecraft/shaders/program/raytracer.fsh`

And edit the following 2 lines in **both** files according to v (These should be near the top):

```glsl
const vec2 VOXEL_STORAGE_RESOLUTION = vec2(1024, 705);  // This should be the screen resolution you used earlier
const float LAYER_SIZE = 88;                            // This should be "v"
```

You should also edit the following line in raytracer.fsh:

```glsl
const int MAX_STEPS = 200; // Set this to roughly 2-3 times "v"
```

The shader should work at this point with the increased view distance.

## Expert mode

> Textures are now generated, please refer to [Tart Tin](https://github.com/BalintCsala/TartTin)

## Pro mode

> Only do this if you know what you are doing. Good rule of thumb: If you don't know if you'll know what you will be doing, you won't.

A couple of values can be configured in `assets/minecraft/shaders/program/raytracer.fsh` for a better effect. Each of these has a huge impact on performance, so make sure you don't burn out your graphcs card.

A description of each of these values is included in the following code block:

```glsl
// This determines how many steps (approx. blocks) the GI ray will go through to check bounce lighting (complexity: O(N))
const int MAX_GLOBAL_ILLUMINATION_STEPS = 10;
// This will determine how many bounces a GI ray is allowed to do to calculate the light level at a pixel (complexity: O(N^2))
const int MAX_GLOBAL_ILLUMINATION_BOUNCES = 3;
// This will determine the amount of reflection bounces a ray will do, if you increase it, mirror rooms will be better (complexity: O(N^˘3))
const int MAX_REFLECTION_BOUNCES = 10;
// The color of the sun multiplied by intensity
const vec3 SUN_COLOR = 1.0 * vec3(1.0, 0.95, 0.8);
// The color of the sky multiplied by intensity. This also affects ambient lighting
const vec3 SKY_COLOR = 1 * vec3(0.2, 0.35, 0.5);
// The maximum strength of emissive materials, larger values will result in coarser individual steps, but larger maximums
const float MAX_EMISSION_STRENGTH = 5;
```

## Examples

These images were taken without any noise reduction. Looks less noisy in-game.

![example1](images/gi-example1.png)

![example2](images/gi-example2.png)

![example3](images/gi-example3.png)
