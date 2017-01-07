---
layout: post
title:  "Simulate a simple business process with BonitaBPM"
date:   2016-12-26 14:52:07 +0200
categories: main
icons: 
- fa-code
---
BonitaBPM is one of the many BPMS available, community edition is free and downloadable from [here][bonitasoft] , the software uses standard BPM notation (BPMN).
We will create a simple process where an employee proposes a new marketing strategy and awaits for the manager approval.

The elements we are going to need are here listed:
1. A pool representing the whole process
2. Two swimlanes, one for the employee and one for the manager.
3. A business object with a set of variables.
3. Start/end events
4. Activities

The first thing we are going to do is setup a simple business data model. You can do so by going on "Development->Business Data model -> Manage..".

Once we are set, create a business data model as illustrated in the following picture:

![Business object]({{ site.url }}/assets/images/marketing_proposal/1.PNG){: .center-image}

Now we can create our diagram, it will consist of one activities and two events (start, end) . A couple of actors will be involved and we're gonna separate their activities through swimlanes:


![The diagram]({{ site.url }}/assets/images/marketing_proposal/2.PNG){: .center-image}

First, let's attach our object to the pool. Select your pool and from the bottom menu select the "Data" tab. Add a businessProposal pool variable. We are going to edit the pool variable's default value by selecting edit, then click the pencil tool just next to the "Default value" text box just as shown in the picture:

![The diagram]({{ site.url }}/assets/images/marketing_proposal/3.PNG){: .center-image}

You'll find some code, we are just going to change the status to "pending" instead of taking it from the input form:

{% highlight groovy %}
def marketingProposalVar = new it.systemonesolution.model.marketingProposal()
marketingProposalVar.Title = proposalInput.Title
marketingProposalVar.Description = proposalInput.Description
marketingProposalVar.status = "pending"
return marketingProposalVar
{% endhighlight %}

Secondly, we will need to create a contract to the start event. Select the execution tab and create a new contract selecting "Add from data". Unselect the non-mandatory fields such as status.

We'll need to make sure that just the manager is able to fill out the review form. For example, in the standard ACME company provided by the software, we have that "Helen Kelly" is "Walter Bates" manager. So we're going to assign the employee lane to walter bates and the manger one to helen kelly.
To do this, select the swimlane and on the "General" tab select "Actors". Then configure it accordingly to your organization. I created a helen.kelly actor and attached it to the helen.kelly user.

Running the process in its current status will provide automatically generated forms such as this one:

![Form1]({{ site.url }}/assets/images/marketing_proposal/4.PNG){: .center-image}

Logging in as walter.bates and executing the proposal task and helen.kelly and doing the review task will complete the whole process execution.
We'll now use the UI designer to show some details of the proposal so that the manager can actually review it.

Select the activity "Review marketing strategy" and go to the "execution" tabs then click "form" and the small pencil near the textbox. The UI designer will pop open.

Let's add a new JSON API variable from the bottom menu. Call it "proposaldata" and this as its content data: 

"/bonita/{{context.proposal_ref.link}}"

Then you'll be able to easily access the variable fields using the text widget {{proposaldata.field}} . We'll add two widget displaying "Title" and "Description" so that the manager can actually review it. 

We're all set, you can run the process and simulate it easily now. Just hit the run process button and see for yourself how it works. Mind you have to log in as helen kelly to execute the review operation. Password is "bpm" for all of the users in the ACME organizations. The following pictures illustrate the process:

![Exec1]({{ site.url }}/assets/images/marketing_proposal/6.PNG){: .center-image}
![Exec2]({{ site.url }}/assets/images/marketing_proposal/7.PNG){: .center-image}

That's pretty much the foundations of what you can do to run a really basic process with bonita. Of course there are many other things that can be done but I wouldn't stress it in a small blog post.
Some ideas:

1. Add constraints for the status parameter.
2. Add a select instead of a textfield for the status parameter.
3. A control panel for the manager to overview all the requests.

[bonitasoft]: http://www.bonitasoft.com/