---
title:  Building a social media sharing Android plugin for Unity.
date:   2015-03-16 10:55:00
description: A tutorial on how to build an Android plugin that provides the functionality of native sharing on Android devices for Unity.
keywords: [Android, Java, Unity Asset, Unity, Sharing On Facebook, Sharing On Twitter, Game Development, Frisp Social]
---

When writing a game with friends using Swift we had a feature where players could share their score via a screenshot to social media. I needed to implement this in the Unity version of the game for both Android and iPhone. Unity doesn’t provide an API for sharing to social media on Android and the SDK’s require you to set up Facebook and Twitter applications. The more efficient way is to use native sharing. Native sharing doesn’t require the player to authorize Facebook or Twitter applications when wanting to share their screen. To do this I needed to implement an Android library that Unity could interact with.

First thing I did was create a directory for the Android library using the following command:

{% highlight bash %}
mkdir android-social-plugin
{% endhighlight %}

Then inside that directory I created a blank Android library by running the following command (you will need to <a target="_blank" href="https://developer.android.com/sdk/installing/index.html">install the android sdk</a> for the below to work):

{% highlight bash %}
android create lib-project \
  --name FrispSocial \
  --target android-14 \
  --path . \
  --package com.frispgames.frispsocial
{% endhighlight %}

Then I added the following to the build.xml that was generated:

{% highlight xml %}
  <target name="jar" depends="debug">
    <jar destfile="bin/frisp-social.jar" basedir="bin/classes" />
  </target>
{% endhighlight %}

The above allows me to build a jar by using the command ```ant jar```. The jar file is what we will add to the Unity project a little later.

Next I created the following folders to follow the package structure:
{% highlight bash %}
mkdir -p src/com/frispgames/frispsocial
{% endhighlight %}
The next step was to write the android code required to share an image and text to social media. I created the file: ```src/com/frispgames/frispsocial/FrispSocial.java```

{% highlight java %}
package com.frispgames.frispsocial;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.Bitmap.CompressFormat;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.provider.MediaStore;
import android.provider.MediaStore.Images;
import android.provider.MediaStore.Images.Media;
import android.util.Log;
import android.util.Base64;
import java.io.ByteArrayOutputStream;
import com.unity3d.player.UnityPlayer;

public class FrispSocial {

  public static void shareImage(String caption, String message, String media) {
    try {
      byte[] byteArray = Base64.decode(media, 0);

      Uri image = getImageUri(unityActivity(), byteArray);

      Intent shareIntent = new Intent();
      shareIntent.setAction(Intent.ACTION_SEND);
      shareIntent.setType("image/*");
      shareIntent.putExtra(Intent.EXTRA_TEXT, message);
      shareIntent.putExtra(Intent.EXTRA_STREAM, image);
      shareIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
      shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);

      unityActivity().startActivity(Intent.createChooser(
        shareIntent, caption
      ));

    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  private static Uri getImageUri(Context Context, byte[] byteArray) {
    try {
      Bitmap bitmap = BitmapFactory.decodeByteArray(
        byteArray, 0, byteArray.length
      );

      ByteArrayOutputStream bytes = new ByteArrayOutputStream();

      bitmap.compress(Bitmap.CompressFormat.JPEG, 100, bytes);

      String path = MediaStore.Images.Media.insertImage(
        Context.getContentResolver(), bitmap, "Screenshot", null
      );

      return Uri.parse(path);
    } catch (Exception ex) {
      Log.d("FrispSocial", ex.getMessage());
    }

    return Uri.parse("");
  }

  private static Activity unityActivity() {
    return UnityPlayer.currentActivity;
  }
}
{% endhighlight %}
The code above has a couple dependencies these need to be placed in the libs directory:

* classes.jar
* android-support-v4.jar

You can find these <a target="_blank" href="https://github.com/frispgames/android-social-library/tree/master/libs">here</a> as I have open sourced the android library.

The next step was to add the following line into the AndroidManifest.xml to make sure the Unity project that uses this library inherits the correct permissions:
{% highlight xml %}
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
{% endhighlight %}

The final step is to run ```ant jar``` to create the jar file. This will need to be placed under the ```Plugins/Android/``` directory in your unity project. Now you are able to interact with the methods inside the jar file using Unity. In my next blog post I will cover more about how I used this Android Library inside of a Unity project.