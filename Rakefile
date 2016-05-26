require 'bundler'
Bundler.require

require 'open-uri'
require 'pathname'

$tmp_dir = Pathname 'tmp'

$bootswatch_url = 'https://bootswatch.com/'
$repo = 'git@github.com:kayhide/middleman-cheir.git'
$repo_dir = $tmp_dir.join('repo')

def say status, message, color
  puts [
    Colored.colorize('%12s' % status, foreground: color),
    message
  ].join(' ')
end

desc 'List bootswatch themes'
task :list => [:fetch_themes] do
  puts @themes.keys
end

desc "Clone middleman-cheir repo to #{$repo_dir}"
task :clone do
  $tmp_dir.mkpath
  `git clone #{$repo} #{$repo_dir}`
end

directory $repo_dir => [:clone]

desc 'Update branches of local repo'
task :update => [$repo_dir, :fetch_themes] do
  @themes.each do |name, files|
    say :swatch, name, :cyan
    Dir.chdir $repo_dir do
      `git checkout master`
      `git checkout -b #{name}`
      files.each do |f|
        say :update, f, :green
        src = File.join $bootswatch_url, f
        dst = File.join 'source/stylesheets', File.basename(f)
        open(dst, 'w') { |o| open(src) { |i| o.write i.read } }
        `git add #{dst}`
      end
      `git commit -m 'swatch #{name.capitalize}'`
    end
  end
end

desc 'Push all branches of local repo'
task :push => [$repo_dir] do
  Dir.chdir $repo_dir do
    `git push origin --all`
  end
end

task :fetch_themes do
  @agent = Mechanize.new
  @doc = @agent.get $bootswatch_url
  @themes = @doc.links.map(&:href).grep(/\.scss$/).group_by{|f| f.split('/').first}
end
