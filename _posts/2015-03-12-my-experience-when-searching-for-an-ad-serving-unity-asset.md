---
title:  My experience when searching for an ad serving Unity asset.
date:   2015-03-12 16:25:00
description: My experience when searching for a Unity asset for iPhone and Android builds that could serve ads from multiple ad networks.
keywords: [Unity, Unity Asset Store, iAd, Admob, Ultimate Mobile, Unity Asset]
author: michael
---

When building a game for iPhone and Android devices for Unity the question of how to monetize always comes up. There is always the concern of wanting to maintain a crisp level of game play while displaying non invasive adverts. A decision made in a team I'm apart of was to display banner ads during lapses in actual gameplay such as in between deaths. This technique was used by the popular game <a target="_blank" href="http://www.dotgears.com/apps/app_flappy.html">Flappy Bird</a>, which managed to <a target="_blank" href="http://www.theverge.com/2014/2/5/5383708/flappy-bird-revenue-50-k-per-day-dong-nguyen-interview">make $50,000 a day from ads</a>.

After some research I figured that I would need to have multiple Ad networks so I had a backup when one failed to load. I chose the following ad networks:

<a target="_blank" href="http://advertising.apple.com/au/">iAd</a>: This is apples ad network. iAd provides the best return on impressions but it often fails to serve ads especially in places like China and Russia.

<a target="_blank" href="https://www.google.com/admob/">Admob</a>: This is googles mobile ad network. Admob does not provide a high return on impressions but it roughly has a 99% success rate of serving ads.

I needed an asset that could provide me the ability to show <a target="_blank" href="http://advertising.apple.com/au/">iAds</a> if they could be served and if they couldn’t then serve Admob ads to maintain 100% fill rate and make the most money.

Because unity only has support for <a target="_blank" href="http://advertising.apple.com/au/">iAds</a> I decided to look into third party solutions. After looking through the Unity Asset store I came across the asset <a target="_blank" href="https://www.assetstore.unity3d.com/en/#!/content/20152">Ultimate Mobile</a> which was $70 USD.

After purchasing <a target="_blank" href="https://www.assetstore.unity3d.com/en/#!/content/20152">Ultimate Mobile</a> I imported the asset into the code base and was quite shocked at how much code came with it. There was well over 80 classes, I wasn’t happy adding all of this code to the game as I was only using about 10 of these classes. Having that many unused classes is quite risky as over time the code will become deprecated and I will need to manage it.

When purchasing assets from the <a target="_blank" href="https://www.assetstore.unity3d.com/en/">Asset store</a> it’s not always obvious what features the asset has and you don’t completely know until you purchase it, luckily Unity offers a two week grace period. It turned out that <a target="_blank" href="https://www.assetstore.unity3d.com/en/#!/content/20152">Ultimate Mobile</a> couldn’t manage multiple ad networks. I emailed the author asking if he could implement this feature but he basically told me to do it myself. I decided to ask for a refund because I was left with a large code base and no support from the author.

After consulting with the team we decided to build our own libraries to manage Ads. This consisted of writing native Objective-C and Java libraries that Unity can interact with through C#. I’ll explain in depth the process I went through to create these in my <a target="_blank" href="http://www.kiwiprogrammer.com/building-a-social-media-sharing-android-plugin-for-unity/">next blog post</a>. It was quite challenging but the intrinsic reward was amazing I felt like I had just hacked into a mainframe computer.

Things to keep in mind when purchasing unity assets:

**What are the risks of buying code?**

In our case <a target="_blank" href="https://www.assetstore.unity3d.com/en/#!/content/20152">Ultimate Mobile</a> was managing a core piece of functionality and if anything became deprecated we would need to cut through the forest of code to figure out how to update it. Given the size of the code base there would surely be side effects too. When you buy something that is doing everything under the sun you need to be careful. You may be getting a lot of features for one purchase but you may be getting a lot more than you bargain for in maintenance costs.

**How well maintained is the code by the author?**

Software progresses very quickly and libraries become deprecated all the time. If you don't have an author that is supporting the code it makes for a very expensive exercise later on down the track when you have to try and maintain the code yourself.

**Would you be better off writing the asset yourself?**

My suggestion would be to have a go building it yourself first. The benefit of this is that you can keep it small and to the point, whereas some assets include more code than you actually intend to use, which causes your code base to become unnecessarily large.
