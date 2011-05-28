# Goldberg 1.0.0

© Copyright 2010–2011 [C42 Engineering][]. All Rights Reserved.
[![Build Status](http://goldberg.c42.in/projects/goldberg.png)](http://goldberg.c42.in/projects/goldberg)

Goldberg is an alternative to [CruiseControl.rb][] that adheres to the same principles of being simple to use and easy to contribute to. It currently meets all common use cases of a lightweight CI server, and we plan to add more over time. A large majority of projects should be able to switch from CC.rb to Goldberg with little or no effort.

Goldberg can be used to continuously integrate codebases built using any language, not just Ruby.

Visit [goldberg.c42.in][] to see a live Goldberg server.

## Setting up your own Goldberg server

### Prerequisites

Git and Ruby (we usually use MRI 1.9.2, but any flavour should work).
RVM if you want to be able to run projects on different rubies.

Your project should have a Gemfile for [Bundler][].

Goldberg is currently tested only on Linux/Mac OS X but should run on JRuby on Windows.

### Installation
     
       # If you're on Ubuntu, you might need:
       sudo apt-get install sqlite3 libsqlite3-dev
     
       git clone git://github.com/c42/goldberg.git
       bundle install
       rake db:migrate

### Setting up a new repository
     
       ./bin/goldberg add <url> <name> [--branch <branch_name>]

By default it assumes the <code>master</code> branch. If you want to build on any other branch, use the -b --branch flag to specify it. The default command is <code>rake</code>, but you can also use "rake db:migrate && rake spec" if you have a rails project to build.

### Starting Goldberg
     
       # Start the CI server and web front-end at port 4242.
       bin/goldberg start [4242]

### Stopping Goldberg
     
       # Stop a running CI server
       bin/goldberg stop

### Tracking build status

Goldberg generates feeds that work with all CruiseControl-compatible monitors like [CCMenu (mac)][], [BuildNotify (linux)][] & CCTray (windows). The feed is located in the root and is named `cc.xml` (for finicky monitors we also have cctray.xml & XmlStatusReport.aspx). eg: [goldberg.c42.in/cc.xml][]

### Configuration

Goldberg will be checking out your code in \~/.goldberg. If you want to override this create an environment variable called GOLDBERG\_PATH.

You can configure the poller by copying the `config/goldberg.yml.sample` to `config/goldberg.yml`.

#### Force build

By default, the poller is configured to poll every 10 seconds. Even if you click on “force build,” it actually just sets a flag in DB for poller to build the project irrespective of the updates. If you want to it to do a build immediately after clicking on “force build”, then change the frequency to 1 second.

PS: Changing the frequency of poller to 1 second will not cause git calls every one second, as the project controls the frequency at which it should be polled as explained below.

#### Project based configuration

Every project in goldberg can have its own custom configuration by means of adding (either on goldberg instance or by checking it in with the codebase) `goldberg_config.rb` at the root of your codebase. As of now only the following configurations can be overridden, but going further this configuration will be used to configure even more.
     
      #Goldberg configuration
      Project.configure do |config|
        config.frequency = 20
        config.ruby = '1.9.2' # Your server needs to have rvm installed for this setting to be considered
        config.environment_variables = {"FOO" => "bar"}
        config.after_build Proc.new { |build, project| `touch ~/Desktop/actually_built`}
        config.timeout = 10.minutes
        config.command = 'make' #to be used if you're using anything other than rake
      end

If you want the project to be checked for updates every 5 seconds, you will need to change the poller frequency to less than 5 seconds using `goldberg.yml` as mentioned above.

#### Callbacks & Email Notifications

Goldberg provides on_build_completion, on_build_failure, on_build_success & on_red_to_green callbacks, which can be used to extend Goldberg and add functionality that is not provided out of the box. All the callbacks have access to the build that was completed & an object of email notification, which can be used to configure the mails to be sent on these events. The on_build_completion callback has an extra parameter previous_build_status.

The callbacks are part of goldberg_config.rb

		#Goldberg callbacks
		Project.configure do |config|
		
			config.on_build_completion do |build,notification,previous_build_status| 
		
				# sending mail
				notification.from('from@example.com').to('to@example.com').with_subject("build for #{build.project.name} #{build.status}").send
				
			end
			
			config.on_build_success do |build,notification| 
				# code to deploy on staging
			end
			
			config.on_build_failure do |build,notification|
				# post to IRC channel & send mail
			end

			config.on_red_to_green do |build,notification|
				# post to IRC channel & deploy on staging
			end
		end
		
Assume you want to post a message on IRC channel & there is a gem that can be used to do so, you can simply require the gem at the start of the project_config.rb file & write the code to post message in any of the callbacks.

### Setting up production instance

We suggest that Goldberg should be used behind apache, ngin-x or any such web server with passenger, unicorn or mongrel. If you don't want such setup, you should use a process monitoring tool like God or Monit to manage the server processes & restart them if they happen to die.

A sample god script file <code>config/god-script.rb</code> is available with Goldberg. Details for setting up God can be found at [http://god.rubyforge.org/]

### Help
     
      # To get man page style help
      ./bin/goldberg help [command]

## Contributors

-   Srushtivratha Ambekallu ([srushti][])
-   Jasim A Basheer ([jasim][])
-   Saager Mhatre ([dexterous][])
-   Tejas Dinkar ([gja][])
-   Niranjan Parajape ([achamian][])
-   Todd Sedano ([professor][])
-   Drew Olson ([drewolson][])
-   Aakash Dharmadhikari ([aakashd][])

  [C42 Engineering]: http://c42.in
  [CruiseControl.rb]: https://github.com/thoughtworks/cruisecontrol.rb
  [goldberg.c42.in]: http://goldberg.c42.in
  [Bundler]: http://gembundler.com/
  [CCMenu (mac)]: http://ccmenu.sourceforge.net/
  [BuildNotify (linux)]: https://bitbucket.org/Anay/buildnotify/wiki/Home
  [goldberg.c42.in/XmlStatusReport.aspx]: http://goldberg.c42.in/XmlStatusReport.aspx
  [srushti]: http://github.com/srushti
  [jasim]: http://github.com/jasim
  [dexterous]: http://github.com/dexterous
  [gja]: http://github.com/gja
  [achamian]: http://github.com/achamian
  [professor]: http://github.com/professor
  [drewolson]: http://github.com/drewolson
  [aakashd]: http://github.com/aakashd
