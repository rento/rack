# Rakefile for Rack.  -*-ruby-*-
require 'rake/rdoctask'
require 'rake/testtask'


desc "Run all the tests"
task :default => [:test]

desc "Make an archive as .tar.gz"
task :dist => [:chmod, :changelog, :rdoc, "SPEC"] do
  FileUtils.touch("RDOX")
  sh "git archive --format=tar --prefix=#{release}/ HEAD^{tree} >#{release}.tar"
  sh "pax -waf #{release}.tar -s ':^:#{release}/:' RDOX SPEC ChangeLog doc rack.gemspec"
  sh "gzip -f -9 #{release}.tar"
end

desc "Make an official release"
task :officialrelease do
  puts "Official build for #{release}..."
  sh "rm -rf stage"
  sh "git clone --shared . stage"
  sh "cd stage && rake officialrelease_really"
  sh "mv stage/#{release}.tar.gz stage/#{release}.gem ."
end

task :officialrelease_really => ["SPEC", :dist, :gem] do
  sh "sha1sum #{release}.tar.gz #{release}.gem"
end

def release
  require File.dirname(__FILE__) + "/lib/rack"
  "rack-#{Rack.release}"
end

desc "Make binaries executable"
task :chmod do
  Dir["bin/*"].each { |binary| File.chmod(0775, binary) }
  Dir["test/cgi/test*"].each { |binary| File.chmod(0775, binary) }
end

desc "Generate a ChangeLog"
task :changelog do
  File.open("ChangeLog", "w") { |out|
    `git log -z`.split("\0").map { |chunk|
      author = chunk[/Author: (.*)/, 1].strip
      date = chunk[/Date: (.*)/, 1].strip
      desc, detail = $'.strip.split("\n", 2)
      detail ||= ""
      detail = detail.gsub(/.*darcs-hash:.*/, '')
      detail.rstrip!
      out.puts "#{date}  #{author}"
      out.puts "  * #{desc.strip}"
      out.puts detail  unless detail.empty?
      out.puts
    }
  }
end


desc "Generate RDox"
task "RDOX" do
  sh "specrb -rtest/gemloader -Ilib:test -a --rdox >RDOX"
end

desc "Generate Rack Specification"
task "SPEC" do
  File.open("SPEC", "wb") { |file|
    IO.foreach("lib/rack/lint.rb") { |line|
      if line =~ /## (.*)/
        file.puts $1
      end
    }
  }
end

desc "Run all the fast tests"
task :test do
  sh "specrb -Ilib:test -w #{ENV['TEST'] || '-a'} #{ENV['TESTOPTS'] || '-t "^(?!Rack::Handler|Rack::Adapter|Rack::Session::Memcache|rackup)"'}"
end

desc "Run all the tests"
task :fulltest => [:chmod] do
  sh "specrb -rtest/gemloader -Ilib:test -w #{ENV['TEST'] || '-a'} #{ENV['TESTOPTS']}"
end

task :gem => ["SPEC"] do
  FileUtils.touch("RDOX")
  sh "gem build rack.gemspec"
end

desc "Generate RDoc documentation"
task :rdoc do
  sh(*%w{rdoc --line-numbers --main README
              --title 'Rack\ Documentation' --charset utf-8 -U -o doc} +
              %w{README KNOWN-ISSUES SPEC} +
              Dir["lib/**/*.rb"])
end

task :pushsite => [:rdoc] do
  sh "cd site && git gc"
  sh "rsync -avz doc/ chneukirchen@rack.rubyforge.org:/var/www/gforge-projects/rack/doc/"
  sh "rsync -avz site/ chneukirchen@rack.rubyforge.org:/var/www/gforge-projects/rack/"
  sh "cd site && git push"
end
