---
layout: posts
title:  "Github 다중계정 관리"
date:   2020-02-23 11:42:00 +0900
comments: true
categories: note
tags: 
  - ssh
---

* TOC
{:toc}

## 깃헙 계정이 여러개일 땐 어떻게 해야할까?
`Basic Credentials` 방식을 이용할 할 때는 레파지토리별로 로그인이 가능하기 때문에 별문제가 되지 않는다.  
하지만, ssh 방식을 이용하게 된다면?? 아래와 같이 추가적인 작업이 필요하다.   
1. ssh 키 생성
1. ssh 공개키를 깃헙 설정에 추가
1. ssh 비공개키를 ssh-agent에 등록
1. ssh-agent 설정에 Github 호스트 추가

### ssh 키 생성  
`ssh-keygen -t rsa -C "pjh2359@gmail.com" -f ~/.ssh/example_key`  
![image](https://user-images.githubusercontent.com/25237661/75110717-eb2cd380-5674-11ea-91fb-a8d09dae6e6b.png)  
`example_key` 와 `example_key.pub` 이 생성된다.  
![image](https://user-images.githubusercontent.com/25237661/75110735-419a1200-5675-11ea-8c7a-9cd6ec51e514.png)  
각각 `비공개키`, `공개키`로 이해하면 된다.

### ssh 공개키를 깃헙 설정에 추가
우선 두가지 방법이 있는듯 하다.  
- 계정 설정에서 ssh 키 추가하기
- 레파지토리 설정에서 `deploy key` 설정하기

나는 두가지 계정 설정에 ssh 키를 추가하는 방법을 선택하여 진행했다.  

![image](https://user-images.githubusercontent.com/25237661/75110801-ff250500-5675-11ea-8abf-2bfa5cbb37bc.png)  
![image](https://user-images.githubusercontent.com/25237661/75110827-4a3f1800-5676-11ea-8bef-60f11f2452bf.png)  
![image](https://user-images.githubusercontent.com/25237661/75110840-85414b80-5676-11ea-8540-c5c9a0281753.png)

제목은 편하게 작성하고, `key`입력란에 미리 복사해둔 공개키를 붙여놓고 저장한다.

### ssh 비공개키를 ssh-agent에 등록  
`ssh-add example_key`  
![image](https://user-images.githubusercontent.com/25237661/75110921-92ab0580-5677-11ea-8376-b5da817c72c9.png)

### ssh-agent 설정에 Github 호스트 추가  
``` sh
cd ~/.ssh
vi config
```  
<br>  
**`config`** 파일 작성 (ssh-agent 설정파일)  
``` sh
# 회사계정
Host github.com
    HostName github.com
    User git
    IdentityFile another_key
# 개인계정
Host github-jehuipark
    HostName github.com
    User git
    IdentityFile example_key
```

## remote 저장소 등록
ssh 인증방식을 이용할 때는 원격 저장소 위치를 ssh 규격에 맞추어 작성하게 된다.  

- 회사계정으로 접근가능한 레파지토리를 등록하자  
`git remote add ${remote-name} git@github.com:${github-username}/${github-reponame}`

- 개인계정으로 접근가능한 레파지토리를 등록하자  
`git remote add ${remote-name} git@github-jehuipark:${github-username}/${github-reponame}`

### 무엇이 다른가?
`git@github.com`: github.com 으로 포워딩 하며, `another_key` 키를 사용하도록 한다.  
`git@github-jehuipark.com`: github.com 으로 포워딩 하며, `example_key` 키를 사용하도록 한다.  


## 참조
> 
https://medium.com/@therajanmaurya/git-push-pull-with-two-different-account-and-two-different-user-on-same-machine-a85f9ee7ec61