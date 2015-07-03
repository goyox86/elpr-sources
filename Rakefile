require "bundler/setup"
require "git"
require "logger"

ELPR_REPO_PATH = "../elpr"
GH_PAGES_BRANCH = "gh-pages"
RUSTBOOK_CMD = "rustbook build ."

task default: %w[build]

desc "Uses rustbook to build the HTML and copies it into the #{ELPR_REPO_PATH}"
task :build do
  Kernel.exec RUSTBOOK_CMD
  FileUtils.cp_r "css/.", ELPR_REPO_PATH
  FileUtils.cp_r "_book/.", ELPR_REPO_PATH
end

desc "Build and publishes the book"
task publish: :build do
  g = Git.open(ELPR_REPO_PATH, :log => Logger.new(STDOUT))
  g.reset
  g.checkout(GH_PAGES_BRANCH)
  g.add(all: true)
  begin
    g.commit "Release #{Time.now}"
    g.push(g.remote("origin"), GH_PAGES_BRANCH)
  rescue Git::GitExecuteError => e
    puts "There was a Git error or simply no changes to be "
  end
end
