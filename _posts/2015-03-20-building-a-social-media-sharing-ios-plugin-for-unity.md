---
title:  Building a social media sharing iOS plugin for Unity.
date:   2015-03-20 15:20:00
description: A tutorial on how to build an objective-c class that provides the functionality of native sharing on iPhone devices for Unity, this supports Facebook and Twitter.
keywords: [Objective-C, Unity, Game Development, Unity Asset, Frisp Social, Frisp Social Unity Asset, Sharing On Facebook, Sharing On Twitter]
---

After <a target="_blank" href="http://www.kiwiprogrammer.com/writing-a-class-to-interact-with-an-android-plugin-in-unity/">creating an Android library</a> for sharing, I needed to support iPhone too. Unity doesn’t support native sharing for iPhone, so I had to create an Objective-C plugin that can be used in Unity. Unlike Android, the files don’t need to be packaged, they just need to be added into the Unity project. One of the nice things about working with Objective-C in Unity, is that the files can be symlinked into the xcode project from Unity. This means when you make a change in xcode the same change is made in the Unity project.

The folder structure for including Objective-C files is similar to Android but it allows you to define your own folder structure. Everything needs to live under the ```Plugins/iOS``` folder. For the plugin I’m going to store the files under ```Plugins/iOS/FrispSocial```.

In Objective-C you need to define a header file for all of your classes so I first defined the header file which I called ```Plugins/iOS/FrispSocial/FrispSocial.h```:

{% highlight objectivec %}
@interface FrispSocial : NSObject 
+ (id) instance;
- (void) share:(NSString*)text media: (NSString*) media;
@end
{% endhighlight %}

The next step is to define the class that is going to take care of sharing to Facebook or Twitter which is called ```Plugins/iOS/FrispSocial/FrispSocial.mm```:

{% highlight objectivec %}
#import "FrispSocial.h"

@implementation FrispSocial

static FrispSocial * _instance = [[FrispSocial alloc] init];

+ (id) instance {
    return _instance;
}

- (void) share:(NSString *)text  media:(NSString *)media {
  UIActivityViewController *socialViewController;

  // Create image from image data
  NSData *imageData = [[NSData alloc] initWithBase64EncodedString:
    media 
    options: 0
  ];
  
  UIImage *image = [[UIImage alloc] initWithData:imageData];

  socialViewController = [[UIActivityViewController alloc] 
    initWithActivityItems:@[text, image] 
    applicationActivities:nil
  ];

  UIViewController *rootViewController =  UnityGetGLViewController();

  [rootViewController presentViewController: 
    socialViewController 
    animated:YES 
    completion:nil
  ];
}

extern "C" { 
  void _Share(char* text, char* encodedMedia) {
    NSString *status = [NSString stringWithUTF8String: text];
    NSString *media = [NSString stringWithUTF8String: encodedMedia];
    
    [[FrispSocial instance] share:status media:media];
  }
}

@end
{% endhighlight %}

I use the singleton pattern instead of a static method for thread safety reasons. Initializing the singleton instance on class load ensures thread safety (<a target="_blank" href="http://csharpindepth.com/Articles/General/Singleton.aspx">more information about thread safe singleton implementations</a>).

The interesting part here is the extern "C" block. The extern "C" block is what enables access to call the Objective-C methods in C#. The methods wrapped in the extern "C" block use C <a target="_blank" href="http://en.wikipedia.org/wiki/Linkage_%28software%29">linkage</a> rather than C++ linkage. The reason for using C linkage is that it doesn’t mangle the method names. C++ linkage mangles the method names as it supports overloading which means the method name can’t be the unique identifier. Unity statically links the classes into the game making the _Share method available to be called in a wrapper class.

In my next post I will go through the wrapper class I wrote in Unity to use the Objective-C class defined here. 