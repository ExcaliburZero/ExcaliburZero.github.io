# encoding: utf-8

task :default => [:test]

require 'html-proofer'

task :test do
  sh "bundle exec jekyll build"
  sh "bundle exec htmlproofer '_site/' --allow-hash-href"
end
