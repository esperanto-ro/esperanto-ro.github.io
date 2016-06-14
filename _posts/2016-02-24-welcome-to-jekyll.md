---
title:  "Welcome to Jekyll!"
header:
  teaser: "https://farm5.staticflickr.com/4076/4940499208_b79b77fb0a_z.jpg"
categories:
  - some-category
tags:
  - update
---

You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets - here's some Scala code:

```scala
/**
 * Function that starts an asynchronous consumer run-loop or in case
 * a run-loop is already in progress this function increments the
 * itemsToPush count. And in case we've exceeded the bufferSize, then
 * we start to apply back-pressure.
 */
@tailrec private def pushToConsumer(state: State): Future[Ack] = {
  // no run-loop is active? then we need to start one
  if (state.itemsToPush == 0) {
    val update = state.copy(
      nextAckPromise = Promise[Ack](),
      appliesBackPressure = false,
      itemsToPush = 1)

    if (!stateRef.compareAndSet(state, update)) {
      // CAS failed, retry
      pushToConsumer(stateRef.get)
    }
    else {
      scheduler.execute(new Runnable { def run() = fastLoop(update, 0, 0) })
      Continue
    }
  }
  else {
    val appliesBackPressure = state.shouldBackPressure(bufferSize)
    val update = state.copy(
      itemsToPush = state.itemsToPush + 1,
      appliesBackPressure = appliesBackPressure)

    if (!stateRef.compareAndSet(state, update)) {
      // CAS failed, retry
      pushToConsumer(stateRef.get)
    }
    else if (appliesBackPressure) {
      // the buffer is full, so we need to apply back-pressure
      state.nextAckPromise.future
    }
    else {
      // everything normal, buffer is not full
      Continue
    }
  }
}
```

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
