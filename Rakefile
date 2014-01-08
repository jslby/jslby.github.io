require 'rubygems'
require 'rake'
require 'rubygems'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'
require 'fileutils'


    desc "Generate blog files"
    task :generate do
      Jekyll::Site.new(Jekyll.configuration({
        "source"      => ".",
        "destination" => "_site"
      })).process
    end


    desc "Generate and publish blog to gh-pages"
    task :publish => [:generate] do
      Dir.mktmpdir do |tmp|
        cp_r "_site/.", tmp
				Dir.chdir tmp
        message = "Site updated at #{Time.now.utc}"
        system "git add . --all"
        system "git commit -m #{message.shellescape}"
        system "git push --force"
        system "echo Finish!"
      end
    end

task :default => :publish
