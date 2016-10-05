---
layout: post
title:  "Thoughts And Recipes With Components"
date:   2016-10-04 23:35:49 -0700
categories: recipes components ui
---
Component based UIs offer a clean and powerful way to construct UIs quickly and reliably. This is the first in a series of posts that will share thoughts and related recipes for approaching component based UIs. The recipes should be applicable to any component driven framework though I'll be using Polymer for the purposes of demonstration.

One of the perks of building with components is we can work declaratively. Everyone will tell you that this means we code by writing what we want done instead of how we want it done. Sounds great, no? 

I have no clue what that is supposed to mean. Surely, if we could simply instruct our computers as to how we want things done we would be a) living in the future and b) searching for new jobs.

{% highlight html %}
<dom-module id="a-component">
  <template></template>
  <script>
    Polymer({
      is: 'a-component',

      attached: function () {
        this.addEventListener('property.changed', this.onPropertyChanged);
      },

      onPropertyChanged: function (e) {}

      detached: function () {
        this.removeEventListener('property.changed', this.onPropertyChanged);
      }
    })
  </script>
</dom-module>
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
