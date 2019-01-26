---
layout: posts
title:  지킬 블로그를 깃헙 페이지로 배포하기
date:   2019-01-13 20:50:21 +0900
comments: true
categories: blog
tags:
  - jekyll
---

지킬 패키지로 작성된 블로그를 깃허브에 배포하는 방법에는 두 가지가 있습니다.

- master 브랜치로 빌드전 소스를 commit - push 하기

- master 브랜치로 빌드완료된 정적 페이지 소스를 commit - push 하기

원래는 첫번째 방법으로 배포를 수행하다가 불편한점이 한 두가지가 아니여서, 두번째 방법으로 배포하는것으로 방법을 변경했습니다.

1. **root경로에 Rakefile이 존재한다면 아래와 같이 작성해주세요. (_Rakefile이 없다면 생성후 진행해주세요._)**

    ```ruby
    require "rubygems"
    require "tmpdir"

    require "bundler/setup"
    require "jekyll"


    # Change your GitHub reponame
    GITHUB_REPONAME = "JeHuiPark/JeHuiPark.github.io"
    # 운영환경 빌드 아웃풋 경로
    PROD_DESTINATION = "_site_prod"

    puts "레파지토리 경로 = https://github.com/#{GITHUB_REPONAME}"
    puts "브랜치명을 입력해주세요"
    branch = $stdin.gets.chomp
    puts "branch name = #{branch}"
    puts "계속 진행하시려면 아무키나 입력해주세요"
    system "pause"

    namespace :site do
      desc "Generate blog files"
      task :generate do
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

          system "git init"
          system "git add ."
          message = "Site updated at #{Time.now.utc}"
          system "git commit -m #{message.inspect}"
          system "git remote add origin https://github.com/#{GITHUB_REPONAME}.git"
          system "git push origin master:refs/heads/master --force"
          system "pause"

          Dir.chdir pwd
        end
      end
    end
    ```

1. **아래와 같이 batch파일을 작성한 후 저장합니다.**

    ```batch
    @echo off
    chcp 65001
    :: batch 파일 위치에 따라 아래 라인을 수정해주세요.
    cd JeHuiPark.github.io
    rake site:publish
    ```

1. **batch 파일을 실행후 아래와 같이 진행**

    ![publ_step1](https://user-images.githubusercontent.com/25237661/51084921-67b46980-1775-11e9-91e9-4136de6f6f27.jpg)
