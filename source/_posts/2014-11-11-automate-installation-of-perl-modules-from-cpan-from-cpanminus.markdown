---
layout: post
title: "Automate installation of perl modules using cpanminus manager"
date: 2014-11-11 21:31:46 +0530
comments: true
categories: Puppet perl cpan cpanminus
---

This manifest will help you to install perl libraries directly from cpan, You can set you cpan local repository or you can use any cpan mirror out there.
All I did is created a defination to install the perl libraries from cpan repository.
<!-- more -->

{% codeblock Terminal lang:puppet %}
# Install perl modules from cpan

class cpan {
	
	package { [ 'perl-YAML-Perl', 'perl-Error', 'perl-CPAN-DistnameInfo', 'perl-CPAN-Meta-Check', 
		'perl-File-pushd', 'perl-local-lib', 'perl-Module-CPANfile', 'perl-App-cpanminus', 
		'perl-aliased', 'make' ] : ensure => present }

	define perl ( $cpan_repo = "http://mirror.teklinks.com/CPAN/") {
		exec { "$name" : 
			command => "cpanm --mirror $cpan_repo --mirror-only $name", 
			require => Package["make", "perl-App-cpanminus" ], 
			unless => "perldoc -l $name" }
	} # End of cpan_perl
} #End of perl
{% endcodeblock %}

You can add like this in your example.pp

{% codeblock Terminal lang:puppet %}

cpan::perl { "Text::ASCIITable" :  }
{% endcodeblock %}
