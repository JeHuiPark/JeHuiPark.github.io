---
layout: posts
title:  "Vim 어린이가 만든 GitHub Co-Author Vim Plugin"
date:   2020-03-29 14:00:00 +0900
comments: true
categories: note
tags: 
  - vim
  - github
---

* TOC
{:toc}

## GitHub Co-Author?
`Co-Author` 의 사전적 의미를 네이버에 검색하면 공동저자라고 나온다.  
![image](https://user-images.githubusercontent.com/25237661/77842393-8f51ef00-71cc-11ea-94fb-005522612e99.png){:width="300px"}

GitHub 는 이렇게 하나의 커밋에 2명 이상의 기여자가 존재할 경우에 누가 이 커밋에 기여했는지 UI로 표현해주는 기능을 제공한다.  
![image](https://user-images.githubusercontent.com/25237661/77842434-06878300-71cd-11ea-846c-2f2ad3e8407e.png)

## 난 이 기능이 좋다.
현재 회사에서는 코드를 작성하면 모든 코드는 빠짐없이 PR 과정을 거치게 된다.  
그렇다보니, 내 코드에 다른 사람의 의견이 반영되는 경우도 빈번하게 발생 하는데, 이런 경우에 GitHub 에서 지원하는 `Co-Author` 기능을 이용하면 해당 변경사항에 어떤 사람이 기여했는지 기록을 남기는 동시에 UI 로 표시까지 해주니 금상첨화다. 

### Co-Author 표시하기
간단하다*(귀찮다)* 커밋 메시지를 작성할 때 GitHub 에서 [가이드](https://help.github.com/en/github/committing-changes-to-your-project/creating-a-commit-with-multiple-authors) 하는 포멧에 맞추어 작성하면 된다.

### 막상 작성을 해보려고 하면?
간단한데, 막상 작성하려고 하면 되게 귀찮은 작업이다. 
팀원의 GitHub 계정명도 알아야하고, 이메일정보도 알아야 하는데 어디에 복사해 두고 하자니 커밋 메시지 작성하는데 한 세월이 걸릴 것 같은 느낌적인 느낌이 들었다. (완전 노가다)
![image](https://user-images.githubusercontent.com/25237661/77842923-d0e59880-71d2-11ea-8124-28bcd51cb7d7.png)

그래서, 커밋 메시지에 컨트리뷰터 작성할 때 꿀팁이 있는지 [기계인간 존그립](https://johngrib.github.io/)님에게 질문해보았다.

존그립 님이 이런 답변을 주었다.
> vim 의 자동완성 기능을 이용하면 될듯하다. 만들어두면 쓸만하겠는데요?

## vim 을 이용하자
존그립 님이 과거에 커밋메시지 관련하여 만든 [플러그인](https://github.com/johngrib/vim-git-msg-wheel)을 참고하면 도움이 될 거라고 하였다.

### 기획

내가 만드려고 기획한 플러그인의 사용 시나리오는 이렇다.
1. 커밋 메시지 입력 화면으로 이동한다.
1. 공동저자 입력을 위해 단축키를 누른다
1. 팀원 목록이 리스트로 출력된다.
1. 팀원을 선택한다.
1. 선택한 팀원정보로 공동저자 정보가 커밋메시지에 추가된다.

**팀원 리스트는 설정 파일로 관리한다.**

### 사전학습

`vimscript` 의 문법을 모르는 상태이기 때문에 [여기에서](https://devhints.io/vimscript) 도움을 많이 받았다.

#### vimscript 연습

##### 배열 다루기
이 스크립트는 `team` 이라는 변수를 배열로 초기화 하고, 배열에 값을 추가하는 코드이다.
``` vimscript
function! TestFunction()
  let team = []
  let team = add(team, '박제희')
  let team = add(team, '존그립')
  echo team
endfunction
```
![image](https://user-images.githubusercontent.com/25237661/77843363-6f73f880-71d7-11ea-83c5-f23595e8c9df.png)


##### 파일 읽기  
이 스크립트는 현재경로에 존재하는 `example` 이라는 파일을 읽어서 라인별로 `echo` 를 하는 코드이다.
``` vimscript
function! TestFunction()
  let fileAbsolutePath = expand('%:p:h')
  let records = readfile(fileAbsolutePath . '/example')
  for record in records
    echo record
  endfor
endfunction
```
![image](https://user-images.githubusercontent.com/25237661/77843575-6be17100-71d9-11ea-8fdc-88df9fe273dd.png)


##### 키맵핑 하기
이 코드는 vim 편집모드에서 ctrl + l 키를 입력할 시 TestFunction을 실행하라는 코드이다.  
기대 하는 결과는 편집모드에서 `Co-Authored-By: jehuipark <email>` 텍스트가 입력될 것이다.  
``` vimscript
function! TestFunction()
  return 'Co-Authored-By: jehuipark <email>'
endfunction

imap <C-l> <C-R>=TestFunction()<CR>
```
![image](https://user-images.githubusercontent.com/25237661/77843700-af88aa80-71da-11ea-820e-2884d8a10d8d.png)

### 개발완료
이렇게 단위별로 기능을 작성하는 연습을 끝낸 후에 본격적으로 내가 원하는 플러그인을 만들기 위해 개발을 시작하였고, 조악한 코드지만 어찌어찌 완성을 하였다!

이렇게 만든 스크립트는 [GitHub 저장소](https://github.com/JeHuiPark/github-co-author-vim-plugin)에 올려두었다.


**설정파일의 내용**
![image](https://user-images.githubusercontent.com/25237661/77854220-de773e80-7223-11ea-9608-cdbe1d7506c7.png)

자! 이제 사용을 해보자 
![github-co-plugin](https://user-images.githubusercontent.com/25237661/77853939-11203780-7222-11ea-8414-48b336fed2a8.gif)


하하하하하 잘된다.

## 참고자료
>- [기계인간 존그립 님의 포스팅1](https://github.com/johngrib/vim-git-msg-wheel)  
- [기계인간 존그립 님의 포스팅2](https://johngrib.github.io/wiki/vim-auto-completion/)  
- [박준근 님의 vim-plug](https://github.com/johngrib/vim-git-msg-wheel)  
- [vim script cheatsheet](https://devhints.io/vimscript)  
- [어떤 외국인 블로그](https://blog.semanticart.com/2017/01/05/lets-write-a-basic-vim-plugin/)
