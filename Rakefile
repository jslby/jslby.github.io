require 'rubygems'
    require 'rake'
    require 'rdoc'
    require 'date'
    require 'yaml'
    require 'tmpdir'
    require 'jekyll'

    desc "Generate blog files"
    task :generate do
      Jekyll::Site.new(Jekyll.configuration({
        "source"      => ".",
        "destination" => "_site"
      })).process
    end


    desc "Generate and publish blog"
    task :publish => [:generate] do
      Dir.mktmpdir do |tmp|
        system "mv _site/* #{tmp}"
        system "mv #{tmp}/* ."
        message = "Site updated at #{Time.now.utc}"
        system "git add . --all"
        system "git commit -am #{message.shellescape}"
        system "echo Finish!"
      end
    end

task :default => :publish