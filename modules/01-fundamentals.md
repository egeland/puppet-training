# Module 01: Getting Started

Disaster! The operations team is still drunk from last Friday and the task falls
to you - and you alone - to save Clickbait Enterprises by taking on the mighty
powers of DevOps and provisioning infrastructure in their absence.

But before we can save the day, we need to get some basics sorted first!


## Task 01: Connecting to your servers

The following information should be provided by your trainer:
* IP/name of the Puppet master server.
* IP/name of the Puppet ubuntu client server.

You should be able to connect with:

    ssh ubuntu@HOSTNAME

If your SSH key is not the usual `.ssh/id-rsa`, you may need to select your key
with `ssh -i KEYNAME ubuntu@HOSTNAME`.

All exercises will be done as the `ubuntu` user unless otherwise specified. We
also use `master` and `client` to reflect the two servers - it is recommended to
set these entries up in your `/etc/hosts` file for the duration of the training.


## Task 02: Running Puppet client

The Ubuntu node has been pre-configured to talk with your Puppet master. You
should be able to run Puppet after login and see it display a message confirming
that things are working:

    ubuntu@client:~$ sudo puppet agent --test
    Info: Using configured environment 'training'
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for client
    Info: Applying configuration version '1476232246'
    Notice: Puppet is configured and using the training environment
    Notice: /Stage[main]/Main/Node[default]/Notify[Puppet is configured and
    using the training environment]/message: defined 'message' as 'Puppet is
    configured and using the training environment'
    Notice: Applied catalog in 0.02 seconds
    ubuntu@client:~$

This is the command to use any time you want Puppet to apply a change you have
made to an actual server. Note the use of `sudo`, if you try to run the command
without sudo, it will try to work, but will fail.

You can also run it in a `dry-run` or `noop` mode. In this mode, it will not
make any actual changes to the system, but rather will show what would have
otherwise taken place. This is a very handy argument to use after making changes
in order to validate they will behave as expected.

    $ sudo puppet agent --test --noop
    Info: Using configured environment 'training'
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Applying configuration version '1476232440'
    Notice: /Stage[main]/Main/Node[default]/Notify[Puppet is configured and
    using the training environment]/message: current_value absent, should be
    Puppet is configured and using the training environment (noop)
    Notice: Node[default]: Would have triggered 'refresh' from 1 events
    Notice: Class[Main]: Would have triggered 'refresh' from 1 events
    Notice: Stage[main]: Would have triggered 'refresh' from 1 events
    Notice: Applied catalog in 0.02 seconds

Normally in production, Puppet will be running in the background and applying
the latest changes regularly (usually every 30 mins). For easier understanding
with training, we have disabled this background agent.


## Task 03: Change that message

We are now going to make a change on the Puppet master and validate that it gets
reflected on the client. In order to do this, we are going to connect to the
master:

  ssh ubuntu@master

In a real world environment, we would store all the Puppet modules in a git
service such as Github or Bitbucket. For training purposes, we have setup
local repositories on the server. You can view these repos at:

    ubuntu@master:~$ cd /home/myrepos/
    ubuntu@master:/home/myrepos$ ls -1
    environments
    README

You can see that we have an `environments` directory and a README file.

Inside the environments directory we have a `manifests/site.pp` file. Edit this
file, and change the message we are returning to the default node definition

    ubuntu@master:/home/myrepos/environments$ cat manifests/site.pp
    # Default Node configuration
    node default {
      notify { 'Hello World!': }
    }

Now run Puppet on the client server again. You will notice that nothing has
changed. That's because we have not committed the repo yet.

Let's commit the change back on the master:

     ubuntu@master:/home/myrepos/environments$ git commit -am "I can haz commit?"
     [2016-10-12 00:44:34 - DEBUG] Fetching '/home/myrepos/environments' to determine current branches.
     [2016-10-12 00:44:34 - INFO] Deploying environment /etc/puppetlabs/code/environments/master
     [2016-10-12 00:44:34 - INFO] Environment master is now at 152f2096e7ce8c46f87557af7de4c684db38c97b
     [2016-10-12 00:44:34 - INFO] Deploying environment /etc/puppetlabs/code/environments/production
     [2016-10-12 00:44:34 - INFO] Environment production is now at 12408db94266e58796b7de2d40b44e8df89ada6d
     [2016-10-12 00:44:34 - INFO] Deploying environment /etc/puppetlabs/code/environments/training
     [2016-10-12 00:44:34 - INFO] Environment training is now at 4eacb6016a924f3a5e9709d881c28f8b3cfa1816
     [training 4eacb60] I can haz commit?
     1 file changed, 1 insertion(+), 1 deletion(-)

Wow, what was all that noise? That's been generated by r10k, which we'll talk
about in the next module. For now, let's just check out the results of our
handiwork.

    ubuntu@client:~$ sudo puppet agent --test
    Info: Using configured environment 'training'
    Info: Retrieving pluginfacts
    Info: Retrieving plugin
    Info: Caching catalog for client
    Info: Applying configuration version '1476233259'
    Notice: Hello World!
    Notice: /Stage[main]/Main/Node[default]/Notify[Hello World!]/message: defined 'message' as 'Hello World!'
    Notice: Applied catalog in 0.02 seconds

If this worked - congrats! You've made your first change to a Puppet manifest
and validated end-to-end functionality of your environment.