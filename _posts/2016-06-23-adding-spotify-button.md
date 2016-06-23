---
layout: post
title: Add Spotify Play Button to my Jekyll theme
comments: true
spotifyTrack: spotify:track:6WrGEAVGRD6R6dCCAwS7QC

---

I like to listen music while I'm writing stuff, so today I decided to embed a
[**Spotify Play Button**](https://developer.spotify.com/technologies/widgets/spotify-play-button/) in my blog.
This button will help me to remember, in the future, what I was listening during a post writing.  

Adding the Spotify button to my Jekyll theme has been very simple. That's how I did to introduce this little improvement.  

### Basics
As you can see from my Github profile, this blog stands on the shoulder of Jekyll.
Also, I chose an interesting theme named **Hydejack** that I forked from this [Github repository](https://github.com/qwtel/hydejack).

**Hydejack** comes with a set of useful Jekyll configuration parameters allowing to customize page layouts.

Looking at the [`_layouts\post.html`](https://github.com/qwtel/hydejack/blob/master/_layouts/post.html) file, I saw that the current layout of a post page can be affected by specific parameters. These must be written at the top of each Markdown file representing a post. For example, looking at the `.md` file of this post itself, you can see the following:

{% highlight html %}
{% raw %}
---
  layout: post
  title: Add Spotify Play Button to my Jekyll theme
  comments: true
---
{% endraw %}
{% endhighlight html %}

### Adding the `spotifyTrack` parameter

I've defined a new custom parameter, named `spotifyTrack`, that must contain a **Spotify URI** reference.  
For example, for this post, I am using this: `spotify:track:6WrGEAVGRD6R6dCCAwS7QC`

### Adding the button

Then, to display a **Spotify Play Button** in each blog post page, I've simply edited the `_layouts\post.html` file of my **Hydejack** fork by adding the following:

{% highlight html %}
{% raw %}
{% if post.spotifyTrack %}
<aside class="spotify">
  <div>
    <div>I was listening to:</div>
    <br/>
    <iframe width="300" height="80" frameborder="0" allowtransparency="true"
    src="https://embed.spotify.com/?uri={{ post.spotifyTrack }}"></iframe>
  </div>
</aside>
{% endif %}
{% endraw %}
{% endhighlight %}

The above snippet, when processed by Jekyll, checks if in the current Markdown file does exist a reference to a parameter named `spotifyTrack`. If so, it displays an `iframe` containing the **Spotify Play Button** referring to the specific **Spotify URI**.

### Outcome

Easy come easy go, the outcome of my tiny improvement is at the bottom of this post!
