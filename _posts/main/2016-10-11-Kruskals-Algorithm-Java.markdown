---
layout: post
title:  "Minimum-spanning-tree: Network(Node,Arc) class and Kruskal's algorithm in java."
date:   2016-10-11 22:02:07 +0200
categories: main
icons: 
- icon-java
---
I've been recently studying some operational research methods at my University. One interesting algorithm I've came across is Kruskal's algorithm to build a minimum spanning tree.
I've built an overly-simplistic java class that handles Nodes and Arcs, here are some of the highlights:

A network class that abstracts a network made of both arcs and nodes:

{% highlight JAVA %}

public class Network {

    private ArrayList<Arc> arcs = new ArrayList<>();
    private HashMap<Integer,Node> nodes = new HashMap<>();

    public Network(int numNodes) {
        for(int i = 1; i <= numNodes; i++)
            nodes.put(i,new Node(i));
    }

    public void addArc(int source, int destination,int cost) {
        arcs.add(new Arc(nodes.get(source),nodes.get(destination),cost));
    }

...

{% endhighlight %}

Small classes to define an Arc and Nodes: 

{% highlight JAVA %}
class Arc{
    Node origin;
    Node destination;
    int cost;

    Arc(Node origin, Node destination, int cost) {
        this.origin = origin;
        this.destination = destination;
        this.cost = cost;
    }
}

class Node {
    int id;
    Integer tag;

    Node(int id) {
        this.id = id;
        this.tag = null;
    }
}
{% endhighlight %}

Now suppose we have the following network:

![Network]({{ site.url }}/assets/images/kruskalnetwork.PNG){: .center-image}

We want to determine the minimum-spanning tree. Kruskal's algorithm is **greedy**, first we order the arcs from the cheapest one to the most expensive. Then it picks the first arcs with the smallest cost and keeps doing so until they form a loop or each node has been labeled. If the introduced arc forms a loop, it will be skipped and the next arc will be examined.

How do we label the nodes? There are four basic possibilities:

1. __Both nodes are untagged.__ We tag both of them with a new tag. Arc is added to the solution.
2. __One of the two nodes are tagged.__ We tag the untagged one with the other's tag. Arc is added to the solution.
3. __Both are tagged but with different tags.__ We are joining spanning trees. The arc is added to the solution and all the nodes from the joining trees will have the same tag.
4. __Both nodes are tagged with same tag.__ In this case we're about to form a loop, arc is not introduced and we skip to the next one.

Java implementation of Kruskal's algorithm
-------------

Using the simple Network class we introduced before we can apply the previous rules and generate a minimum spanning tree.

{% highlight JAVA%}

    public ArrayList<Arc> start(){
        ArrayList<Arc> solution = new ArrayList<>();
        network.getArcs().sort((o1, o2) -> o1.cost - o2.cost);
        for(int i = 0; i < network.getArcs().size(); i++){
            Node n1 = network.getArcs().get(i).origin;
            Node n2 = network.getArcs().get(i).destination;

            if(n1.tag == null && n2.tag == null){ //Case 1
                n1.tag = currentTag;
                n2.tag = currentTag;
                currentTag++;
                solution.add(network.getArcs().get(i));
            }
            else if(n1.tag == null){ //Case2
                n1.tag = n2.tag;
                solution.add(network.getArcs().get(i));
            }
            else if(n2.tag == null){ //Case 2
                n2.tag = n1.tag;
                solution.add(network.getArcs().get(i));
            }
            else if(!n1.tag.equals(n2.tag)){ //Case 3
                int tag = n1.tag;
                network.getNodes().values().stream().filter(n -> n.tag.equals(tag)).forEach(n -> n.tag = n2.tag);
                solution.add(network.getArcs().get(i));
            }
        }
        return solution;
    }

{% endhighlight %}

First we sort the arcs according to the costs, then we simply apply the rules I've wrote previously. 
Let's try and solve the network I've posted in the previous image. Here's the main:

{% highlight JAVA%}

    public static void main(String[] args) {
        Network network = new Network(7);
        network.addArc(1,2,1);
        network.addArc(3,4,2);
        network.addArc(4,5,8);
        network.addArc(3,6,6);
        network.addArc(4,7,6);
        network.addArc(1,4,7);
        network.addArc(5,6,4);
        network.addArc(4,6,5);
        network.addArc(5,7,3);
        network.addArc(6,7,3);
        network.addArc(2,3,8);
        network.addArc(1,3,8);
        network.addArc(1,5,8);
        network.addArc(2,4,10);
        Kruskal kruskal = new Kruskal(network);
        ArrayList<Arc> solutions = kruskal.start();
        kruskal.printSolution(solutions);
    }

{% endhighlight %}

Executing the program will give us the following result:


{% highlight JAVA%}

1-2
3-4
5-7
6-7
4-6
1-4

{% endhighlight %}

Drawing the nodes with the arcs given by the output will give us the minimum spanning tree:

![NetworkSpanningTree]({{ site.url }}/assets/images/kruskalspanning.PNG){: .center-image}

You can find the source code at: [gist][gist-url]

[gist-url]: https://gist.github.com/arocketman/477d2888a91eba08291c13baf5fa1a38