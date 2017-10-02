---
layout: post
title:  "Jenkins on OpenShift Tips"
date:   2017-09-29 16:01:44 +0200
categories: jekyll update
---
# Tips for working with Jenkins on OpenShift

This document describes a few strange problems I encountered while developing performance tests
for Jaeger Instrumentation and how I resolved them.

## Resolving Mysterious Jenkins Issues

While working on Jaeger Instrumentation performance tests, I ran into 2 different types of failures:
+ The job fails with a message in its log saying it was aborted, even though I had not aborted it
+ The job fails with a message in its log saying Jenkins had restarted, even though I had not restarted Jenkins

In both cases the root of the problem appears to be that Jenkins runs out of memory.  When creating a Jenkins instance on OpenShift, the default memory allocated is only 512mb.  Depending on your workload I'd recommend increasing this to at least 1024mb or higher.  Currently for Jenkins performance tests we are using 2048mb.

## Connecting to a running Jenkins instance
Sometimes it's useful to be able to connect to the pod running Jenkins and work directly on the file system.  

{% highlight shell %}
# First make sure you are logged in to the correct cluster
oc login -u <username> https:<cluster ip address>:<cluster port>
# Find the name of the Jenkins pod
oc get pods | grep jenkins
# This command should return something like:
jenkins-2-s2c32                                     1/1       Running     0          5d
# Now login
oc rsh jenkins-2-s2c32
# You will be logged into the root of the file system as user root.
# To get to the Jenkins home directory:
cd /var/lib/jenkins
As always, be careful not to delete anything important.
{% endhighlight %}

## Jenkinsfile Headaches

Currently Jenkins (and the Groovy compiler it uses) is very sensitive about syntax errors and tends to not give very specific failure information or even line numbers when your code contains something it doesn't like.  If, like me, you don't know groovy very well, then I'd recommend committing code to github in small chunks to make it easier to find the source failures.  As an alternative when developing code create a duplicate job where you can code in the script window until you get things working, and then commit your changes to source control.


