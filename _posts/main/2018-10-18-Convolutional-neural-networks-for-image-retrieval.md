---
layout: post
title:  "Extracting features from Convolutional neural networks for image retrieval"
date:   2018-04-10 12:02:07 +0200
categories: main
icons: 
- database
- icon-python
---

Recently I've had the opportunity to further explore convolutional neural networks. More specifically I analyzed how layer activations of a convolutional neural network can be effectively used for different kind of purposes.
Recent works highlighted how it's possible to use layer activations from the fully connected layers to perform object detection, just by removing the last classifying layer and by transforming the activations in a feature vector that can be used for different purposes.

Recent scientific articles such as the one of Zhi et al[1] have succeeded in using features extracted from the last pooling layer and use the vector for content based image retrieval purposes.

Perhaps an even more interesting discovery is the fact that loosely trained CNNs still perform really well on CBIR tasks. Where by loosely trained I don't mean that they're trained on a small dataset, but I mean that the CBIR keeps on working well even on tasks that are unrelated to the ones for which the CNN was previously trained for.

![Vgg16]({{ site.url }}/assets/images/zoom-vgg16.png){: .center-image}

In the figure above the popular VGG-16 architecture is showed. Extracting features from a fully connected layer would result in a vector with a length of 4096. Which is.. big, but arguably not that big . 
However, if you'd extract from the max pooling layer, you'd have a 7x7x512 size feature vector, that totals to 25088. On the bright side, compressing it will result in a smaller size vector, a GZIP conversion could work just fine and reduce the size drastically.

So, how do you extract these features? Python libraries such as Caffè are well suited for this task, as a matter or fact caffè is the natural evolution of decaf which is one of the first frameworks created to specifically extract feature vectors from CNNs. 

I did it with deeplearning4j since I was learning the framework at that time. So here are the basic steps: 

1) Instantiate a ComputationGraph
2) Resize and normalize an image with the specifics of the given CNN (224x224 for VGG-16)
3) Feedforward an image
4) Get an INDArray from the desired output layer ('fc2' or 'pool5' in my case). 
5) Compare this INDArray with the other ones with a simple simularity measure such as L2 or cosine.

Arguably a L2 norm could improve retrieval results as highlighted by some similar articles. 

Results obtained were quite suprising. The system worked exceptionally well even on tasks unrelated to the ones the CNN was trained for. Here's just a glimpse of code to get you started:

{% highlight JAVA %}

    Map<String, INDArray> stringINDArrayMap = extract(file, vgg16transfer);
    INDArray fc2 = stringINDArrayMap.get(EXTRACTION_LAYER);
    INDArray normalized = fc2.div(fc2.norm2Number());
    saveCompressed(file,normalized);
    
{% endhighlight %}

Where the extract method basically contains the operations of step 2 and a vgg16.feedForward(image, false) for step 3. And saveCompressed is just a call to a method that compresses the features with a basic compressor provided by ND4j but you could arguably use the array as-is.

[1] http://wineyard.in/Abstract/mtech/DIP/2016/bp/16D38.pdf