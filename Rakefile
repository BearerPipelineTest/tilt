require 'rake/testtask'
task :default => [:test]

# SPECS =====================================================================

desc 'Run tests (default)'
Rake::TestTask.new(:test) do |t|
  t.test_files = FileList['test/*_test.rb']
  t.warning = false
end

# DOCUMENTATION =============================================================

begin
  require 'yard'
  YARD::Rake::YardocTask.new do |t|
    t.files = [
      'lib/tilt.rb', 'lib/tilt/mapping.rb', 'lib/tilt/template.rb',
      '-',
      '*.md', 'docs/*.md',
    ]

    t.options <<
      '--no-private' <<
      '--protected' <<
      '-m' << 'markdown' <<
      '--asset' << 'docs/common.css:css/common.css'
  end
rescue LoadError
end

task :man do
  require 'ronn'
  ENV['RONN_MANUAL'] = "Tilt Manual"
  ENV['RONN_ORGANIZATION'] = "Tilt #{SPEC.version}"
  sh "ronn -w -s toc -r5 --markdown man/*.ronn"
end

# PACKAGING =================================================================

if defined?(Gem)
  SPEC = eval(File.read('tilt.gemspec'))

  def package(ext='')
    "pkg/tilt-#{SPEC.version}" + ext
  end

  desc 'Build packages'
  task :package => package('.gem')

  desc 'Build and install as local gem'
  task :install => package('.gem') do
    sh "gem install #{package('.gem')}"
  end

  directory 'pkg/'

  file package('.gem') => %w[pkg/ tilt.gemspec] + SPEC.files do |f|
    sh "gem build tilt.gemspec"
    mv File.basename(f.name), f.name
  end
end

# GEMSPEC ===================================================================

file 'tilt.gemspec' => FileList['{lib,test}/**','Rakefile'] do |f|
  # read version from tilt.rb
  version = File.read('lib/tilt.rb')[/VERSION = '(.*)'/] && $1
  # read spec file and split out manifest section
  spec = File.
    read(f.name).
    sub(/s\.version\s*=\s*'.*'/, "s.version = '#{version}'")
  parts = spec.split("  # = MANIFEST =\n")
  # determine file list from git ls-files
  files = `git ls-files -- lib bin COPYING`.
    split("\n").sort.reject{ |file| file =~ /^\./ }.
    map{ |file| "    #{file}" }.join("\n")
  # piece file back together and write...
  parts[1] = "  s.files = %w[\n#{files}\n  ]\n"
  spec = parts.join("  # = MANIFEST =\n")
  spec.sub!(/s.date = '.*'/, "s.date = '#{Time.now.strftime("%Y-%m-%d")}'")
  File.open(f.name, 'w') { |io| io.write(spec) }
  puts "updated #{f.name}"
end
