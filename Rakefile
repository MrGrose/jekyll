# frozen_string_literal: true

require "rubygems"
require "rake"
require "rdoc"
require "date"
require "yaml"

$LOAD_PATH.unshift File.expand_path("lib", __dir__)
require "jekyll/version"

Dir.glob("rake/**.rake").each { |f| import f }

#############################################################################
#
# Helper functions
#
#############################################################################

def name
  "jekyll"
end

def version
  Jekyll::VERSION
end

def docs_name
  "#{name}-docs"
end

def docs_folder
  "docs"
end

def gemspec_file
  "#{name}.gemspec"
end

def gem_file
  "#{name}-#{Gem::Version.new(version)}.gem"
end

def normalize_bullets(markdown)
  markdown.gsub(%r!\n\s{2}\*{1}!, "\n-")
end

def linkify_prs(markdown)
  markdown.gsub(%r!(?<\!&)#(\d+)!) do |word|
    "[#{word}]({{ site.repository }}/issues/#{word.delete("#")})"
  end
end

def linkify(markdown)
  linkify_prs(markdown)
end

def liquid_escape(markdown)
  markdown.gsub(%r!(`{[{%].+[}%]}`)!, "{% raw %}\\1{% endraw %}")
end

def custom_release_header_anchors(markdown)
  header_regexp = %r!^(\d{1,2})\.(\d{1,2})\.(\d{1,2}) \/ \d{4}-\d{2}-\d{2}!
  section_regexp = %r!^### \w+ \w+$!
  markdown.split(%r!^##\s!).map do |release_notes|
    _, major, minor, patch = *release_notes.match(header_regexp)
    release_notes
      .gsub(header_regexp, "\\0\n{: #v\\1-\\2-\\3}")
      .gsub(section_regexp) { |section| "#{section}\n{: ##{slugify(section)}-v#{major}-#{minor}-#{patch}}" }
  end.join("\n## ")
end

def slugify(header)
  header.delete("#").strip.downcase.gsub(%r!\s+!, "-")
end

def remove_head_from_history(markdown)
  index = markdown =~ %r!^##\s+\d+\.\d+\.\d+!
  markdown[index..-1]
end

def converted_history(markdown)
  remove_head_from_history(
    custom_release_header_anchors(
      liquid_escape(
        linkify(
          normalize_bullets(markdown)
        )
      )
    )
  )
end

def siteify_file(file, overrides_front_matter = {})
  abort "You seem to have misplaced your #{file} file. I can haz?" unless File.exist?(file)
  title = begin
            File.read(file).match(%r!\A# (.*)$!)[1]
          rescue NoMethodError
            File.basename(file, ".*").downcase.capitalize
          end
  slug  = File.basename(file, ".markdown").downcase
  front_matter = {
    "title"     => title,
    "permalink" => "/docs/#{slug}/",
    "note"      => "This file is autogenerated. Edit /#{file} instead.",
  }.merge(overrides_front_matter)
  contents = "#{front_matter.to_yaml}---\n\n#{content_for(file)}"
  File.write("#{docs_folder}/_docs/#{slug}.md", contents)
end

def content_for(file)
  contents = File.read(file)
  case file
  when "History.markdown"
    converted_history(contents)
  else
    contents.gsub(%r!\A# .*\n\n?!, "")
  end
end

#############################################################################
#
# Standard tasks
#
#############################################################################

multitask :default => [:test, :features]

task :spec => :test
require "rake/testtask"
Rake::TestTask.new(:test) do |test|
  test.libs << "lib" << "test"
  test.pattern = "test/**/test_*.rb"
  test.verbose = true
end

require "rdoc/task"
Rake::RDocTask.new do |rdoc|
  rdoc.rdoc_dir = "rdoc"
  rdoc.title = "#{name} #{version}"
  rdoc.rdoc_files.include("README*")
  rdoc.rdoc_files.include("lib/**/*.rb")
end

begin
  require "cucumber/rake/task"
  Cucumber::Rake::Task.new(:features) do |t|
    t.profile = "travis"
  end
  Cucumber::Rake::Task.new(:"features:html", "Run Cucumber features and produce HTML output") do |t|
    t.profile = "html_report"
  end
rescue LoadError
  desc "Cucumber rake task not available"
  task :features do
    abort "Cucumber rake task is not available. Be sure to install cucumber as a gem or plugin"
  end
end

desc "Open an irb session preloaded with this library"
task :console do
  sh "irb -rubygems -r ./lib/#{name}.rb"
end
