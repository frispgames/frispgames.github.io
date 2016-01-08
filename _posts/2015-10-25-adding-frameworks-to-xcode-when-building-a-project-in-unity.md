---
title:  Adding frameworks to Xcode when building a project in Unity.
date:   2015-10-25 11:11:00
description: Tutorial on how to write a post processor script to automatically add iOS frameworks when building an iPhone project in Unity.
keywords: [iOS frameworks, Unity, iPhone]
---

During the development of a game I was building with friends I needed to use the <a target="_blank" href="https://developers.google.com/admob/ios/download#downloadios">Google Mobile Ads Framework</a> so that I could serve ads from the <a target="_blank" href="http://www.google.com.au/admob/">Admob</a> network. Every time I built the project for the iPhone I would have to add the framework manually in Xcode then compile it for the iPhone. After about 30 builds it started to get a bit cumbersome so I decided to investigate ways to automate the process.

Most of the solutions I looked at suggested manually reading in the <a target="_blank" href="http://www.file-extensions.org/pbproj-file-extension">pbproj</a> file and then adding reference lines in the file, this seemed a bit low level. After spending half a day messing around with that solution I found out that Unity has it's own API for adding frameworks to Xcode here is the <a target="_blank" href="http://docs.unity3d.com/ScriptReference/iOS.Xcode.PBXProject.html">documentation</a>.

Now that I had some idea of how to add the frameworks via code I needed to figure out where I could run this code. In Unity there is a concept called editor scripting, basically it's where you create a class under the ```Editor``` folder in the root of your project and add callback methods to that class. The callback methods will get invoked during certain life cycle stages of the Unity editor. The callback I'm interested in is ```OnPostProcessBuild```.

In Unity when you add a framework file to the root of your project it will automatically copy the framework file into the Frameworks folder in iPhone project. The first step is to add the framework to the root of your project in Unity.

Here I define the class ```FrispAdsPostProcessor``` which adds the frameworks to Xcode:
{% highlight csharp %}

using UnityEngine;
using UnityEditor;
using UnityEditor.Callbacks;
using System;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using UnityEditor.iOS.Xcode;

public class FrispAdsPostProcessor {

  [PostProcessBuild(1500)]
  public static void OnPostProcessBuild(BuildTarget target, string path) {
    #if UNITY_IPHONE
      if (target != BuildTarget.iOS) return;

      string buildName = Path.GetFileNameWithoutExtension(path);
      string pbxprojPath = path + "/Unity-iPhone.xcodeproj/project.pbxproj";

      PBXProject proj = new PBXProject();
      
      proj.ReadFromString(File.ReadAllText(pbxprojPath));

      string buildTarget = proj.TargetGuidByName("Unity-iPhone");

      DirectoryInfo projectParent = Directory.GetParent(Application.dataPath);

      char divider = Path.DirectorySeparatorChar;

      DirectoryInfo destinationFolder =
        new DirectoryInfo(path + divider + "Frameworks");
    
      foreach(DirectoryInfo file in destinationFolder.GetDirectories()) {
        string filePath = "Frameworks/"+ file.Name;
        proj.AddFile(filePath, filePath, PBXSourceTree.Source);
        proj.AddFrameworkToProject (buildTarget, file.Name, false);
      }

      proj.SetBuildProperty(
        buildTarget, "FRAMEWORK_SEARCH_PATHS", "$(SRCROOT)/Frameworks"
      );
      proj.AddBuildProperty(
        buildTarget, "FRAMEWORK_SEARCH_PATHS", "$(inherited)"
      );
      proj.SetBuildProperty(
        buildTarget, "CLANG_ENABLE_MODULES", "YES"
      );

      File.WriteAllText(pbxprojPath, proj.WriteToString());
    #endif
  }
}
{% endhighlight %}

First I create a PBXProject instance for the ```Unity-iPhone``` project created by unity:

{% highlight csharp %}
string buildName = Path.GetFileNameWithoutExtension(path);
string pbxprojPath = path + "/Unity-iPhone.xcodeproj/project.pbxproj";

PBXProject proj = new PBXProject();

proj.ReadFromString(File.ReadAllText(pbxprojPath));
{% endhighlight %}

Then I add the framework files to the project as well as as the framework itself:

{% highlight csharp %}
string buildTarget = proj.TargetGuidByName("Unity-iPhone");

DirectoryInfo projectParent = Directory.GetParent(Application.dataPath);

char divider = Path.DirectorySeparatorChar;

DirectoryInfo destinationFolder = 
  new DirectoryInfo(path + divider + "Frameworks");

foreach(DirectoryInfo file in destinationFolder.GetDirectories()) {
  string filePath = "Frameworks/"+ file.Name;
  proj.AddFile(filePath, filePath, PBXSourceTree.Source);
  proj.AddFrameworkToProject (buildTarget, file.Name, false);
}
{% endhighlight %}

Next I add the ```Frameworks/``` folder to the search path:

{% highlight csharp %}
proj.SetBuildProperty(
  buildTarget, "FRAMEWORK_SEARCH_PATHS", "$(SRCROOT)/Frameworks"
);
proj.AddBuildProperty(
  buildTarget, "FRAMEWORK_SEARCH_PATHS", "$(inherited)"
);
{% endhighlight %}

Some of the classes in the <a target="_blank" href="https://github.com/googleads/googleads-mobile-unity">google modile ads unity library</a> require modules enabled, the following achieves this:

{% highlight csharp %}
proj.SetBuildProperty(
  buildTarget, "CLANG_ENABLE_MODULES", "YES"
);
{% endhighlight %}

Last but not least I need to apply all of these changes:

{% highlight csharp %}
File.WriteAllText(pbxprojPath, proj.WriteToString());
{% endhighlight %}

With the ```FrispAdsPostProcessor``` class, anytime I add a framework to the root of our project in Unity, it will automatically be added into the Xcode project when built. There are all sorts of other things you can do to the Xcode project after it has been built I recommend you take a look at the <a target="_blank" href="http://docs.unity3d.com/ScriptReference/iOS.Xcode.PBXProject.html">documentation</a>. You can find the up to date class mentioned on <a target="_blank" href="https://github.com/frispgames/frisp-ads-unity-asset/blob/master/Assets/Editor/FrispAds/FrispAdsPostProcessor.cs">github</a> as I have open sourced it.



