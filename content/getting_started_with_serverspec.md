Title: Getting started with serverspec
Date: 2014-08-03 12:36
Category: blog
Tags: linux, sysadmin, devops, serverspec

*Serverspec* leys you can write *RSpec* tests for checking your servers are configured correctly.

What is *rspec*?
-------------------

RSpec is testing tool for the Ruby programming language.
Born under the banner of Behaviour-Driven Development




An example taken straight from rspec.info,

	require 'bowling'
	describe Bowling, "#score" do
		it "returns 0 for all gutter game" do
			bowling = Bowling.new
			20.times { bowling.hit(0) }
			bowling.score.should eq(0)
		end
	end

It might look a bit strange the first time.
A simple example in serverspec will clear this up.

	describe package('apache2') do
	    it { should be_installed }
	end

In this example we are interested in the apache2 package and we want to ensure that apach2 is installed.

Lets get started
--------------------

Install the serverspec gem

	$ sudo apt-get install -y ruby
	$ sudo gem install serverspec

create a place to work

	$ mkdir -p ~/code/serverspec-example
	$ cd ~/code/serverspec-example


Execute the serverspec-init script to get started.

	$ serverspec-init
	Select OS type:

	1) UN*X
	2) Windows

	Select number: 1

	Select a backend type:

	1) SSH
	2) Exec (local)

	Select number: 1

	Vagrant instance y/n: y
	+ spec/
	+ spec/localhost/
    + spec/localhost/httpd_spec.rb
	+ spec/spec_helper.rb
	+ Rakefile

We can now run the rake command to invoke server spec on your local machine

	$ rake spec
	......
	
	Finished in 0.99715 seconds
	6 examples, 0 failures

If we wanted to add some security checks into the mix we can just add a new spec.rb file

	$ $EDITOR spec/localhost/security_spec.rb
	require 'spec_helper'
	describe file('/etc/passwd') do
		# /etc/passwd should be a file
		it { should be_file }
		# it should be owned by root:root
		it { should be_owned_by "root" }
		it { should be_grouped_into "root" }
		# Only root may update the file, however everyone can read
		it { should be_mode 644 }
	end

Check our handy work.

	$ rake spec
	.............
	
	Finished in 0.99715 seconds
	10 examples, 0 failures
