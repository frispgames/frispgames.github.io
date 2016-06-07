---
title:  Best practices for handling different aspect ratios and screen resolutions in Unity 2D for mobile games.
date:   2016-05-25 18:10:00
description: A summary of best practices in Unity 2D to handle different aspect ratios and screen resolutions on mobile devices.
keywords: [Unity, 2D, Game Development, Aspect Ratio, Resolution, Asset Bundles, Dynamically loading assets, Unity Resource folder]
author: calvin
---

After spending quite a while investigating options for handling the multitude of different aspect ratios and screen resolutions on mobile devices it became clear that there really isn't a standard or 'magic bullet' way of doing things in Unity for 2D games. So I'm going to share with you a summary of the best practices that I found along the way and then tell you what we used for Bouncy Bones. At the time of writing, Unity 5.3 is the current version and so best practices are based on this version of Unity.

When choosing a way to tackle the resolution independence problem there is a fork in the road where you can choose either to go with a 3rd party toolkit or stay native. It is up to you to review the 3rd party tools out there vs. staying native for your game. I believe staying native is the best way to go because then you don't have any dependencies on a third party and as Unity changes this will be less of a headache to manage. I'm going to discuss the options you have for staying native in this blog post.

##### You have two options when staying native:

1.  Scaling your graphics up and down with a single set of assets built for a target reference resolution and target aspect ratio.
2.  Dynamically loading specific assets at runtime for the native resolution on the current device.

#### Building for a reference resolution

This means you are either upscaling low resolution assets which will look pixelated on high resolution devices, or downscaling high resolution assets which look great but incurs a huge memory performance hit on older devices. Using ultra high resolution assets is a no brainer in this case. So this means deciding on the highest target resolution of the device you want to cater for and setting this as your reference resolution. Some target resolutions for devices you might to keep in mind: iPhone 6plus is 1920x1080 and the new Galaxy S7 2560x1440 (Both with 16:9 aspect ratio).

Below I am going to list the problems you are going to face if you build for a target resolution and target aspect ratio. These problems are not independent and can occur together, it depends on the current device and how it differs from your targets.

### Problem number 1: Different aspect ratio to your target aspect ratio

There is no way to solve this problem **100%**. For example if my game's target resolution is 16:9 and then my game plays on an iPad with a 4:3 aspect ratio then less width is going to be shown. This may or may not be a problem for your game play and is something you will need to consider!

![iPad (4:3 aspect ratio) view of Bouncy Bones - built for 16:9](/assets/blog-images/2016-05-25/ipad_view_from_iphone6s.jpg)

As you can see, this is what Bouncy Bones which is designed for 16:9 aspect ratio looks like when played on an iPad with 4:3 aspect ratio. The masked out area is what you miss when playing on an iPad.

If you want your game to **look exactly the same** at each variation of aspect ratio the only way to solve this is to use letter-boxing or pillar-boxing (black bars appear). This is what happens when you watch an old movie on a widescreen TV.

![Letterboxing 16:9 widescreen aspect ratio into iPad 4:3 aspect ratio](/assets/blog-images/2016-05-25/16_9_aspect_ratio_in_ipad_display_2.jpg)

Here you can see what Bouncy Bones would look like with letter-boxing applied for an iPad display.

The other option is to scale the 16:9 display into the 4:3 display to fit everything. **Don't do this**, as you can see in the image below this will stretch and distort your assets horribly as you are non-uniformally scaling in both the X and Y scale.

![Un-uniform scaling of 16:9 into 4:3 aspect ratio](/assets/blog-images/2016-05-25/6s_into_ipad_1024x768_scaled.jpg)

### Problem number 2: Different screen resolution to your target resolution

You have more options here than **problem number 1**. Here is a bit of Unity background to help support what I am going to say next:

**By default Unity scales your sprite assets by the height.** In unity the height of the viewport is dictated by two variables:

1.  The pixels per unit of your sprite assets (this is normally fixed to one value across all your sprite assets to keep a consistent scale). With Bouncy Bones we used the default import value of 100 pixels per unit.
2.  Orthographic size of the Camera - Viewport height in unity units / 2.

By controlling the orthographic size, either by fixing it's value or changing it at runtime through scripts we are able to control **the ratio of source pixels to device pixels when your game is played**. This is very important as this directly impacts the quality of your graphics and the scaling of your sprites.

**Let's discuss each of these options:**

### Change the orthographic size based on the current device's resolution

**Pixel perfect display** boils down to the ratio of source pixels from your asset sprites to the current device's pixels. A 1:1 ratio or a **ratio where the source pixels divide evenly** into the device pixels (factor of 2) will create a pixel perfect display.

For example: If I build my game for 960x640, my game will be pixel perfect to play on a device with 480x320 because 2 source pixels will fit into one device pixel with a ratio of 2:1.

Here is a [good resource](https://youtu.be/rMCLWt1DuqI?t=31m12s) where a Unity developer from the Unity 2D team shows you how you can change the orthographic size of the camera at runtime.

**For direct reference here is the same script from that video that you can use to change the camera's orthographic size at runtime (Attach this script to your main camera game object):**

{% highlight C# %}
using UnityEngine;

public class PixelPerfectCamera : MonoBehaviour {

    public float pixelsToUnits = 100;
    private Camera camera;

    void Awake () {
        camera = Camera.main;
    }
    
    void Update () {
        camera.orthographicSize = Screen.height / pixelsToUnits / 2;
    }
}
{% endhighlight %}

Here is an image of various iOS devices with different resolutions to help aid the explainations below; the square tile is 100x100px and the pixels per unit has been set to 100 so that the tile is exactly 1 Unity unit by 1 Unity unit.

![Comparison of different iOS devices with and without chaning camera orthographic size](/assets/blog-images/2016-05-25/orthographic_size_comparison.png)

As you can see when the orthographic size changes at runtime to adjust to the current device resolution it maintains the original / target scale of your assets so that they will always appear the same size **(Unity will not scale your assets)**. This will keep the 1:1 source to screen pixels ratio. Consequently the assets stay the same size regardless of resolution. This might be a problem for you if there is a large gap between low resolution and high resolution device sizes. As you can see/imagine the 100x100px square tile will take up a lot more space on the iPhone 3GS than it does on the iPad 2.

### Not change the orthographic size

If you want your sprites to remain the same apparent size regardless of screen resolution, then simply set the orthographic size to a fixed value according to your target resolution and leave it there regardless of the device screen resolution. As seen in the above diagram, this will mean your sprites' pixels will scale up and down to the current device and there is high chance that this will create slight pixelation in the display (you will not have 1:1 source to screen pixels).

##### Explanation:

**For example:** say I have built my game for the target resolution of 960x640. If my game plays on a device with a native resolution of 640x480...

**HEIGHT** - 960 pixels of my source pixels needs to fit into 640 device pixels with a ratio of 960/640 = 1.5. 1.5:1 source pixels to device pixels. Shrinking 1.5 source pixels to fit into 1 device pixel creates distortion. So be wary!   
**WIDTH** - The width displayed is determined by the aspect ratio of the device.

###Conclusion:

There is a trade off whether you want pixel perfect sprites (1 source pixel == 1 display pixel) vs. sprites to remain the same apparent size regardless of screen resolution because you can't have both when designing for a target resolution.

#####Pros vs Cons of building for a target resolution and target aspect ratio:

**Pros:**

+ A lot simpler than the other scripting solutions listed below using sprite switching.
+ Unity does a really good job scaling high definition assets to make them look nice.

**Cons:**

+ Ultra high resolution graphics will incur a huge performance hit on older devices (low frame rates) because a lot more more memory is consumed to scale down larger graphics
+ Trade off of pixel perfect display vs loss of relative sprite sizing to device resolution (can't have both)
+ Larger build sizes

### Dynamically loading assets (SD, HD, UD)

If you want more control than simply upscaling or downscaling graphics you'll need to write scripts for substituting in different sprite versions (SD, HD, UD) depending on the device's native resolution since there is no native GUI solution in unity.

There is currently two options for substituting different sprite versions depending on device resolution; Using Unity asset resource folder or AssetBundle variants. There is pros and cons for each which I am going to list:

##### Unity asset resource folder

The Resources folder is a special folder which allows you to access assets by file path and name in your scripts without them having to be part of the scene. This allows for loading resources at runtime without them being loaded into memory until requested. This is how we can load SD (standard definition), HD (high definition), UD (ultra-high definition) sprites dynamically based on the resolution of the device. In the <a target="_blank" href="https://youtu.be/rMCLWt1DuqI?t=2m25s">same video above</a> there is a tutorial to using the resources folder to load specific sprite types dynamically. [This forum thread](http://forum.unity3d.com/threads/working-multi-resolution-sprite-sheets.274683/) also shows another way of implementing it with scripts.

**Pros:**

+ Pixel perfect sprites without Unity scaling up or down assets.
+ Better support for low-end and older devices with memory limitations.

**Cons:**

+ All of the assets in the resources folder are always included in your build which could mean really large builds if you are not careful.
+ Switching between asset versions (HD, SD, UD) incurs a large memory hit because it requires both asset versions to be loaded into memory.

##### AssetBundle variants

An Asset Bundle is an external collection of assets that exist outside of your project build and can be loaded on demand by your application. This allows you to stream in content for switching sprites dynamically based on resolution by having a SD, HD and UD asset bundle. To get more of an introduction to Unity's asset bundles, [check out this link](https://unity3d.com/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager) from Unity's tutorial pages. There was a great tutorial / live training session done by Unity [here](http://unity3d.com/learn/tutorials/modules/intermediate/live-training-archive/unity5-asset-bundles) but for some reason it is not working anymore :(((!

**Pros:**

+ Smaller build sizes since assets in the bundles are not part of the project build.
+ Allows us to create platform and device specific asset bundles to target native resolutions on devices.
+ No longer requires Unity pro with version 5.0 +.

**Cons:**

+ Requires an internet connection to download asset bundles if you are distributing from a remote server (asset bundles can be pre-cached and be part of the game build but this really defeats the point of them).

##### What Unity suggests right now?

**Solution:** AssetBundle Variants

From reading through a number of forum threads the feeling I got from unity was that due to the huge number of devices, aspect ratios and resolutions, it is not practically possible to automate resolution independence for every possible situation and every possible game in a user interface without cutting corners. Scripting provides this level of flexibility needed. They want to make things easier down the road, but for what I know for 5.3 this is what you have to work with. It is a top priority for the 2D team on the list of improvements and that it is a common problem that people raise. Watch this space!

### What we did for Bouncy Bones:

Here is a selection of screenshots to show how the game plays across different resolutions and aspect ratios:

![Comparison of Game Screenshots across iOS devices](/assets/blog-images/2016-05-25/game_screenshot_comparisons.png)

+ We built Bouncy Bones with a target resolution in mind. For the highest possible resolution we wanted to support at the time of building Bouncy Bones was the Samsung Galaxy S7 with a resolution of 2560x1440. So this is the resolution that we targeted when building our background assets.
+ We built for the widest possible aspect ratio which is 16:9 and just display less width for devices with a smaller aspect ratio (4:3, 16:10 etc.). We feel that the 16:9 aspect ratio targeted the most popular mobile phones on the market and feel that gameplay is not too significantly impacted when less width is displayed with smaller aspect ratios. Obviously it is harder and a slight disadvantage because you have less time to react to obstacles and projectiles that appear but the slight difference in game play isn’t enough to merit a reduction in user experience by adding black bars.
+ Our sprites maintain the same relative height across devices because we fix the orthographic size. The only thing that changes is the width of the game displayed and because we targeted the widest aspect ratio this works perfectly for us.
+ The overhead of a larger memory footprint created by bigger assets from our target resolution is an OK tradeoff with the power of the majority of today's devices getting better and better and that older devices with standard definition resolutions (SD) are gradually being phased out.
+ In practice, pixel perfect displays are very hard to achieve across the variety of screen resolutions out there. For Bouncy Bones and the lack of native support in Unity meant that the cost to get this working perfectly vs. benefit to handle all screen resolutions was very small (80/20 principle). In testing we found that when Unity scaled down our assets they looked very good but not perfect when rendered at any size less than or equal to the original pixel size of the sprite.
+ With all this being said, a problem we faced was that our our build sizes became too big with the ultra high definition assets. So a tradeoff had to be made between asset sizing and quality. We lowered the max size of the textures to reduce the build size and tested it on the devices to make sure it still looked good. There is no best answer for these optimisations because what might work for Bouncy Bones might not work for you. So just experiment and figure out works for you.

### Conclusion:

Hopefully this sheds some more light on the different ways to handle different aspect ratios and screen resolutions in Unity for 2D games. It is a very complicated problem to solve and is very situational and this is why Unity has not built any native support. To reiterate what I said in the introduction; there is no magic way to solve this problem for every game and it is up to you to decide what is important to you when designing your game. Good luck!

##### Where to next- Handy references!

1. [http://v-play.net/doc/vplay-different-screen-sizes/](http://v-play.net/doc/vplay-different-screen-sizes/)
2. [http://www.third-helix.com/2012/02/05/making-2d-games-with-unity.html](http://www.third-helix.com/2012/02/05/making-2d-games-with-unity.html)
5. [http://www.reddit.com/r/Unity3D/comments/1sp864/handling_ios_android_and_desktop_aspect_ratios/](http://www.reddit.com/r/Unity3D/comments/1sp864/handling_ios_android_and_desktop_aspect_ratios/)
6. [http://gamedev.stackexchange.com/questions/79546/how-do-you-handle-aspect-ratio-differences-with-unity-2d](http://gamedev.stackexchange.com/questions/79546/how-do-you-handle-aspect-ratio-differences-with-unity-2d)
7. [http://www.reddit.com/r/Unity2D/comments/2qii4h/best_practices_for_creating_art_assets_on/](http://www.reddit.com/r/Unity2D/comments/2qii4h/best_practices_for_creating_art_assets_on/)
9. [http://forum.unity3d.com/threads/working-multi-resolution-sprite-sheets.274683/](http://forum.unity3d.com/threads/working-multi-resolution-sprite-sheets.274683/)
10. [https://www.youtube.com/watch?v=rMCLWt1DuqI](https://www.youtube.com/watch?v=rMCLWt1DuqI)
11. [https://unity3d.com/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager](https://unity3d.com/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager)





