---
layout: post
title: "Ruby subcene.com subtitle downloader"
date: 2014-10-15 09:14:28 +0530
comments: true
categories: ruby script
---

I have written a ruby script in which it will download the subtitle as you need. To demonstrate more in this, I would like to share  why I did this. Because I was tried of going onto subscene.com
and searching for a movie and then download the subtitle and then unzip it. which was very painful for me.

This let me write a small ruby wrapper to do all this thing automatically. This basically takes 4 arguments, like `--movie_name` `--year` `--resolution` and if you want any specific ripper subtitle then you can pass `--author`.
Just copy this script under `$PATH` and run `subscene` like this
<!-- more -->

{% codeblock Terminal lang:bash %}
# subscene 
Missing options: movie_name, year, resolution
*** Subcene subtitle downloader ***
Usage : subscene -m deliver-us-from-evil -y 2014 -r 720
    -m, --movie_name                 Name of the Movie
    -y, --year                       Year of the Movie
    -r, --resolution                 Resolution of the Movie
    -a, --author                     Author/Ripper of the Movie
    -h, --help                       Display this screen

{% endcodeblock %}

This show that it is looking for some arguments, so we will pass it then. As an example, I'm taking this movie `22 Jump Street`

**NOTE** :
Movie name should be in this format `"22-jump-street"` `"gotham-first-season"` `"lost-sixth-season"` or `"deliver-us-from-evil"`

{% codeblock Terminal lang:bash %}
# subscene -m 22-jump-street -y 2014 -r 720 -a YIFY`
Successfully saved 22.Jump.Street.2014.720p.BluRay.x264-SPARKS.srt at /home/Movies/
{% endcodeblock %}

{% codeblock subscene lang:ruby %}

#!/usr/bin/ruby
# Ashish Jaiswal
# Date 12-10-2014

require 'nokogiri'
require 'open-uri'
require 'zip'
require 'optparse'

class String
	def red;            "\033[31m#{self}\033[0m" end
	def green;          "\033[32m#{self}\033[0m" end
	def brown;          "\033[33m#{self}\033[0m" end
end

options = {} 

optparse = OptionParser.new do |opts|
	opts.banner = "*** Subcene subtitle downloader ***\nUsage : subscene -m deliver-us-from-evil -y 2014 -r 720".green

	opts.on("-m", "--movie_name movie_name", "Name of the Movie".brown) do |movie|
	options[:movie_name] =  movie
	end

	opts.on("-y", "--year year", "Year of the Movie".brown) do |year|
	options[:year] = year
	end

	opts.on("-r", "--resolution res", OptionParser::DecimalInteger, "Resolution of the Movie".brown) do |res|
	options[:resolution] = res
	end

	opts.on("-a", "--author author", "Author/Ripper of the Movie".brown) do |author|
	options[:author] = author
	end

	# This displays the help screen, all programs are assumed to have this option.
	opts.on( '-h', '--help', 'Display this screen'.brown ) do
	puts opts
	exit
	end
end

begin
	optparse.parse!
	mandatory = [:movie_name, :year, :resolution]
	missing = mandatory.select { | param | options[param].nil? }
	if not missing.empty?    
	puts "Missing options: #{missing.join(', ')}".red   
	puts optparse                                   
	exit                                            
	end
rescue OptionParser::ParseError, OptionParser::MissingArgument
	puts "Error: #{$!}"
	exit
end

# To open the link and parse
subscene_page = Nokogiri::HTML(open("http://subscene.com/subtitles/#{options[:movie_name]}/english"))

# To get the first Movie Name with Ripper
subscene_name = subscene_page.css('div.content.clearfix table tbody tr td.a1 a span').map { | p | p.text.gsub('English', "").strip }.grep(/#{options[:year]}.#{options[:resolution]}p.BluRay.x264-#{options[:author]}/).first

# Hash method used to bind name and url
a = Hash[subscene_page.search('div.content.clearfix table tbody tr td.a1 a').map { | i | [i.text.gsub('English', "").strip, i['href']] }]

# Getting the actual link 
subtitle_link = "http://subscene.com#{a.values_at("#{subscene_name}")}".gsub(/[\[\]\"]/, "")

# Parsing the actual link
download_page = Nokogiri::HTML(open("#{subtitle_link}"))

# Getting the download link
download_link = download_page.css('div.download a')[0]['href']

# Saving the file
open("#{subscene_name}.zip", 'wb') do |file|
	file << open("http://subscene.com/#{download_link}").read
end

# Unzipping it
def unzip_file (file, destination)
Zip::File.open(file) { |zip_file|
	zip_file.each { |f|
	f_path=File.join(destination, f.name)
	FileUtils.mkdir_p(File.dirname(f_path))
	zip_file.extract(f, f_path) unless File.exist?(f_path)
	$zipname = f.name
	}
}
end

# Remove the stall zip file after unzipping
def remove_file(file)
	  File.delete(file)
end

# Actual Movie to grep and copy file
movie_path = "#{options[:movie_name]}".gsub('-', " ")

# Get the movie path in /home/Movies directory
a = Dir.entries('/home/Movies').select {|entry| File.directory? File.join('/home/Movies',entry) and !(entry =='.' || entry == '..') }.grep(/#{movie_path}/i)
p = "/home/Movies/"
joinner = [ p, a].join

# Calling the defination
unzip_file("#{subscene_name}.zip", joinner)
puts "Successfully saved #{$zipname} at #{joinner}"
remove_file("#{subscene_name}.zip")
{% endcodeblock %}
