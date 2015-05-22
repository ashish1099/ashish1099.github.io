---
layout: post
title: "Ruby song downloader"
date: 2014-06-28 03:29:23 +0530
comments: true
categories: [ruby, script]
---
Since I'm working on puppet DSL language from past year, so always wanted to take on `ruby`. Finally got the chances to do some task.
I have had lost many of my songs library. So wanted to keep track of everything and needed to start from scratch. As I love Hindi/Bollywood 
songs, the only site comes to my mind is `songspk.name` and I had some trouble accessing it, due to Indian goverment blocking or either my ISP.

To get out of this, I found a site `allmp3song.com`, which is truly simple and fast. The only problem was, I can't download the whole movie songs at a time.
<!-- more --> You have to click on each songs and download it manually, which was very frustating. So thought of writing a script. I wrote one in bash which was quite simple by using `wget`. Then I took this as a self owned task in. 


Below is the script, which I wrote, please let me know if any improvement can be done. So far its working like charm..

{% codeblock mp3song.rb lang:ruby %}
#!/usr/bin/ruby
# Date : 18/04/2014
# Author : Ashish Jaiswal
# Email ID : ashish1099@gmail.com
# Purpose : Learning some ruby basics.

require 'nokogiri'
require 'open-uri'
require 'net/http'

filelist_id = ARGV[0]

if ARGV.empty?
	puts " Sorry..... Please read this carefully

Please put album code. To find album code, go to any browser and search for required album.
Example : 
	http://allmp3song.com/filelist/3330/arijit_singh_-_mtv_unplugged_season_3_%282013%29/new2old/1
		
So the album code is 3330"
exit
end

def fetch(uri_str, songs_name, song_name, limit = 10)
	# You should choose a better exception.
	raise ArgumentError, 'too many HTTP redirects' if limit == 0

		encoded_url = URI.encode(uri_str)
		url = URI.parse(encoded_url)	
		req = Net::HTTP::Get.new(url.path, { 'User-Agent' => "Ruby/#{RUBY_VERSION}" })
		response = Net::HTTP.start(url.host, url.port) do |http| 
			http.request(req)
		end
	
	case response
		when Net::HTTPSuccess then
			response
		when Net::HTTPRedirection then
			location = response['location']
			puts "Started downloading ====> ===> ==> #{song_name}\n\n"
			begin
				File.open(songs_name, "wb") do | saved_file |
        	                    open((URI.parse(URI.encode("#{location}"))), "rb") do | read_file |
	                                saved_file.write(read_file.read)					
        	                              end # End of read_file
                	            end # End of saved_file
				puts "Saved as #{song_name}"	
				puts "=============================================================================================================="
			rescue OpenURI::HTTPError
				puts "Song is not present on the server.. Sorry :)\n\n"
			end
		else
			response.value
		end
end


BASE_MP3SONGS_URL = "http://allmp3song.com"
FILELIST_URL = "#{BASE_MP3SONGS_URL}/filelist/#{filelist_id}/*/new2old/1"

# Calling NOKOGIRI to parse HTML 
songs_list = Nokogiri::HTML(open(FILELIST_URL))

# Get the album name
album_name = songs_list.css('div#mainDiv h2').text.strip.gsub(/\s: Mp3 Songs/, "")
album_path = "/home/songs/#{album_name}/"
Dir.mkdir(album_path) unless File.exists?(album_path)

# Get the list of page of particular albums.
pagination = songs_list.css('div.pgn').text.strip.gsub(/\D/, "").slice!(1).to_i

# Run a loop to get the songs_id and based on id get the songs_name
# Example http://allmp3song.com/files/download/id/24398

(0..10).each do | i |
	begin	
		song_id = songs_list.css("a.fileName")[i]["href"].match(/\d{3,}/)
		between_url = "/files/download/id/"
		url_download = [ BASE_MP3SONGS_URL, between_url, song_id ].join
		song_name = songs_list.css("a.fileName div > text()").text.strip.split("| ").fetch(i).gsub(/\D$/, "")
		songs_name = "#{album_path}#{song_name}"
		print fetch(url_download, songs_name, song_name)
	rescue NoMethodError
	end # End of BEGIN
end # End of DO

# Looking for Second page.
if pagination > 1
	FILELIST_URL02 = "#{BASE_MP3SONGS_URL}/filelist/#{filelist_id}/*/new2old/2"
	songs_list02 = Nokogiri::HTML(open(FILELIST_URL02))
(0..10).each do | i |
	begin	
		song_id = songs_list02.css("a.fileName")[i]["href"].match(/\d{3,}/)
		between_url = "/files/download/id/"
		url_download = [ BASE_MP3SONGS_URL, between_url, song_id ].join
		song_name = songs_list02.css("a.fileName div > text()").text.strip.split("| ").fetch(i).gsub(/\D$/, "")
		songs_name = "#{album_path}#{song_name}"
		print fetch(url_download, songs_name, song_name)
	rescue NoMethodError
		end # End of BEGIN
	end # End of DO
end # End of IF
{% endcodeblock %}
