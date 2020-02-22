require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"


# Change your GitHub reponame
GITHUB_REPONAME = "JeHuiPark.github.io"
GITHUB_USERNAME = "JeHuiPark"
# 운영환경 빌드 아웃풋 경로
PROD_DESTINATION = "_site_prod"

puts "레파지토리 경로 = https://github.com/#{GITHUB_USERNAME}/#{GITHUB_REPONAME}"
puts "브랜치명을 입력해주세요"
branch = $stdin.gets.chomp
puts "branch name = #{branch}"

namespace :site do
  desc "Generate blog files"
  task :generate do
    
    FileUtils.rm_rf("#{PROD_DESTINATION}")
    Jekyll::Site.new(Jekyll.configuration({
      "source"      => ".",
      "destination" => "#{PROD_DESTINATION}",
      "mode"=> "production"
    })).process
  end


  desc "Generate and publish blog"

  task :publish => [:generate] do
    Dir.mktmpdir do |tmp|
      cp_r "#{PROD_DESTINATION}/.", tmp

      pwd = Dir.pwd
      Dir.chdir tmp

      system "git config --global user.email \"pjh2359@gmail.com\""
      system "git config --global user.name \"#{GITHUB_USERNAME}\""
      system "git init"
      system "git add ."
      message = "Site updated at #{Time.now.utc}"
      system "git commit -m #{message.inspect}"
      # system "git remote add origin git@github-jehiupark:#{GITHUB_USERNAME}/#{GITHUB_REPONAME}.git"
      system "git remote add origin https://github.com/#{GITHUB_USERNAME}/#{GITHUB_REPONAME}.git"
      system "git push origin master -f"
    end
  end
end
