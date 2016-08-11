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

## WAR Hotel Part 2 -- Run Test Kitchen in a Skeletal Cookbook

### Install the ChefDK

You need the ChefDK to be a Chef, so [download](https://downloads.chef.io/chef-dk/) and install it and make sure you can run __`chef -v`__ on your command line and the output you get looks similar to this:

{% highlight shell %}
chef -v
Chef Development Kit Version: 0.16.28
chef-client version: 12.12.15
delivery version: master (921828facad8a8bbbd767368bfc72f19bd30e7bd)
berks version: 4.3.5
kitchen version: 1.10.2
{% endhighlight %}

You should also be able to run __`kitchen -v`__ and see the same version for kitchen.

### Create Your Chef Cookbook

Now that we have the ChefDK installed, we're ready to create and develop our WAR Hotel cookbook.  The ChefDK provides an easy command for creating cookbooks. Go to a directory you want your cookbooks.

If you don't have a cookbooks directory, you can create it in your user folder and go into it like this:
{% highlight shell %}
mkdir cookbooks
cd cookbooks
{% endhighlight %}

Now generate the skeletal cookbook (watch out, this command is long and you need to scroll to see the whole thing)
{% highlight shell %}
chef generate cookbook war_hotel -C "Put Your Name Here" -m "put your email here" -I apachev2
{% endhighlight %}

### Install Docker and Configure Kitchen to Use Docker

Now that you can run Test Kitchen and have a cookbook, you're almost ready to run a test. First, it is so worth while to install and use Docker over Vagrant.  Main reason: Docker is just plain faster. So [download Docker](https://www.docker.com/products/docker) and install it. After you install it you should be able to run `docker -v` at your command line and see the version:

{% highlight shell %}
docker -v
Docker version 1.12.0, build 8eab29e
{% endhighlight %}

### Configure Test Kitchen

Now that you have the CHefDK, have created your cookbook, have installed Docker, this is the last step before you get to run a Test Kitchen test. Now that you have Docker installed, we need to configure kitchen to use Docker.  In your favorite code editor ([I like Atom](http://atom.io), install and use it if you don't have an editor) open the .kitchen.yml file that is in your cookbook's main directory (in this lesson it's the ~/cookbooks/war_hotel directory you created earlier).

Make the .kitchen.yml file look like this:
{% highlight yaml %}
---
driver:
  name: docker

provisioner:
  name: chef_zero

verifier:
  name: inspec

platforms:
  - name: centos

suites:
  - name: default
    run_list:
      - recipe[war_hotel::default]
{% endhighlight %}

We're just changing from Vagrant to Docker and we're setting up to use Inspec. More on Inspec later. It'll be our main testing tool.

### Run an Empty Test

OK, that was a lot of installing and setup and not a lot of coding, but that's how you have to get started.  But we're done setting up and we're ready to run a test.  We haven't created any code yet to test, but at least we can verify if we set up our software correctly.

Run this from your command line in the main cookbook directory: __`kitchen test`__. You should see a bunch of stuff go by and ultimately see a line that says `Summary: 0 successful, 0 failures, 0 skipped`. Like this:

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
Summary: 0 successful, 0 failures, 0 skipped

... a bunch of output ommitted here for clarity

Finished testing <default-centos> (0m17.57s).
-----> Kitchen is finished. (0m19.05s)       
```

If your output looks similar and you don't see any error messages, you're done with part 2.

Move on to [part 3](/war-hotel/war-hotel-part3).
