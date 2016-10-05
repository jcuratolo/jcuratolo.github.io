---
layout: post
title:  "Thoughts And Recipes With Components"
date:   2016-10-04 23:35:49 -0700
categories: recipes components ui
---
Component based UIs offer a clean and powerful way to construct UIs quickly and reliably. This is the first in a series of posts that will share thoughts and related recipes for approaching component based UIs. The recipes should be applicable to any component driven framework though I'll be using Polymer for the purposes of demonstration.

One of the perks of building with components is we can work declaratively. Everyone will tell you that this means we code by writing what we want done instead of how we want it done. Sounds great, no? 

I have no clue what that is supposed to mean. Surely, if we could simply instruct our computers as to how we want things done we would surely be living in the future and certainly searching for new jobs. Indeed, it seems a sizeable portion of the population believes that many professionals that habitually use computers in the line of work in fact work almost entirely declaratively with the human operator providing gentle guiding nudges as our electronic beasts of burden toil away. After all, its how Captain Picard designed things in the holodeck' why wouldn't it be so?

Back to the matter at hand. Simply put, working declaratively means we automate the most regular (uniform) parts of our code. Quite often this takes the form of what might be termed **glue code**. In order to do this we must first specify how we want these regularly occurring things done (imperatively!). We then are then free to work declaratively by reusing these specifications automatically.

Consider the simple component defined below. In order to add event listeners we call `addEventLister` and `removeEventListener` (you do clean up after yourself don't you?) with the event name and handler when the component is attached and detached from the DOM. No big deal for a single component with a single event that we are interested in. But what happens at a larger scale with many components and many events? 

{% highlight html %}
<dom-module id="a-component">
  <template></template>
  <script>
    Polymer({
      // This will be the component's tag
      is: 'a-component',
      
      // Called when component is attached to DOM
      attached: function () {
        this.addEventListener('property.changed', this.onPropertyChanged);
      },

      onPropertyChanged: function (e) {}
      
      // Called when component is detached from DOM
      detached: function () {
        this.removeEventListener('property.changed', this.onPropertyChanged);
      }
    })
  </script>
</dom-module>
{% endhighlight %}

First we get tired of writing the same crap over and over again. Won't kill us. We've had worse. 

Second, we expose ourselves to more mistakes because we are writing more and more code. Marco Pierre White, who was at one time the youngest chef to be awarded three Michelin stars, said something along the lines of "Perfection is many small things well." He is, of course, sort of a jerk sometimes and also right. This is important to us because if we do one thing wrong, mistype a single character, our entire program could crap out and therefore we ***require*** perfection in the pursuit of the elusive JavaScripts. Too bad for the whole being human thing.

DRY isn't just about being lazy or even smart. Its also about reducing your code's exposure to your damn ***humanity***, the certainty with which you will make stupid mistakes and thus waste time and brain juice finding and fixing things that could have been avoided.

A third thing happens. Our code is a little more muddy. Not hard to read, mind you, but somewhat less clear since **glue code** is co-located with code that actually describes something useful and unique: Real-Deal Logic. One of my favorite things after beef and mountains.

Wouldn't it be nice if our component's methods were comprised mostly of actual logic that was unique? We would be able to scan through the code and begin understanding how things worked at a glance without having to wade through largely inert glue. 

Now to be fair, Polymer does provide an idiom for this. This touches on another point which is that its also nice to keep weird framework/library specific stuff out of nice clean code as much as possible. Makes it easier for you to pack up and leave for another framework in the future and can even facilitate things like testing if your framework has a particularly shit testing story.

So what can be done to stave off the angst, stupid mistakes, and confusion? We automate the regular parts, thereby saving ourselves *from* ourselves, from our fat fingers and work declaratively. What if we could set up listeners like this:

{% highlight html %}
<dom-module id="a-component">
  <template>
    <event-listener 
      event="property.changed" 
      handler="onPropertyChanged">
    </event-listener>
  </template>
  <script>
    Polymer({
      is: 'a-component',
      
      onPropertyChanged: function (e) {}
    })
  </script>
</dom-module>
{% endhighlight %}

We could make `<event-listener>` set up the listener and never forget to remove it without failure or error. Other programmers could come along and throw down some more listeners without knowing how it worked; only that it did and it was needed. Perhaps these other programmers on your team are less experienced programmers. Building something like this not only potentiates your code, but the code of others who can stand on your shoulders and help get the job done.

What might `event-listener` look like inside?

{% highlight html %}
<dom-module id="event-listener">
  <template></template>
  <script>
    Polymer({
      is: 'event-listener',
      
      properties: {
        event: String,
        handler: String
      },

      // domHost is the component whose template we are in.
      // In other words, we are keying the handler by name 
      // in the parent element
      attached: function () {
        this.decorate(this.domHost, 'attached', function () {
          this.domHost.addEventListener(this.event, this.domHost[this.handler]);
        }.bind(this));
      },

      // A simple decorator utility
      decorate: function (targetObject, methodName, decorateWith) {
        var methodToWrap = targetObject[methodName];
        targetObject[methodName] = function () {
          decorateWith();
          if (typeof methodToWrap === typeof Function()) {
            methodToWrap.apply(targetObject, arguments);
          }
        }
      },

      detached: function () {
        this.decorate(this.domHost, 'detached', function () {
          this.domHost.removeEventListener(this.event, this.domHost[this.handler]);
        }.bind(this))
      }
{% endhighlight %}

Notice that it has an empty template so it wont actually clutter things up in our app. What are some other things we often do with events? How about stop propagation? Why not add a boolean attribute to `event-listener` so we could set this up along with the listener?

{% highlight html %}
<dom-module id="event-listener">
  <template></template>
  <script>
    Polymer({
      is: 'event-listener',
      
      properties: {
        event: String,
        
        handler: String,

        // Define the property like this when we want to initialize with a value
        stopPropgation: {
          value: false,
          type: Boolean
        }
      },

      attached: function () {
        this.decorate(this.domHost, 'attached', function () {
          this.domHost.addEventListener(this.event, this.eventHandler);
        }.bind(this));
      },

      // A simple decorator utility
      decorate: function (targetObject, methodName, decorateWith) {
        var methodToWrap = targetObject[methodName];
        targetObject[methodName] = function () {
          decorateWith();
          if (typeof methodToWrap === typeof Function()) {
            methodToWrap.apply(targetObject, arguments);
          }
        }
      },

      // Wrap the handler specified on the host
      eventHandler: function (e) {
        if (this.stopPropagation) { e.stopPropagation(); }
        this.domHost[this.handler].call(this.domHost, e);
      },

      detached: function () {
        this.decorate(this.domHost, 'detached', function () {
          this.domHost.removeEventListener(this.event, this.eventHandler.bind(this));
        }.bind(this))
      }
{% endhighlight %}

and then to use our new `event-listener`:

{% highlight html %}
<dom-module id="some-component">
  <template>
    <event-listener 
      event="property.changed" 
      handler="onPropertyChanged" stop-propagation>
    </event-listener>
  </template>
  <script>
    Polymer({
      is: 'some-component',

      onPropertyChanged: function (e) {},
    })
  </script>
</dom-module>
{% endhighlight %}

Setting up your subscriptions like this is easy and keeps the glue out of your code. You won't forget to unsubscribe because you wrote the code to setup the listener the right way once and can take advantage of this every time you throw down a new listener. Instead of writing how you want set up a listener by adding it and removing it during certain component life-cycle events you simply state that you want an event called X handled by a method called Y.