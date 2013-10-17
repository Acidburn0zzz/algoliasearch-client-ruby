# encoding: utf-8

require 'rubygems'
require 'bundler'
begin
  Bundler.setup(:default, :development)
rescue Bundler::BundlerError => e
  $stderr.puts e.message
  $stderr.puts "Run `bundle install` to install missing gems"
  exit e.status_code
end
require 'rake'

require File.join(__FILE__, '..', 'lib', 'algolia', 'version')

require 'jeweler'
Jeweler::Tasks.new do |gem|
  # gem is a Gem::Specification... see http://docs.rubygems.org/read/chapter/20 for more options
  gem.name = "algoliasearch"
  gem.homepage = "http://github.com/algolia/algoliasearch-client-ruby"
  gem.license = "MIT"
  gem.summary = %Q{A simple Ruby client for the algolia.com REST API}
  gem.description = %Q{A simple Ruby client for the algolia.com REST API}
  gem.email = "contact@algolia.com"
  gem.authors = ["Algolia"]
  gem.version = Algolia::VERSION
  # dependencies defined in Gemfile
end
Jeweler::RubygemsDotOrgTasks.new

require 'rake/testtask'
Rake::TestTask.new(:test) do |test|
  test.libs << 'lib' << 'test'
  test.pattern = 'test/**/test_*.rb'
  test.verbose = true
end

require 'rdoc/task'
Rake::RDocTask.new do |rdoc|
  version = Algolia::VERSION
  rdoc.rdoc_dir = 'rdoc'
  rdoc.title = "algoliasearch #{version}"
  rdoc.rdoc_files.include('README*')
  rdoc.rdoc_files.include('lib/**/*.rb')
end

require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new(:spec)

task :default => :spec
