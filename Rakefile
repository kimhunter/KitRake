# Copyright (c) 2011 Kim Hunter.
# All rights reserved.
# 
# Redistribution and use in source and binary forms are permitted provided
# that the above copyright notice and this paragraph are duplicated in all
# such forms and that any documentation, advertising materials, and other
# materials related to such distribution and use acknowledge that the
# software was developed by the Kim Hunter. The name of the copyright holder
# may not be used to endorse or promote products derived from this software
# without specific prior written permission.
# 
# THIS SOFTWARE IS PROVIDED ``AS IS'' AND WITHOUT ANY EXPRESS OR
# IMPLIED WARRANTIES, INCLUDING, WITHOUT LIMITATION, THE IMPLIED
# WARRANTIES OF MERCHANTIBILITY AND FITNESS FOR A PARTICULAR PURPOSE.

require 'rubygems'
require 'rake/clean'
require 'fileutils'
require 'yaml'
require "open-uri"

SRC_FOLDER = "src"
EXCLUDE_PATTERNS = /(Reachability\.[hm])/
SOURCE_FILES = FileList["./*.[hmc]"].select {|f| !(f =~ EXCLUDE_PATTERNS) }

CLEAN.include([SRC_FOLDER])

def inc_revision version
  v = version.to_s.split('.')
  if v.size > 1
    last = v.size - 1 
    if v.last =~ /^[0-9]+(.*)?$/
      v[last] = (v.last.to_i + 1).to_s
      v[last] += $1.to_s unless $1.nil?
    end
  end
  v.join '.'
end

desc "Create KitSpec"
task :kitspec do
  unless File.exists? 'KitSpec'
    kspec = {} 
    kspec['name'] = File.basename Dir.pwd.downcase
    kspec['version'] = 0.1
    f = File.open('KitSpec', 'w')
    f.write kspec.to_yaml
    f.close    
  end
end

desc "Publish"
task :publish => [:kitspec, :kit] do
  `kit publish-local`
end

desc "Bump Version"
task :bump  do
  kitspec = YAML::load_file 'KitSpec'
  old = kitspec['version']
  kitspec['version'] = inc_revision(old)
  f = File.open('KitSpec', 'w')
  f.write kitspec.to_yaml
  f.close
  puts "Updated from #{old} -> #{kitspec['version']}"
end

desc "Find Dups"
task :find_dups do
  basenames = SOURCE_FILES.map {|path| File.basename path}
  dups = basenames.select {|fname| basenames.count(fname) > 1 }
  raise "Duplicates Found CANNOT CONTINUE Dups: \n#{dups.uniq}" unless dups.size.zero?
end


desc "Transform project into Kit Package"
task :kit => [:clean, :find_dups] do
  Dir.mkdir SRC_FOLDER
  SOURCE_FILES.each do |orig_file| 
    FileUtils.cp orig_file, "#{SRC_FOLDER}/#{File.basename(orig_file)}"
  end
  puts "Copied #{SOURCE_FILES.size} files"
end

desc "Get latest rakefile from github"
task :selfupdate do
  url = 'https://github.com/bigkm/KitRake/raw/master/Rakefile'
  file = open url
  content = file.read
  f = File.open __FILE__, 'w'
  f.write content
  f.close
  puts 'RakeFile Updated'
end
