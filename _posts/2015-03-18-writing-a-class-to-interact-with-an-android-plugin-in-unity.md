---
title:  Writing a class to interact with an Android plugin in Unity.
date:   2015-03-18 12:51:00
description: A tutorial on how to create a C# class that can interact with an Android class for interfacing with the native android social media sharing libraries.
keywords: [Unity, Unity Asset, Android, Game Development, C#, Frisp Social, Frisp Social Unity Asset]
---

During the <a target="_blank" href="http://www.kiwiprogrammer.com/building-a-social-media-sharing-android-plugin-for-unity">previous post</a> I went through how to build an Android social media library for unity. This post will go through how I consumed the library in a C# script in Unity to be able to share a screenshot to Twitter or Facebook. Having the ability to create android libraries and use them inside unity gives you the power to do anything that the android SDK can do.

The first step is to add <a target="_blank" href="https://github.com/frispgames/frisp-social-unity-asset/tree/master/Assets/Plugins/Android">frisp-social.jar</a> under the ```Plugins/Android``` directory so that Unity uses this library when you build it for Android. If you don’t put the jar in this folder it will not load it for Android. I originally put it under ```Plugins/Android/FrispGames``` this will not work.

The next step is to write a C# wrapper class that encapsulates the communication between the android layer and the unity layer. The pattern for storing these type of classes in your Unity project is to put them into an ```Extensions``` folder at the root level of your project. For this class I am going to put it under: ```Extensions/FrispSocial/```.

Here is the code for the wrapper class ```Extensions/FrispSocial/FrispAndroidSocial.cs```:

{% highlight csharp %}
using UnityEngine;
using System;
using System.Collections;

#if UNITY_ANDROID && !UNITY_EDITOR
public class FrispAndroidSocial {
  private static readonly FrispAndroidSocial _singleton = 
    new FrispAndroidSocial ();

  private FrispAndroidSocial() {}

  public static FrispAndroidSocial Instance() {
    return _singleton;
  }

  public void ShareImage(String title, String text, Texture2D image) {
    var imageInBytes = image.EncodeToPNG();
    var base64Image = System.Convert.ToBase64String (imageInBytes);
    var socialClass = new AndroidJavaClass (
      "com.frispgames.frispsocial.FrispSocial"
    );

    using (socialClass) {
      socialClass.CallStatic (
        "shareImage", new String[]{title, text, base64Image}
      );
    }
  }
}
#endif
{% endhighlight %}

The ```#if UNITY_ANDROID``` block tells Unity to only make this class available if it is running on the android device. I implemented the singleton pattern for the wrapper class so that fetching an instance is as simple as calling ```FrispAndroidSocial.instance()```. I prefer to use the singleton pattern in Unity for wrapper classes as it makes the method calls thread safe.

ShareImage takes care of encoding the texture to a Base64 string and then sends it through to the java library. It does this by initializing an ```AndroidJavaClass``` which needs to be initialized with the package name and the class appended to it so in this case the package is ```com.frispgames.frispsocial``` and the class is ```FrispSocial```.

After fetching the class from the Android library I can now call methods on it. Here I call the shareImage method and pass through a title, text, and the Base64Image.

Whilst it seems simple, the whole process of creating an Android library and integrating it into Unity was quite painful. Debugging problems took a while as every time you needed to make a change you had to recreate the jar file, export an android project, run it on the simulator then hope that it worked. I much prefered the workflow for adding Objective-C libraries into Unity which will be the topic I will cover in my next post.

All of the code including _frisp-social.jar_ has been open sourced on our <a target="_blank" href="https://github.com/frispgames/frisp-social-unity-asset">company github account</a>.