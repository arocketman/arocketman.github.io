---
layout: post
title:  "Yelp challenge 2017 and social influence maximization."
date:   2017-07-22 14:52:07 +0200
categories: main
icons: 
- fa-code

---

For our big data course we were required to join the Yelp Challenge 2017, in particular we focused on social influence maximization. We would have to find among the yelp users, a set A of initial users that,given a product, would maximize the spread of influence of such product among the network.
Several approaches have been studied in literature, we refer to Kempe's paper mostly you can find it [here][kempe].
There are two models we have taken into consideration a randomic one (IC model) and the threshold one. Refer to Kempe for more information.

The implementation of a greedy algorithm on such a big network (1M+ nodes) wasn't as efficient as me and my colleague would have liked. That's why we chose a very different approach.

Taking in account the definition of "Community within a network": 

>"A network is said to have community structure if the nodes of the network can be easily grouped into sets of nodes such that each set of nodes is densely connected internally. Within complex networks, real networks tend to have community structure."

We can identify such communities with different algorithms, we picked the Label Propagation Algorithm for its computational advantages and spark compatibility.

The algorithm we created uses GraphX along with Pregel API. We want to measure how many nodes can be influenced starting from a single user.

As we previously talked about we use an impact coefficient that is calculated by taking into account the history of the single user.

Our algorithm is based on the IC model [1] works by Kempe et al. Here's an explanation of how the algorithm works:

In the superstep 0, every node receives a message containing the name of the node that starts the whole process. When the initiator node receives the message, it will be activated.

All the active nodes, will send messages to their inactive neighbours. The message payload contains the impact coefficient of the activator node. The coefficient itself is an 'activation probability'. The higher the coefficient, the more probable is the activation of the recipient's node.

All inactive nodes will receive a message containing such activation probability. A random number is thrown and the node will have a chance to be activated. If the activation is unsuccessful, there won't be another attempt of activation by the same node. If the activation is successful, the node will be starting to send messages to its inactive neighbours as per step 2.

The process ends when no further nodes can be activated. This can happen if the whole network was activated or all the inactive nodes were unsuccessfully activated and no further attempts can be made as per step 3.

If two or more nodes attempt to activate the same node, the merge function steps in. In this case the recipient node can be activated by any of the activators, the one who managed to activate the node is not important since we know which vertex the process was started from.

Here's an example:

![Network]({{ site.url }}/assets/images/examplegr.png){: .center-image} : .w100}

Given the following network, here's an execution output:

Vertex 1 , destination id: 2 , value of 0.7

Vertex 1 , destination id: 3 , value of 0.7

Vertex 1 , destination id: 5 , value of 0.7

Vertex 5 , influenced by : 1 , because I received 0.7 and got a value of 0.6831

Vertex 3 , influenced by : 1 , because I received 0.7 and got a value of 0.5820

Vertex 3 , destination id: 4 , value of 0.5

The script finishes since 3 didn't manage to influence two. In the end 1 influences three nodes (1,3,5). Multiple executions give more results that can be averaged:

1 -> influences 1,3,5 (3 nodes) 1 -> influences 1,2,3,4 (4 ndoes) 1 -> influences 1,2,3,4,5 (5 nodes)

1 influences an average of 5 nodes.

## Tech stack

Eventually our final stack is the following:

![Network]({{ site.url }}/assets/images/stackarchi.PNG){: .center-image}

Mongodb as NoSQL database.
Databricks as computational platform.
Spark as processional engine
Scala as programming language
GraphFrame + GraphX as graph abstraction libraries.

For more information refer to:

[https://arocketman.github.io/yelp/][link]

[1] : [kempe][kempe]

[kempe]: https://www.cs.cornell.edu/home/kleinber/kdd03-inf.pdf
[link]: https://arocketman.github.io/yelp/

