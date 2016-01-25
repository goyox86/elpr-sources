require "bundler/setup"

require "git"
require "logger"
require "fileutils"


ELPR_REPO_PATH = File.expand_path(File.dirname(__FILE__)) + "/../elpr"
GH_PAGES_BRANCH = "gh-pages"
RUSTBOOK_CMD = "multirust run nightly rustbook build ."

desc "Uses rustbook to build the HTML and copies it into the #{ELPR_REPO_PATH}"
task :build do
  `#{RUSTBOOK_CMD}`
  fix_rustbook_css!
  FileUtils.cp Dir.glob("rust.css"), ELPR_REPO_PATH
  FileUtils.cp_r Dir.glob("_book/*"), ELPR_REPO_PATH
end

desc "Removes book files"
task :clean do
  FileUtils.rm_r Dir.glob("rust.css")
  FileUtils.rm_r Dir.glob("_book/")
end

desc "Build and publishes the book"
task publish: :build do
  g = Git.open(ELPR_REPO_PATH, :log => Logger.new(STDOUT))
  g.reset
  g.checkout(GH_PAGES_BRANCH)
  g.add(all: true)
  begin
    g.commit "Release #{Time.now}"
    g.pull(g.remote("origin"), GH_PAGES_BRANCH)
    g.push(g.remote("origin"), GH_PAGES_BRANCH)
  rescue Git::GitExecuteError => e
    puts "There was a Git error or there are no changes."
  end
end

def fix_rustbook_css!
  rustbook_css_path = File.expand_path(File.dirname(__FILE__)) + "/_book/rustbook.css"
  rustbook_css_text = File.read(rustbook_css_path)
  fixed_rustbook_css_text = rustbook_css_text.gsub("../rust.css", "./rust.css")
  File.open(rustbook_css_path, "w") { |file| file.puts fixed_rustbook_css_text }
end
