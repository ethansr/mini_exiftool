# Rakefile
require "rake/testtask"
require "rake/clean"
require "rake/rdoctask"
require "rake/gempackagetask"
#---
# The name of your project
PROJECT = "MiniExiftool"

# Your name, used in packaging.
MY_NAME = "Jan Friedrich"

# Your email address, used in packaging.
MY_EMAIL = "janfri@web.de"

# Short summary of your project, used in packaging.
PROJECT_SUMMARY = 'A library for nice OO access to the Exiftool commandline application written by Phil Harvey.'

# The project's package name (as opposed to its display name). Used for
# RubyForge connectivity and packaging.
UNIX_NAME = "mini_exiftool"
UNIX_NAME_RUBYFORGE = "miniexiftool"

# Your RubyForge user name.
RUBYFORGE_USER = ENV["RUBYFORGE_USER"] || "janfri"

# Directory on RubyForge where your website's files should be uploaded.
WEBSITE_DIR = "doc"

# Output directory for the rdoc html files.
# If you don't have a custom homepage, and want to use the RDoc
# index.html as homepage, just set it to WEBSITE_DIR.
RDOC_HTML_DIR = "#{WEBSITE_DIR}"
#---
# Variable settings for extension support.
EXT_DIR = "ext"
HAVE_EXT = File.directory?(EXT_DIR)
EXTCONF_FILES = FileList["#{EXT_DIR}/**/extconf.rb"]
EXT_SOURCES = FileList["#{EXT_DIR}/**/*.{c,h}"]
# Eventually add other files from EXT_DIR, like "MANIFEST"
EXT_DIST_FILES = EXT_SOURCES + EXTCONF_FILES
#---
REQUIRE_PATHS = ["lib"]
REQUIRE_PATHS << EXT_DIR if HAVE_EXT
$LOAD_PATH.concat(REQUIRE_PATHS)
# This library file defines the MyProject::VERSION constant.
require "#{UNIX_NAME}"
PROJECT_VERSION = eval("#{PROJECT}::Version") # e.g. "1.0.2"
#---
# Clobber object files and Makefiles generated by extconf.rb.
CLOBBER.include("#{EXT_DIR}/**/*.{so,dll,o}", "#{EXT_DIR}/**/Makefile")
# Clobber .config generated by setup.rb.
CLOBBER.include(".config")
#---
# Options common to RDocTask AND Gem::Specification.
#   The --main argument specifies which file appears on the index.html page
GENERAL_RDOC_OPTS = {
  "--title" => "#{PROJECT} API documentation",
  "--main" => "README"
}

# Additional RDoc formatted files, besides the Ruby source files.
RDOC_FILES = FileList["README"]
# Remove the following line if you don't want to extract RDoc from
# the extension C sources.
RDOC_FILES.include(EXT_SOURCES)

# Ruby library code.
LIB_FILES = FileList["lib/**/*.rb"]

# Filelist with Test::Unit test cases.
TEST_FILES = FileList["test/**/test_*.rb"]

# Executable scripts, all non-garbage files under bin/.
BIN_FILES = FileList["bin/*"]

# This filelist is used to create source packages.
# Include all Ruby and RDoc files.
DIST_FILES = FileList["**/*.rb", "**/*.rdoc"]
DIST_FILES.include("Rakefile", "COPYING")
DIST_FILES.include(BIN_FILES)
DIST_FILES.include("data/**/*", "test/data/**/*")
DIST_FILES.include("#{WEBSITE_DIR}/**/*.{html,css}", "man/*.[0-9]")
# Don't package files which are autogenerated by RDocTask
DIST_FILES.exclude(/^(\.\/)?#{RDOC_HTML_DIR}(\/|$)/)
# Include extension source files.
DIST_FILES.include(EXT_DIST_FILES)
# Don't package temporary files, perhaps created by tests.
DIST_FILES.exclude("**/temp_*", "**/*.tmp")
# Don't get into recursion...
DIST_FILES.exclude(/^(\.\/)?pkg(\/|$)/)
#---
# Run the tests if rake is invoked without arguments.
task "default" => ["test"]

test_task_name = HAVE_EXT ? "run-tests" : "test"
Rake::TestTask.new(test_task_name) do |t|
  t.test_files = TEST_FILES
  t.libs = REQUIRE_PATHS
end
#---
# Set an environment variable with any configuration options you want to
# be passed through to "setup.rb config".
CONFIG_OPTS = ENV["CONFIG"]
if HAVE_EXT
  file_create ".config" do
    ruby "setup.rb config #{CONFIG_OPTS}"
  end
  
  desc "Configure and make extension. " +
    "The CONFIG variable is passed to `setup.rb config'"
  task "make-ext" => ".config" do
    # The -q option suppresses messages from setup.rb.
    ruby "setup.rb -q setup"
  end
  
  desc "Run tests after making the extension."
  task "test" do
    Rake::Task["make-ext"].invoke
    Rake::Task["run-tests"].invoke
  end
end
#---
# The "rdoc" task generates API documentation.
Rake::RDocTask.new("rdoc") do |t|
  t.rdoc_files = RDOC_FILES + LIB_FILES
  t.title = GENERAL_RDOC_OPTS["--title"]
  t.main = GENERAL_RDOC_OPTS["--main"]
  t.rdoc_dir = RDOC_HTML_DIR
end
#---
GEM_SPEC = Gem::Specification.new do |s|
  s.name = UNIX_NAME
  s.version = PROJECT_VERSION
  s.summary = PROJECT_SUMMARY
  s.rubyforge_project = UNIX_NAME_RUBYFORGE
  s.homepage = "http://#{UNIX_NAME_RUBYFORGE}.rubyforge.org/"
  s.author = MY_NAME
  s.email = MY_EMAIL
  s.files = DIST_FILES
  s.test_files = TEST_FILES
  s.executables = BIN_FILES.map { |fn| File.basename(fn) }
  s.has_rdoc = true
  s.extra_rdoc_files = RDOC_FILES
  s.rdoc_options = GENERAL_RDOC_OPTS.to_a.flatten
  if HAVE_EXT
    s.extensions = EXTCONF_FILES
    s.require_paths << EXT_DIR
  end
end

# Now we can generate the package-related tasks.
Rake::GemPackageTask.new(GEM_SPEC) do |pkg|
  pkg.need_zip = true
  pkg.need_tar = true
end
#---
desc "Upload website to RubyForge. " +
  "scp will prompt for your RubyForge password."
task "publish-website" => ["rdoc"] do
  rubyforge_path = "/var/www/gforge-projects/#{UNIX_NAME_RUBYFORGE}/"
  sh "scp -r #{WEBSITE_DIR}/* " +
    "#{RUBYFORGE_USER}@rubyforge.org:#{rubyforge_path}",
    :verbose => true
end
#---
task "rubyforge-setup" do
  unless File.exist?(File.join(ENV["HOME"], ".rubyforge"))
    puts "rubyforge will ask you to edit its config.yml now."
    puts "Please set the `username' and `password' entries"
    puts "to your RubyForge username and RubyForge password!"
    puts "Press ENTER to continue."
    $stdin.gets
    sh "rubyforge setup", :verbose => true
  end
end

task "rubyforge-login" => ["rubyforge-setup"] do
  # Note: We assume that username and password were set in
  # rubyforge's config.yml.
  sh "rubyforge login", :verbose => true
end

task "publish-packages" => ["package", "rubyforge-login"] do
  # Upload packages under pkg/ to RubyForge
  # This task makes some assumptions:
  # * You have already created a package on the "Files" tab on the
  #   RubyForge project page. See pkg_name variable below.
  # * You made entries under package_ids and group_ids for this
  #   project in rubyforge's config.yml.  If not, eventually read
  #   "rubyforge --help" and then run "rubyforge setup".
  pkg_name = ENV["PKG_NAME"] || UNIX_NAME
  cmd = "rubyforge add_release miniexiftool mini_exiftool " +
        "#{PROJECT_VERSION} #{UNIX_NAME}-#{PROJECT_VERSION}"
  cd "pkg" do
  sh(cmd + ".gem", :verbose => true)
    sh(cmd + ".tgz", :verbose => true)
    sh(cmd + ".zip", :verbose => true)
  end
end
#---
# The "prepare-release" task makes sure your tests run, and then generates
# files for a new release.
desc "Run tests, generate RDoc and create packages."
task "prepare-release" => ["clobber"] do
  puts "Preparing release of #{PROJECT} version #{VERSION}"
  Rake::Task["test"].invoke
  Rake::Task["rdoc"].invoke
  Rake::Task["package"].invoke
end

# The "publish" task is the overarching task for the whole project. It
# builds a release and then publishes it to RubyForge.
desc "Publish new release of #{PROJECT}"
task "publish" => ["prepare-release"] do
  puts "Uploading documentation..."
  Rake::Task["publish-website"].invoke
  puts "Checking for rubyforge command..."
  `rubyforge --help`
  if $? == 0
    puts "Uploading packages..."
    Rake::Task["publish-packages"].invoke
    puts "Release done!"
  else
    puts "Can't invoke rubyforge command."
    puts "Either install rubyforge with 'gem install rubyforge'"
    puts "and retry or upload the package files manually!"
  end
end
