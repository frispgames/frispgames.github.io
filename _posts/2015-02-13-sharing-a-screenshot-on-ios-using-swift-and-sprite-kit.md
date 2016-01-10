---
title:  Sharing a screenshot on iOS using Swift and Sprite Kit.
date:   2015-02-13 10:18:00
description: A tutorial on writing code to share a screenshot in iOS when developing a game using Swift and Sprite Kit.
keywords: [Swift, Sprite Kit, Game Development, Sharing on Twitter, Sharing on Facebook]
author: michael
---

One way I have found that works well for marketing a game is to give players the ability to share a screenshot of their score on Twitter or Facebook. It works well because as soon as people start sharing their score, their friends get curious and want to play too. In this post I will run through how I implemented the ability to share a screenshot to Twitter or Facebook using Swift and Sprite Kit.

The following class takes care of sharing the player's score to Facebook or Twitter:

{% highlight swift %}
import Foundation
import UIKit
import SpriteKit

class Social {

  func shareScore(scene: SKScene) {
    let postText: String = "Check out my score! Can you beat it?"
    let postImage: UIImage = getScreenshot(scene)
    let activityItems = [postText, postImage]
    let activityController = UIActivityViewController(
      activityItems: activityItems,
      applicationActivities: nil
    )

    var controller: UIViewController = scene.view!.window!.rootViewController!

    controller.presentViewController(
      activityController,
      animated: true,
      completion: nil
    )
  }

  func getScreenshot(scene: SKScene) -> UIImage {
    let snapshotView = scene.view!.snapshotViewAfterScreenUpdates(true)
    let bounds = UIScreen.mainScreen().bounds

    UIGraphicsBeginImageContextWithOptions(bounds.size, false, 0)

    snapshotView.drawViewHierarchyInRect(bounds, afterScreenUpdates: true)

    var screenshotImage : UIImage = UIGraphicsGetImageFromCurrentImageContext()

    UIGraphicsEndImageContext()

    return screenshotImage;
  }
}
{% endhighlight %}

When shareScore() is called the following comes up on the iPhone:

![Sharing image using Sprite Kit and Swift](/assets/blog-images/2015-02-13/BouncyShare.png)

Then when you click to share on twitter you get the following screen:

![Sharing image on Twitter using Sprite Kit and Swift](/assets/blog-images/2015-02-13/BouncyTwitterShare.png)

You get a similar screen for sharing on Facebook. You can use this marketing technique to drive more people to your game, then monetize by displaying Ads or having in game purchases.
