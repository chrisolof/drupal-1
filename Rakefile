# encoding: utf-8
require 'foodcritic'
require 'rspec/core/rake_task'
require 'rubocop/rake_task'
require 'erb'
require 'ostruct'
require 'chef/cookbook/metadata'

# Provides a basic Readme class so we can use a erb template.
class Readme < OpenStruct
  def render(template)
    ERB.new(template).result(binding)
  end
end

def rake_tasks
  documentation = ''
  s = `rake -T`.split("\n")
  s.each do |l|
    documentation << "    #{l}\n" if l =~ /^rake/
  end
  documentation
end

desc 'Run RuboCop style and lint checks'
Rubocop::RakeTask.new(:rubocop)

desc 'Run Foodcritic lint checks'
FoodCritic::Rake::LintTask.new(:foodcritic) do |t|
  t.options = { fail_tags: ['any'] }
end

desc 'Run ChefSpec examples'
RSpec::Core::RakeTask.new(:spec)

desc 'Generate the Readme.md file.'
task :readme do
  metadata = Chef::Cookbook::Metadata.new
  metadata.from_file('metadata.rb')
  authors = `git shortlog -sn`.b.scan(/[^\d\s].*/).map do |a|
    a == 'Making GitHub Delicious.' ? nil : a
  end
  markdown = Readme.new(
                         metadata: metadata,
                         rake_tasks: rake_tasks,
                         authors: authors)
  new_readme = markdown.render(File.read('templates/default/readme.md.erb'))
  File.open('README.md', 'w') { |file| file.write(new_readme) }
  puts new_readme
end

desc 'Run all tests'
task test: [:rubocop, :foodcritic, :spec]
task default: :test

begin
  require 'kitchen/rake_tasks'
  Kitchen::RakeTasks.new

  desc 'Alias for kitchen:all'
  task integration: 'kitchen:all'

  task test: :integration
rescue LoadError
  puts '>>>>> Kitchen gem not loaded, omitting tasks' unless ENV['CI']
end
