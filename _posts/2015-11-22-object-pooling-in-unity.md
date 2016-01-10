---
title:  Object pooling in Unity
date:   2015-11-22 14:16:00
description: Information on object pooling in Unity including the Unity asset I wrote for abstracting object pooling.
keywords: [object pooling, unity]
author: michael
---

During the development of a game I was building for work I started to notice that some frames would randomly jump for a short span of time. After observing the game in scene mode I soon noticed that it was when objects were being instantiated and destroyed that the jumps would occur. After doing some research I came across the concept of object pooling which is where you keep a set of similar game objects in a pool that you recycle instead of instantiating and destroying them.

Most of the examples I saw of object pooling had a specific solution for a particular problem so I decided to create a generic solution for a common problem.

In most 2D projects I have worked on it's pretty common to have a script for instantiating objects and a script for cleaning up objects after they are no longer on the screen. Using the <a target="_blank" href="https://github.com/frispgames/frisp-object-pooling-asset">asset</a> I built, the script for instantiating game objects now looks like this:

{% highlight csharp %}
using UnityEngine;
using System.Collections;

using GameObjectPool = FrispGames.ObjectPooling.GameObjectPool;

public class DropObject : MonoBehaviour {

  private GameObjectPool _objectPool;

  public GameObject[] ObjectsToPool;
  public int PoolSize;
  public string PoolTag;

  void Start () {
    this._objectPool = GameObjectPool.CreatePool(objectsToPool, PoolSize, PoolTag);
  }
  
  void Update () {
    if (Input.GetKeyUp(KeyCode.Space)) {
      Drop();
    }
  }

  private void Drop () {
    GameObject obj = _objectPool.GetObject ();
    obj.transform.position = transform.position;
  }
}
{% endhighlight %}

To create an object pool we need an array of objects that we want to have in the same pool, this could be one element or many elements. The pool size is for having a count of objects we should store in memory when we load the game initially. The pool tag is for indexing the pool for later on if we need to reference it outside of this class.

In the `Drop` method we are asking the pool for an object and changing the position as the pooled object has a `Vector.zero` position. When you call `GetObject` it will retrieve an object from the pool if one exists and set it to be active in the scene. If one doesn't exist it will instantiate one. This means that the object pool will auto scale if the pool is all dried up.

The script for cleaning up objects now looks like this:

{% highlight csharp %}
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

using GameObjectPool = FrispGames.ObjectPooling.GameObjectPool;

public class ObjectDestroyer : MonoBehaviour {

  public string TagToDestroy;

  void OnTriggerEnter2D (Collider2D other) {
    if(TagToDestroy == other.tag) {
      if(GameObjectPool.HasPool(other.tag)) {
        GameObjectPool pool = GameObjectPool.GetPool(other.tag);
        pool.RecycleObject(other.gameObject);
      } else {
        Destroy (other.gameObject);
      }
    }
  }
}
{% endhighlight %}

Firstly we check whether we should be destroying these types of objects by checking the tag. Then we check if a pool for this object exists. If it does we will recycle the object so that it goes back into the pool for reuse. If there is no pool then we will just destroy the game object how we usually would.

Object pooling is a performance optimisation and isn't always necesary but for some older devices that games will be played on it can make all the difference. I've open sourced this asset on the <a target="_blank" href="https://github.com/frispgames/frisp-object-pooling-asset">frisp games github</a> feel free to leave any comments or suggestions this is just the first iteration of the asset.

