require "bundler/setup"

require "git"
require "logger"
require "fileutils"


ELPR_REPO_PATH = File.expand_path(File.dirname(__FILE__)) + "/../elpr"
GH_PAGES_BRANCH = "gh-pages"
RUSTBOOK_CMD = "./bin/rustbook build ."

desc "Uses rustbook to build the HTML and copies it into the #{ELPR_REPO_PATH}"
task :build do
  `#{RUSTBOOK_CMD}`
  FileUtils.cp_r Dir.glob("css/*"), ELPR_REPO_PATH
  FileUtils.cp_r Dir.glob("_book/*"), ELPR_REPO_PATH
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
