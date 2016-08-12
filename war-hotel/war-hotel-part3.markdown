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

## WAR Hotel Part 3 -- Test Driven Development

### Don't Write Recipes First, Write Tests

It's hard not to dive write in and start building out your cookbook, to see the results you're being asked to develop.  But let me tell you one thing I've seen stand out in my years, if you test first, you feel safer, you are safer.  Safer for two main reasons:

1. Your tests prove to your customer that you understand what they want you to do. Your customer asks you to code something and you write the test first and they see what you are testing.  It becomes your agreement, your understanding together, that when that test passes, you finished what they asked.
2. Your tests keep you from breaking stuff on down the road.  As you write tests and implement code that makes those tests pass, you'll have a _lot_ of tests that should all pass. If you change code that unknowingly affects something else you are unaware of and that test you wrote a long time ago fails, you catch it, not your customer.

Testing first, having it fail, writing the code to make it pass is the way to go and how you'll learn from me. If you aren't yet convinced, [don't take my word for it](https://en.wikipedia.org/wiki/Test-driven_development#Benefits).

By the way, writing tests first is called Test Driven Development (TDD).

### Create the Test Directory for Test Kitchen

If you don't have an integration test directory in your cookbook, create one like this in your war_hotel cookbook directory:

{% highlight shell %}
mkdir -p ./test/integration/default
{% endhighlight %}

The directory should show up in your editor and you can create a new files to test your cookbooks.

### Write the Java tests

WARs need Java to run. So let's focus on Java first.  Write a test for Java by first creating a file called `test_install_java.rb` in your default test directory that you created in the previous step. We'll be using [Chef Inspec](https://www.chef.io/inspec/) to test our cookbook. You can read about it here in detail, but we'll gradually ramp up knowledge on Inspec as part of the WAR Hotel.

Inspec is pretty easy to get. It's a language that reads like what it's doing, and it checks things you think should be there in a lot of good ways.  Need to know if a user is on a machine? Easy.  Need to know if a particular computer program is running? Easy. As part of testing our cookbook, we need to know if our cookbook installed Java. Verify this with Inspec?  Easy.

Remember we're going to write a test without doing the work in the cookbook first, so this test is going to fail first; however, we'll quickly follow up with writing the cookbook code to make the test pass (TDD in action).

In the 'test_install_java.rb' file that you created, add the following test:

{% highlight ruby %}
describe command('java -version') do
  its('exit_status') { should eq 0 }
  its('stderr') { should match /1.8.0_101/ }
end
{% endhighlight %}

What is going on here? We are testing to see if the Java software we need on our machine is actually on our machine. We do this in two ways:

1. `its('exit_status') { should eq 0 }` tests to see that if the Java command can be run. If our cookbook configures our machine successfully, typing `java -version` should work fine and return 0. If it doesn't return 0, we know we don't have Java installed right.
2. `its('stderr') { should match /1.8.0_101/ }` tests to see that if the right version of Java is installed. The Java command returns it's version if we ask it too and that 'match' word up there just makes sure it's the version we want.

So you now have a very acceptable test you can run to make sure you are installing Java correctly. We'll run it, but we have a bit more testing to write.

### Write the Java Repo Test

Remember the experience thing I have going for me? We we learned something along the way of installing packages that reside on repository servers. You can't depend on them. Every server goes down and if you are depending on a repository server, yep, it's going to go down too. If you don't allow for this, you won't be able to keep your promises to your customers. You'll try to install the package, the repository server won't be there, and your code will fail.

My friend [Kevin](https://github.com/kmbulebu) came up with a neat idea: use a repository server that you maintain for all the major groups of things you need to install, set up a repository for that group.  This way you control the server, know when it goes down, and you can fix it.

What this means practically is that we need to set up a yum repository ourselves (we're doing this on CentOS) and configure our machine to pull Java from that yum repository.  Good thing this is a pretty easy thing to test and code.

Write a test for your Java repo by first creating a file called `test_install_java.rb` in your default test directory that you created in the previous step. Put these tests in it:

{% highlight ruby %}
describe file('/etc/yum.repos.d/java.repo') do
  it { should be_file }
  its('content') { should match /#{Regexp.escape('https://dl.bintray.com/flacito/rpm-java')}/ }
  its('content') { should match /#{Regexp.escape("enabled=0")}/ }
end
{% endhighlight %}

What is going on here? We are testing to see if the package repository is properly configured for us to install Java. We do this in 3 ways:

1. The yum repo config file must exists
2. The file must contain the URL to our repo server
3. The file should indicate that our repo is disabled.  This keeps it from being used in yum upgrades, we want to control that specifically.

So now we make sure we are configuring our Java package repo correctly. Let's try and run it.

### Run the Test Again

Run this from your command line in the main cookbook directory: __`kitchen test`__. You should see a bunch of stuff go by and ultimately see a line that says `Summary: 0 successful, 5 failures, 0 skipped`. Like this:

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

       ✖  2 failed
          fail:
          expected: 0
               got: 1

          (compared using ==)

          fail: expected "sudo: java: command not found\n" to match /1.8.0_101/
          Diff:
          @@ -1,2 +1,2 @@
          -/1.8.0_101/
          +sudo: java: command not found

       ✖  3 failed
          fail: expected `File /etc/yum.repos.d/pivotal-app-suite.repo.file?` to return true, got false
          fail: expected nil to match /http:\/\/192\.168\.1\.12\/rpm\/rhel7\/java\//
          fail: expected nil to match /enabled=0/

     Summary: 0 successful, 5 failures, 0 skipped

... a bunch of output ommitted here for clarity

Finished testing <default-centos> (0m17.57s).
-----> Kitchen is finished. (0m19.05s)       
```

Remember, we expected this to fail because we haven't written any cookbook logic to install Java (that's next). So if your output looks similar and you see two failing tests, you're done with part 3.

Move on to [part 4](/war-hotel/war-hotel-part4).
