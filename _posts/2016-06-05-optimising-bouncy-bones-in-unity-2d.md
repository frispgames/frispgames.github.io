---
title:  Optimising Bouncy Bones in Unity 2D
date:   2016-06-05 11:33:00
description: Optimising Bouncy Bones and other optimisation techniques in Unity 2D.
keywords: [Unity, 2D, Game Development, Optimisation, Bouncy Bones, Build Sizes, CPU Optimisations, Memory Optimisations]
author: calvin
---

Here are some optimisation techniques we employed for Bouncy Bones after running into performance issues on older devices and build sizes being way too big ( originally 250 MB on the App Store prior to optimisations...). We managed to reduce our build size on the App Store to 180 MB after applying these changes.

**It is very important to use Unity's profiling tool on the device rather than in the Unity editor because this will give you a truer representation of performance..**

So after investigating around the net for optimisation best practices and profiling BouncyBones on devices with the Unity Profiler these are the optimisations we made:

##### CPU OPTIMISATIONS:

+ Overhead - Removed Fix errors/exceptions/ get rid of debug statements - these created unnecessary overheads in the CPU cycle.
+ Physics2D.FixedUpdate: We noticed up to 50% of the CPU was being used each frame to calculate physics collisions. Our game is very collision intensive and so by correctly setting our layers properly in our Collision Matrix we managed to avoid any unnecessary collisions and minimised the amount of collision listeners.
+ The water is made of many many small meshes to imitate spring like behaviour with object collisions. We heavily reduced the amount of meshes making up the water to a point where it looked good but didn't cause such a load on collision calculations.
+ It is important to be careful of the types of colliders you use on your game objects; PolygonColliders are more expensive to test than Circle and Box Colliders. We switched to these where we could to make a trade off on performance (reasonable approximations - box and circle) and 100% accurate Polygon colliders.
+ Although not an optimisation.. We ended up Increasing the Fixed Timestep (Physics timestep) to increase the number of times the physics simulation was being updated because we noticed that rendering wasn't as smooth as it could be. This was because the physics system was not updating the game objects' position fast enough on each frame.
+ Quality Settings - The graphics quality Unity will render. Turn off Anti-aliasing, turn off Anisotropic textures, turn Soft Particles off, disable Shadows and leave VSync on since it is default enabled for iOS anyway.

##### MEMORY OPTIMISATIONS:

+ Need power of two (POT) textures - If you want to compress your textures, the textures need to be POT. This will drastically reduce the memory footprint of your textures in your game. Unity will do POT scaling internally for you but this will cost additional memory and CPU usage. Turn off MipMaps - no need to have these in 2D.
+ Set the Max Size of your sprite assets correctly as this will reduce the size of your textures.
+ Set the texture compression on a texture by texture basis - this will come down to trial and error of what works best for each texture (does the texture require an alpha channel/ how important is the colour information?). Start with compressed  PVRTC 4 bit then work towards 16 bit RGBA then finally RGB 24 bit / RGBA 32 bit. Obviously the richer the colour information required the more bits you will need, so try 16 bit RGBA or RGBA 32 bit. **NOTE: only POT textures can be compressed to PVRTC format.**
+ Need to use Sprite Atlasing - Managing one sprite map is easier for the GPU than individually loading and removing individual assets. [Unity Packing Tags](http://docs.unity3d.com/Manual/SpritePacker.html) does this internally already. Make sure to group sprites in your atlases that are used similarly in the scene otherwise you will not notice that much benefit.

**IN HINDSIGHT**, with Bouncy Bones being our first game, our biggest mistake was not building our assets with POT sizing from the beginning. When it came to optimising later we were unable to compress our assets since you can only compress POT textures. This made our memory footprint huge and our build sizes ridiculously big.. Make sure to compress your assets where possible. We tried using Unity Packing Tags to make POT Atlas sheets to fix this problem where we could but this only worked for smaller things like all of Widdy's animated skeleton pieces because a lot of our background assets were too big to fit into Atlas sheets reasonably anyway. This is where 70% of our texture size came from.. Without getting into too much detail, we were way too far down a rabbit hole to go back and fix all our background sizes. With testing we found that Bouncy Bones still performed OK without making this optimisation anyway. So learn from our mistakes!

**Some references I looked at that I found useful:**

1. [http://docs.unity3d.com/Manual/SpritePacker.html](http://docs.unity3d.com/Manual/SpritePacker.html)
2. [http://www.katsbits.com/tutorials/textures/make-better-textures-correct-size-and-power-of-two.php](http://www.katsbits.com/tutorials/textures/make-better-textures-correct-size-and-power-of-two.php)
3. [http://docs.unity3d.com/Manual/ReducingFilesize.html](http://docs.unity3d.com/Manual/ReducingFilesize.html)
4. [https://unity3d.com/learn/tutorials/modules/intermediate/physics/physics-best-practices](https://unity3d.com/learn/tutorials/modules/intermediate/physics/physics-best-practices)
6. [http://sicklebrick.com/?p=411](http://sicklebrick.com/?p=411)
7. [http://team2bit.com/wordpress/2015/04/12/mobile-frame-rate-and-memory-issues-in-unity/](http://team2bit.com/wordpress/2015/04/12/mobile-frame-rate-and-memory-issues-in-unity/)
8. [http://team2bit.com/wordpress/2015/05/01/2d-mobile-game-performance-in-unity-part-2/](http://team2bit.com/wordpress/2015/05/01/2d-mobile-game-performance-in-unity-part-2/)
9. [http://biobeasts.artix.com/unity-memory-management/](http://biobeasts.artix.com/unity-memory-management/)
10. [http://biobeasts.artix.com/unity-2d-texture-optimization/](http://biobeasts.artix.com/unity-2d-texture-optimization/)
11. [http://gamedev.stackexchange.com/questions/102837/unity-2d-graphics-optimization-for-mobile-devices](http://gamedev.stackexchange.com/questions/102837/unity-2d-graphics-optimization-for-mobile-devices)
12. [https://divillysausages.com/2016/01/21/performance-tips-for-unity-2d-mobile/](https://divillysausages.com/2016/01/21/performance-tips-for-unity-2d-mobile/)