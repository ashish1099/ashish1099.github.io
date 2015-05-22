---
layout: post
title: "My first puppet types and providers"
date: 2014-11-13 16:34:57 +0530
comments: true
categories: puppet cpan perl
---

This is first time I wrote the type and provider for a simple installation of the perl libraries
directly from cpan.


<!-- more -->
This is tree structure of the cpan module.
{% codeblock dns.pp lang:puppet %}
puppetmaster:/etc/puppet/master/modules/cpan # tree
.
├── lib
│   └── puppet
│       ├── provider
│       │   └── cpan
│       │       └── cpan.rb
│       └── type
│           └── cpan.rb
└── manifests
    └── init.pp

6 directories, 3 files
{% endcodeblock %}

{% codeblock dns.pp lang:ruby %}
# lib/puppet/provider/cpan/cpan.rb 
require 'puppet/provider/package'

Puppet::Type.type(:cpan).provide(:cpan, :parent => Puppet::Provider::Package ) do
	desc "Installation of perl modules from cpan"

	commands :cpanm => "/usr/bin/cpanm" if Puppet::FileSystem.exist? '/usr/bin/cpanm'
	commands :perldoc => "perldoc"

	# TODO add cpan as different way of installation.
	def create
		cpanm resource[:name]
	end

	def destroy
		cpanm('-U', '-f', resource[:name])
	end

	def exists?
		begin
			perldoc('-l', resource[:name])
			true
		rescue Puppet::ExecutionFailure => e
			false
		end
	end
end
{% endcodeblock %}


{% codeblock cpan.rb lang:ruby %}
# lib/puppet/type/cpan.rb
Puppet::Type.newtype(:cpan) do
	
	@doc = "Manage the installation and uninstallation of perl libraries from cpan repository using cpanminus manager,
		you can pass your own local cpan repository. 

		Valid options :- 
		1. 'ensure' => present||absent 		# Present is the default options, absent will uninstall the perl libraries.
		2. 'version' => latest 			# Latest version of perl libraries will get installed, if you want you can pass the version number.

		Example: 
			cpan { 'Text::ASCIITable' : ensure => present }
			cpan { 'Text::ASCIITable' : ensure => present, version => '1.0.1'} "

	ensurable do
		defaultvalues
		defaultto :present
	end

	newparam(:name, :namevar => true) do
		desc "Name of the Perl library"
	end
	
	newproperty(:version) do
		desc "Specify version number"
		validate do |value|
			fail("Invalid version #{value}") unless value =~ /^[0-9A-Za-z\.:-]+$/
		end
	end
end
{% endcodeblock %}
