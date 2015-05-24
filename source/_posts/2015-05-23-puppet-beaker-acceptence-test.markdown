---
layout: post
title: "Puppet Beaker Acceptence Test"
date: 2015-05-23 02:33:08 +0530
comments: true
categories: beaker puppet testing automation 
---
## Puppet Beaker Installation 

**Wiki**: [Beaker Wiki](https://github.com/puppetlabs/beaker/wiki/Beaker-Installation)   
**Note**: 
``` 
Required 
 - Ruby 1.9+
 - libxml2, libxslt (needed for the Nokogiri gem)
 - g++ (needed for the unf_ext gem)
 - curl (needed for some DSL functions to be able to execute successfully)
```

* Install prerequisite
```
apt-get install ruby-dev libxml2-dev libxslt1-dev g++ zlib1g-dev curl make
```

* Install beaker
```
gem install beaker --no-rdoc --no-ri
```
<!-- more -->
* Incase beaker gem failed, because of nokogiri gem errorm run this command
```
gem install nokogiri -q --no-rdoc --no-ri -v "1.6.5" -- --use-system-libraries --with-xml2-include=/usr/include/libxml2 && gem install beaker --no-rdoc --no-ri
```
[Note: At time of writing latest nokogiri is 1.6.6, so installed last version of stable gem]

* Check beaker is installed
```
beaker --help
```

## Beaker Testing
**Wiki**: [Beaker Wiki](https://github.com/puppetlabs/beaker)    
**Note**: 
```
gem install beaker-rspec
```
**Test**: Modules tested is on role module in puppet git.   

* Create a spec dir under a module
```
mkdir -p spec/acceptance/nodesets
```

* Create a helper spec file at spec/spec_helper_acceptance.rb. This file is going be called whenever we run a specific class test.
{% codeblock spec_helper_acceptance.rb lang:ruby %}
cat spec/spec_helper_acceptance.rb
require 'beaker-rspec'

# Install puppet on the host
hosts.each do |host|
  install_puppet
end

# We don't suport this platforms
UNSUPPORTED_PLATFORMS = ['Suse','windows','AIX','Solaris']

RSpec.configure do |c|

  # log formatter in document style
  c.formatter = :documentation

  # Configure all nodes in nodeset
  c.before :suite do
    hosts.each do |host|
      # Update the /etc/hosts file to get it autosign
      on host, 'echo "`facter ipaddress` `facter hostname`.example.com `facter hostname`" > /etc/hosts'
      on host, 'echo "8.8.8.8  puppet.example.com puppet" >> /etc/hosts'

      # Run puppet agent
      #on host, puppet_agent('-t'), { :acceptable_exit_codes => 0 } # should exit with zero.. but later
      on host, puppet_agent('-t'), { :accept_all_exit_codes => true }
    end
  end
end
{% endcodeblock %}

* Create a nodeset file, this file is where we define docker configuration. We are creating two files i.e `default.yml` and `centos6-64.yml`
{% codeblock default.yml lang:yaml %}
HOSTS:
  centos-7-64:
    roles:
     - agent
    platform: el-7-x86_64
    image: centos:7
    hypervisor: docker
CONFIG:
  type: foss
  log_level: verbose
  docker_preserve_image: true
{% endcodeblock %}
{% codeblock centos6-64.yml lang:yaml %}
HOSTS:
  centos-6-64:
    roles:
     - agent
    platform: el-6-x86_64
    image: centos:6.6
    hypervisor: docker
CONFIG:
  type: foss
  log_level: verbose
  docker_preserve_image: true
{% endcodeblock %}

* Create a test acceptance, which you would like to test when the puppet agent run is completed. In this case we rolling role::lamp, so f.ex we are cehcking apache port and status
{% codeblock spec_helper_acceptance.rb lang:ruby %}
 cat acceptance/service_check.rb
require 'spec_helper_acceptance'

describe 'apache class', :unless => UNSUPPORTED_PLATFORMS.include?(fact('osfamily')) do
  case fact('osfamily')
  when 'RedHat'
    package_name = 'httpd'
    service_name = 'httpd'
  when 'Debian'
    package_name = 'apache2'
    service_name = 'apache2'
  when 'FreeBSD'
    package_name = 'apache24'
    service_name = 'apache24'
  when 'Gentoo'
    package_name = 'www-servers/apache'
    service_name = 'apache2'
  end

  context 'chech apache in role-lamp' do
    describe package(package_name) do
      it { is_expected.to be_installed }
    end
    describe service(service_name) do
      it { is_expected.to be_enabled }
      it { is_expected.to be_running }
    end

    describe port(80) do
      it { should be_listening }
    end
    describe port(443) do
      it { should be_listening }
    end
  end
end
{% endcodeblock %}

* Now all the files are in place, we can run some test
Note: Run this command from top of the module dir.
```
rspec spec/acceptance/service_check_service.rb
```
  The above command will look for file default.yml at acceptance/nodeset/default.yml
```
BEAKER_set=centos6-64 rspec spec/acceptance/service_check.rb
```
  If you want to check other OS, other then in default.yml, we can add a file nodeset dir, like whatevername.yml

* If you want to not to delete the node after running some test or to troubleshoot. pass these variable
```
BEAKER_set=centos6-64 BEAKER_destroy=no rspec spec/acceptance/service_check.rb
```
 - If running other then default.yml
```
BEAKER_destroy=no rspec spec/acceptance/service_check.rb
```
 - The dafult one.

* So your module structure should be like this, Just ignore the junit, logs dir, it will get created after you run the test.
```
.
├── manifests
├── templates
├── files
│   ├── init.pp
└── spec
    ├── acceptance
    │   ├── nodesets
    │   │   ├── default.yml
    │   │   ├── centos6-05.yml
    │   ├── service_check.rb
    ├── spec_helper_acceptance.rb
```
