# Implementing Business Solutions using the Power of Enterprise Open Source

![](_site/images/volunteerboatersstockphoto.png)

Red Hat, as the leading provider of an integrated, hybrid-cloud, enterprise software stack, needed to showcase its portfolio to its Public Sector customers.  To do so, it created a demonstration of a business solution that can serve as an enabler of humanity at its best:  voluntary collaboration between neighbors of a community during a disaster.  Specifically, this business solution, called [the Emergency Response demonstration](https://erdemo.io), is themed after the [Cajun Navy response to Hurricane Harvey](https://en.wikipedia.org/wiki/Cajun_Navy) in 2017.  The Cajun Navy first responders of Hurricane Harvey embodied the Power of Communities.

This blog post is part of a larger series of [webinars](https://www.brighttalk.com/webcast/16623/398059) and [articles]() that discuss the Power of Communities and how it can be impactful to your lines of business.

One challenge that you might still have is identifying those business use cases in your own organization that could also benefit from the power of enterprise open-source innovation in a manner similar to what you have observed with Red Hat's Emergency Response demo application.  In this article, I want to continue to explore the *Solution Spectrum* and a sampling of business use-cases with the hope of highlighting patterns you can use.

# Solution Spectrum
From a birds-eye perspective, business processes often fall into one or more categories of the following solution spectrum:

![](site/images/../../images/solution_spectrum.png)

**Straight Through** business processes can often times be fully automated.  These business processes are well defined and the business owner can graphically model its flow along with any business exception handling.  A single department in your organization might have many business processes and its often the case that these business processes have dependencies and interact with each other.  Using that same graphical model that the business owner created, business process **instances** are spawn.  These business process instances are typically triggered from some other business application in the form of an event.  The business process may include one or more *wait states* where its best to place that business process instance in a sleep mode until some other external business application triggers another event to re-awaken it and continue to the next defined task.

Recall from the Emergency Response demo application that the life-cycle of each *incident* is fully managed by a corresponding business process instance.  These instances execute as per the graphical business process definition which was previously defined in close collaboration with the *incident commander*.

The volume of business process instances that your business may need to execute on for a specific business process will vary greatly from just a couple a day to extreme scenarios of tens of thousands per second.  In all cases, your lines of business will greatly benefit from the modeling of your business processes and executing those same business processes in a consistent manner.

Historically, the tools used to model and execute *straight through* business processes have been prohibitively expensive (from both a monetary price as well as IT requirements perspective) to consider for all but the most critical use cases in larger enterprise organizations.  This is no longer the case.  Industry standardization along with the power of open source innovation now makes the automation of *straight through* business processes feasible to virtually all organizations globally. 

**Human Intensive** business processes share many of the same characteristics of *straight through* business processes.  However, as the name suggests, *human intensive* business processes involve humans and often can not be fully automated.  The *wait states* seen in *straight through* business processes are periods of time where the *knowledge workers* of your organization are executing tasks and adding value to the business process.

Assignment of knowledge workers to a specific business process instance can occur in a variety of ways.  The assignment can be specific to a group of knowledge workers.  Once one of the knowledge workers in the group is available, they can *claim* the next task of a business process to work on.  If the knowledge worker is unable to complete the task for whatever reason, then the task can be placed back on the queue for a different knowledge worker in the group to work on.  In some scenarios, assignment of knowledge workers to a task of a business process can be more complex.  It might be the case that business **rules** (also defined in collaboration with the business owner) need to execute so as to determine the assignment.  Semi-related is that often times its not initially apparent if a solution for a line of business is more process or rules oriented.  More often than not, the business solution is a mixture of both.  The tool that your IT department selects to implement your solution should offer both business process as well as business rule capabilities.

As a best practice, a human task should be set with a deadline for completion.  Remedial action for tasks that have not completed as per that deadline can vary.  It may be as simple as placing the task back on the queue but often times notifications / reminders should be sent out and the task should be *escalated* to a more senior member of the organization.

Being that your knowledge workers will be imputing value on your business processes during these human tasks, the user interfaces associated with these tasks should be intuitive and powerful enough to complete the task given the knowledge worker's role.  These user interfaces (often created by developers using any one of the many mature open-source UI frameworks on the market today), interact with the APIs exposed by the business process engine that manages your business processes.  On the topic of roles, these security roles and the assignment of your knowledge workers to those roles should already exist in your Identity Management solution.  Your IT department will know all the details.  It is critical that the business process and rules tooling used to implement your business solution integrates seamlessly with your organization's Identity Management solution.

**Case Management** builds off of the concepts discussed with *straight through* and *human intensive* business processes.  Case Management business processes, however, are more unpredictable, data-driven and unstructured.  To work on a case, your line of business may need to include knowledge workers, customers and various sets of case participants to collaborate in the decision-making.  Often, an ad hoc inclusion of knowledge workers is required as the processes experience unknown contents and events.  Hence, a solution is required to model the patterns of work which are complex, unpredictable, unstructured, unknown, and those which require a higher degree of collaboration, complex decision-making, dynamism, and so on.

An adoptive case is often modeled collaboratively between business owner and developer by specifying *milestones*.  Milestones are logical checkpoints in the case that represent the completion of a deliverable.  Milestones are used to trace the progress of that case.

Decisions made on a case by one or more knowledge workers in your organization are going to be made in the context of the files and data associated with that case.  Subsequently, integration with a Content Management System (where all of the files of a case reside) is often a requirement of case management.


# Use Cases Implemented using Enterprise Open Source Innovation

**Hospital employee scheduling and patient on-boarding**

**Case Management of unemployment benefit enrollments**

**Vehicle routing optimization between for postage / logistics carriers** 

**Command-n-Control applications operating in low-bandwidth, intermittent network environments**

**Long term maintenance and transparency of tax rules**
