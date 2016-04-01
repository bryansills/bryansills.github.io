---
title: Test Post Please Ignore
layout: post
---

Hey guys! This is a test of my new blog, so I am gonna copy a blog post from Jake Wharton for testing a few things.

----

When [the Palette library](https://developer.android.com/tools/support-library/features.html#v7-palette) was teased the inevitable feature request came in to Picasso for a means of supporting it. This was certainly not an unreasonable request, and while we might not explicitly support it directly we will probably add something in the future to more easily facilitate its use.

There are two ways that Picasso can notify the caller of a successful image download: the `Callback` when loading directly into an `ImageView` or the more generalized `Target`. A `Target` is given direct access to the `Bitmap` object but the `Callback` is not (although you can get it indirectly with some cleverness). Regardless of `Bitmap` access is the problem that both of these are called on the main thread. Since Palette does a decent amount of computation we don't want to do it here anyways.

----

Saving the allocation for our transformation is easy. We can use the well-known pattern of object pooling. Thanks to recent support library updates, this is even easier to do than before with the [`Pools`](https://developer.android.com/reference/android/support/v4/util/Pools.html) helper.

```java
public final PaletteTransformation implements Transformation {
  private static final Pool<PaletteTransformation> POOL = new SynchronizedPool<>(5);

  public static PaletteTransformation getInstance() {
    PaletteTransformation instance = POOL.obtain();
    return instance != null ? instance : new PaletteTransformation();
  }

  private Palette palette;

  private PaletteTransformation() {}

  public Palette extractPaletteAndRelease() {
    Palette palette = this.palette;
    if (palette == null) {
      throw new IllegalStateException("Transformation was not run.");
    }
    this.palette = null;
    POOL.release(this);
    return palette;
  }

  // ...
}
```

`Downloader` and the upcoming `RequestHandler` are means to obtaining the original `Bitmap` instances (or `InputStream` instances) to fulfill a request. While we could invoke Palette here, I'm going to immediately reject it for a few reasons. There are multiple sources from which an image can be loaded which means we need to duplicate our logic across all of them. Additionally, the number of sources is constantly changing and some of them you cannot replace (I'm looking at you, drawable resource ID loading). Sometimes sources provide an `InputStream` which means we now have the burden of doing the initial `Bitmap` decoding ourselves--a job which is supposed to be Picasso's, right? Finally, the `Bitmap` at this level is the raw, original sized version. Not only will Palette operate much more slowly on it but a later transformation might alter the color makeup of the image.

<amp-img width="600" height="500" layout="responsive" src="http://loremflickr.com/600/500/puppy"></amp-img>


It looks like the `Bitmap` key based approach is the most viable at the expense of potential (but not proven) variance in the `Palette` instances for multiple `Bitmaps` of the same image URL.

----

This is a long post and there isn't exactly a nice neat bow to tie it all together with. I wrote it in this way because I wanted to showcase two very important things when it comes to how we do feature and API design:

 1. Not every use case deserves a first-party API but with some clever thinking you can usually accomplish what you are after by thinking outside of the box. We ultimately will support the use of Palette in Picasso but less as a first-class citizen and more through a general means of passing along arbitrary metadata through the pipeline.
 2. Exhausting multiple implementations and solutions to a problem is essential for ironing out the best approach. This was a journey to the final conclusion and it's one we frequently make for most of the changes we make to Picasso and all of our open source libraries. Not only can we be sure that we reached a good solution, but we also are able to defend and justify our decisions.

If you come upon any other bright ideas for applying Palette into Picasso I'd love to hear about them. Otherwise get playing with the fantastic Palette library today and keep an eye out for Picasso 2.4 in the next few days!