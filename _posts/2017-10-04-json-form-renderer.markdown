---
layout: post
title:  "JSON Form Renderer"
date:  2017-1-13 20:15:35 -0700
categories: recipes components ui
---

There's a pretty good chance that your application will have more than one form. Depending on the size and nature of the project there could be something along the lines of a whole ton of them; and by a whole ton I mean that its probably in your interest to find a way to make them fast and easy.

If you don't have the luxury of a standardized form format(s) (hopefully no more than a handful) then I feel for you. We can be so much more useful when we aren't spending time building and debugging forms. A team that is perhaps a little short on hands could do worse than finding a quick way to lay down forms early on in a project.

There's no doubt that forms can be a beast. Sure, a simple form with no bells a whistles is fairly trivial but as soon as you start adding modern amenities it is a different ball game. Its a little like that game othello: very simple up front. When we expect things like client side validation (both sync/async), serverside validation, helpful icons, hopefully friendly error messages, fields that appear and disappear in certain orders, conditional validation, and who knows what else it becomes a different beast.

So we divide and conquer. Eat the elephant (one bite at a time). Don't build big apps, build lots of little small apps. Same idea. Chop the problem up. So lets only worry about laying the form out.

At the minimum, all we need to layout a form are the form name, the fields, and their order. Each field will have type, id, and label. Something along these lines:

{% highlight JSON %}
{
    "name": "loginForm",
    "fields": [
        {
            "id": "username",
            "type": "text",
            "label": "Username"
        },
        {
            "id": "password",
            "type": "password",
            "label": "Password"
        }
    ]
}
{% endhighlight %}

And dasss it! Faster than throwing down HTML and CSS and more reusable. Remember "Use the whole bull," or something like that. We could write a component that could spit this puppy out for us. Check it out:

{% highlight html %}
    <dom-module id="json-form-renderer">
        <template>
            <style>
                label {
                    display: block;
                    font-family: sans-serif;
                    font-weight: bold;
                    padding: 5px;
                }

                input {
                    display: block;
                    font-family: sans-serif;
                    padding: 5px;
                }
            </style>
            <form name="[[ jsonForm.name ]]">
            <template is="dom-repeat" id="repeater" items="[[ jsonForm.fields ]]" as="input">
                <template is="dom-if" if="[[ isType('text', input.type) ]]">
                    <label>
                        [[ input.label ]]
                        <input id$="[[ input.id ]]" type="text">
                    </label>
                </template>

                <template is="dom-if" if="[[ isType('password', input.type) ]]">
                    <label>
                        [[ input.label ]]
                        <input id$="[[ input.id ]]" type="password">                        
                    </label>
                </template>
            </template>
            </form>
        </template>
        <script>
            HTMLImports.whenReady(function() {
                Polymer({
                    is: 'json-form-renderer',
                    properties: {
                        jsonForm: Object
                    },
                    isType: function (reference, inputType) {
                        return inputType === reference;
                    }
                });
            });
        </script>
    </dom-module>
{% endhighlight %}

We just accept the JSON form definition as a property and switch on the type key of each field to decide which type of input we need. We can spin this into bootstrap forms no problem. A new form style just needs a new renderer.

{% highlight html %}
    <dom-module id="json-form-renderer">
        <template>
            <style>
                label {
                    display: block;
                    font-family: sans-serif;
                    font-weight: bold;
                    padding: 5px;
                }

                input {
                    display: block;
                    font-family: sans-serif;
                    padding: 5px;
                }
            </style>
            <form name="[[ jsonForm.name ]]">
            <template is="dom-repeat" id="repeater" items="[[ jsonForm.fields ]]" as="input">
                <template is="dom-if" if="[[ isType('text', input.type) ]]">
                    <div class="form-group">
                        <label for="[[ input.id ]]">[[ input.label ]]</label>
                        <input id$="[[ input.id ]]" type="[[ input.type ]]" class="form-control">
                    </div>
                </template>

                <template is="dom-if" if="[[ isType('password', input.type) ]]">
                    <div class="form-group">
                        <label for="[[ input.id ]]">[[ input.label ]]</label>
                        <input id$="[[ input.id ]]" type="[[ input.type ]]" class="form-control">
                    </div>
                </template>
            </template>
            </form>
        </template>
        <script>
            HTMLImports.whenReady(function() {
                Polymer({
                    is: 'json-form-renderer',
                    properties: {
                        jsonForm: Object
                    },
                    isType: function (reference, inputType) {
                        return inputType === reference;
                    }
                });
            });
        </script>
    </dom-module>
{% endhighlight %}

Now we can throw down forms quick and easy. These JSON form definitions could be AJAXed in and piped into the form render. Maybe something like this:

{% highlight html%}
    <ajax-request 
        method="GET" 
        url="forms/loginForm" 
        last-response="/{/{ jsonForm /}/}">
    </ajax-request>

    <json-form-renderer json-form="[[ jsonForm ]]"></json-form-renderer>
{% endhighlight %}

In Polymer \{\{ \}\} indicates two way binding. It's fine. It's not an antipattern. Don't get me started. [[ ]] is a one way binding down into the form renderer. As soon as the request finishes and changes the value of jsonForm the bindings will propagate the change downard to <json-form-renderer> who will begin rendering the form.


