---
title:  "WAR Hotel"
layout: page
categories:
  - WAR Hotel
tags:
  - Chef
  - ruby
  - infrastructure
  - web server
---

## WAR Hotel Part 4 -- Installing Java

Remember that we're going to control our Java package repo. It's beyond the scope of this lesson, but under the covers Chef is going to use [Yum](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Configuring_Yum_and_Yum_Repositories.html), which you can read about if you like.

However, we're not going to just install Java from the default package repository that comes with CentOS Linux that is somewhere out on the Internet.  The test cases in Part 3 showed a bit of how we're going to do this.  We need to configure a specific Yum repository where our Java packages reside, doing so by creating a file in /etc/yum.repos.d.  

### Use the Chef yum Cookbook

Here's where the coolness of Chef comes in to play.  We could do this in a raw way and use a Chef [template](https://docs.chef.io/resource_template.html) resource, but the Chef community has written a [yum cookbook](https://supermarket.chef.io/cookbooks/yum) that does this for us and more.  We don't have the write the code! Let's use the community cookbook.

First thing we have to do is indicate that we intend to use the yum cookbook in our by placing code in our cookbook's `metadata.rb` file (in the main directory of your cookbook). Here's the entire contents of the file, but you'll just add that last line:

{% highlight ruby %}
name 'war_hotel'
maintainer 'Brian Webb'
maintainer_email 'btwebb@gmail.com'
license 'apachev2'
description 'Installs/Configures war_hotel'
long_description 'Installs/Configures war_hotel'
version '0.1.0'

depends 'yum'
{% endhighlight %}

### Write the Code to Configure the Java Package Repository

Next it's fairly easy to use this in our recipe.  We'll create a new file in our recipes directory called `install_java.rb` and make it look like the following:

{% highlight ruby %}
yum_repository 'java' do
  baseurl 'https://dl.bintray.com/flacito/rpm-java'
  description 'Java package repository'
  enabled false  # enabled false to avoid picking up in yum upgrades
  gpgcheck false # we're using the free version of Bintray, no GPG check :(
  action :create
end
{% endhighlight %}

What's this doing?  It's going to create the file we need, `/etc/yum.repos.d/java.repo`, and put the things in it that we're testing for like enabled=0 because we said `enabled false` in calling the yum cookbook.

### Write the Code to Install Java

Now we're ready to actually install the package.  Put this after your yum_repository resource in your `install_java.rb` recipe file:

{% highlight ruby %}
package 'jdk1.8.0_101' do
  options '--enablerepo=java'
  action :install
end
{% endhighlight %}

This installs Java and our tests should now pass. Let's try running them.

### Run the Test Again

Run this from your command line in the main cookbook directory: __`kitchen test`__. You should see a bunch of stuff go by and ultimately see a line that says `Summary: 5 successful, 0 failures, 0 skipped`. Like this:

```
kitchen test
-----> Starting Kitchen (v1.10.2)
-----> Cleaning up any prior instances of <default-centos>
-----> Destroying <default-centos>...
       Finished destroying <default-centos> (0m0.00s).
-----> Testing <default-centos>
-----> Creating <default-centos>...
       Sending build context to Docker daemon 227.8 kB
       Step 1 : FROM centos_chef_client
        ---> 637520284ec0
       Step 2 : ENV container docker

... a bunch of output ommitted here for clarity

-----> Verifying <default-centos>...
       Use `/Users/C62963/CloudStation/Center Stage/cookbooks/war_hotel/test/recipes/default` for testing

       ✔
       ✔

     Summary: 5 successful, 0 failures, 0 skipped

... a bunch of output ommitted here for clarity

Finished testing <default-centos> (0m17.57s).
-----> Kitchen is finished. (0m19.05s)       
```

Yay!  The tests pass. And you just made a big step towards a functioning WAR Hotel. You used Chef to install Java using your own custom package repository. Good work, but we've got a lot more to do.

Move on to [part 5](/war-hotel/war-hotel-part5).
