---
layout: post
title:  "How to make a smooth animation for a height toggle/untoggle view in Android"
date:   2017-01-27 14:52:07 +0200
categories: main
icons: 
- fa-code
- icon-java
- fa-android
---
In this brief article I'll demonstrate how to create a height animation such as this one:

<center>
<iframe src='https://gfycat.com/ifr/ReflectingParallelAquaticleech' frameborder='0' scrolling='no' width='348' height='451' allowfullscreen></iframe>
</center>

To do that, we'll use the Android ValueAnimator, you can find more about it on the [official documentation][ValueAnimator].

Here's the code snippet you'll need to animate:

{% highlight JAVA %}
    /**
     * Creates an animator that smoothly animates the passed view height from startHeight to
     * endHeight.
     * @param view The view that needs to be animated.
     * @param startHeight Starting height of the view.
     * @param endHeight Final height of the view.
     * @return
     */
    private ValueAnimator getToggleAnimation(final android.view.View view , int startHeight , int endHeight) {
        //We create the animator and setup the starting height and the final height. The animator
        //Will create smooth itnermediate values (based on duration) to go across these two values.
        ValueAnimator animator = ValueAnimator.ofInt(startHeight,endHeight);
        //Overriding updateListener so that we can tell the animator what to do at each update.
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //We get the value of the animatedValue, this will be between [startHeight,endHeight]
                int val = (Integer)animation.getAnimatedValue();
                //We retrieve the layout parameters and pick up the height of the View.
                FrameLayout.LayoutParams params = (FrameLayout.LayoutParams) view.getLayoutParams();
                params.height = val;
                //Once we have updated the height all we need to do is to call the set method.
                view.setLayoutParams(params);
            }
        });
        //A duration for the whole animation, this can easily become a function parameter if needed.
        animator.setDuration(Constants.TOGGLE_ANIMATION_DURATION);
        return animator;
    }
{% endhighlight %}

Please notice that the View doesn't need to be final. I am calling this method from an inner class in my project so that's why the keyword is there.

You can easily add the duration parameter so that it can be a little more flexible. 

In the following method we use the getToggleAnimation method with an imageView (profileImageView):

{% highlight JAVA %}
private void toggleDescription() {
    if(profileImageView.getLayoutParams().height == 0) {
    	//We use the oldHeight variable to apply the toggle animation and
    	//restore the untoggled status.
        getToggleAnimation(profileImageView,profileImageView.getHeight(),imageViewOldHeight)
        	.start();
    }
    else{
    	//First we get the old height and save it to a variable so that we can use it back on the if
        imageViewOldHeight = profileImageView.getHeight();
        //We apply the toggle "closing" animation with a destination height of 0.
        getToggleAnimation(profileImageView,imageViewOldHeight,0).start();
    }
}	
{% endhighlight %}

And that's it. To accomplish the final effect I simply called again the getToggleAnimation to all the elements I wanted animated.

[ValueAnimator]: https://developer.android.com/reference/android/animation/ValueAnimator.html
