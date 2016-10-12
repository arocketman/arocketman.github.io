---
layout: post
title:  "Easily add icons to your jekyll's posts"
date:   2016-08-29 14:52:07 +0200
categories: main
icons: 
- fa-code
---
Adding icons to jekyll posts is really easy. We will use font-awesome which you can download and use from [here][font-awesome], open up your post and add the following to your YAML:
{% highlight YAML %}
icons:
- iconname1
- iconname2
{% endhighlight %}

You can find out the icon's name on the fontawesome's website linked before.

Now, you can go in your post-loop and access them easily as follows: 

{% highlight ruby %}
  <ul class="icons">
  	{% raw %}
	    {% for icon in post.icons %}
	    	<li><i class="fa {{icon}} fa-3x"></i></li>
	    {% endfor %}
	{% endraw %}
  </ul>	
{% endhighlight %}	

First, we create an unordered list, then we loop through the icons with a simple for loop. 
After that we use the icon we specified in the YAML as a class for a <i> element. 

That's it, you're done! You can see the result in the projects section of this website. If you want to achieve the same style as my blog here's the CSS:

{% highlight css %}
.icons{
	list-style:none;
	padding: 5px;
	text-align: right;
}

.icons li{
	display:inline;
	padding:5px;
}
{% endhighlight %}

[font-awesome]: http://fontawesome.io/