---
title:  Writing a class to interact with an iOS plugin in Unity.
date:   2015-03-22 10:18:00
description: A tutorial on how to create a C# class that can interact with an objective-c class for interfacing with the native iOS social media sharing libraries.
keywords: [iOS, Objective-C, Unity, Sharing To Facebook, Sharing To Twitter, Game Development, Frisp Social, Frisp Social Unity Asset, Unity Asset]
author: michael
---

Now that I have <a target="_blank" href="http://www.kiwiprogrammer.com/building-a-social-media-sharing-ios-plugin-for-unity">built an iOS plugin</a> for sharing I need to implement a wrapper class for it in C# for Unity. This post will go through the wrapper class code that I put together to add the feature of sharing to Facebook or Twitter for a Unity game for iPhone. I will go into detail about how to make the Objective-C methods available to the C# wrapper class.

From the previous post I have the following files inside the ```Plugins/iOS/FrispSocial``` folder:

* FrispSocial.h
* FrispSocial.mm

Here we define the class ```Extensions/FrispSocial/FrispAppleSocial.cs```:

{% highlight csharp %}
using UnityEngine;
using System;
using System.Collections;
#if UNITY_IPHONE && !UNITY_EDITOR

using System.Runtime.InteropServices;

public class FrispAppleSocial : MonoBehaviour  {
  [DllImport ("__Internal")]
  private static extern void _Share (string text, string image);

  private static readonly FrispAppleSocial _singleton = new FrispAppleSocial ();

  private FrispAppleSocial() {}

  public static FrispAppleSocial Instance() {
    return _singleton;
  }

  public void ShareImage(String text, Texture2D image) {
    var imageInBytes = image.EncodeToPNG();
    var base64Image = System.Convert.ToBase64String (imageInBytes);
    _Share(text, base64Image);
  }
}
#endif
{% endhighlight %}

The Objective-C classes defined under the ```Plugins/iOS``` directory are statically linked into the executable created for the iPhone. This means that the ```_Share``` method defined in the Objective-C class exists in the executable. To get access to it we need to do an internal DLL Import of the method. The following code achieves this:

{% highlight csharp %}
[DllImport ("__Internal")]
private static extern void _Share (string text, string image);
{% endhighlight %}

Given I have imported the method I can now invoke it inside the ```ShareImage``` method with the appropriate parameters.

I found the process of working with iOS and Unity quite enjoyable and not as painful as working with Android. This might be because i’m working on an iMac and I have an iPhone device. In my next post I will go over the class I wrote for unifying the Android and iOS logic.

All of the code including the Objective-C classes has been open sourced on our <a target="_blank" href="https://github.com/frispgames">company github account</a>.
