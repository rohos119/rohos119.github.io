---
layout: post
title:  "[Github Blog] 블로그 생성 with Jekyll(1)"
date:   2022-02-08 23:00:53 +0900
categories: Git
---
오늘은 `github`에서 생성할수 있는 블로그를 만들어볼려고 한다.
포토폴리오를 작성하려다 보니.. 나만 블로그 없어서.....ㅠ
이제는 머리속에만 남기지 말고 글로 남겨서 언제든지 볼수 있게 하려고 한다.


~~갑자기..자이언티 꺼내먹어요가 생각난다~~. 무튼 거두절미하고 본론부터 들어가자


사실 블로그를 설치 완료해야 이글을 쓸 수 있기때문에.. 이미 완료한 상황이지만
나같이 삽질 하는 사람들을 방지하기 위해 써보자.



## 설치 과정


[1. `Github repository` 생성](#github-repository-생성)

[2. local 실행환경(`Ruby`) 설치](#local-환경에서-구동을-위해-Ruby-설치)

[3. `Jekyll` 테마 적용](#Jekyll-theme)

[4. `Jekyll` admin 페이지 적용](#Jekyll-amdin)  



## Github repository 생성

`github repository` 생성은 git을 사용하는 사람이라면 익숙할것이라고 생각한다.


하지만 아닌 사람도 있을 수 있기에 적어보기로 하장!


  1) [`username.github.io`로 프로젝트 생성하기](#username.github.io-로-프로젝트-생성)


  2) `git clone` 으로 __작업할 디렉토리__에 복사하기


  3) `git add -all` or `git add .` 으로 staging


  4) `git commit -m "commit_comment"` 로 commit


  5) `git push -u origin main` or `git push -u origin master` 로 push


위 과정을 순차적으로 따라하면 매우 쉽게 생성이 가능하다. 자 쉬운 과정은 제외하고 
repo는 헷갈릴 수 있으니 작성해본다



### username.github.io 로 프로젝트 생성


github에 로그인 후 메인화면에서 `Repositories`를 클릭하면 `new`라고 새로 생성 할 수 있는 화면이 나온다 아래 사진과 같은 세팅을적용 하면 된다.


repository 생성:
![이미지](/assets/images/Inked_create_repository.jpg)


`README`파일은 왠만하면 생성하자 나중에 귀찮아 진다..


## Local 환경에서 구동을 위해 Ruby 설치


설치이후 실제적으로 포스팅을 할때 commit 하고  github 에서 봇이 자동 deploy 할때까지 기다리고 하면.. 너무 대기소요가 많다 해서 로컬로 확인하고 git 으로 push하는게 훨씬 수월하다.


  1) [`Ruby` 설치하기](#ruby-설치하기) 


  2) [`gem install jekyll bundler`](#jekyll-과-bundler-설치)


  3) [`jekyll new ./` 로 새로운 `jekyll` 프로젝트 생성](#jekyll-프로젝트-생성)


  4) [`bundle install` 로](#bundle-install-하기)


  5) [`bundle exec jekyll serve` 로 구동시키기](#로컬에서-구동시키기)


### Ruby 설치하기


[https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)


위에 설치 링크를 들어가서 빨간색 선택 :
![이미지](/assets/images/install_ruby_for_window.jpg)


클릭하고 전부 next 하고 설치가 잘되었다면
아래명령어가 정상적으로 실행되면 된다.

{% highlight ruby %}
ruby -v
{% endhighlight %}


실행결과 :
![이미지](/assets/images/ruby_version_check.PNG)


### Jekyll 과 bundler 설치

{% highlight ruby %}
gem install jekyll bundler
{% endhighlight %}


위 명령어를 실행하면 jekyll과 bundler를 설치할 수 있다.


`jekyll`은 텍스트 변환 엔진으로, 마크업 언어로 글을 작성하면 변환에서 웹사이트를 만들어준다.


`bundler`는 `Ruby`프로젝트의 버전관리와 `gemfile`을 생성해준다.


자세한 설명은 다음포스트를 올릴 에정!


### Jekyll 프로젝트 생성

{% highlight ruby %}
jekyll new ./"프로젝트네임"
{% endhighlight %}

위 명령어를 치면 새로운 `jekyll`프로젝트가 생성된다.

생성결과 :
![이미지](/assets/images/newjekyll.PNG)

생성결과처럼 파일이 생성되면 완료!


### bundle install 하기

{% highlight ruby %}
bundle install
{% endhighlight %}

`bundle install`을 실행시키면 `Gemfile` 에 기술된
의존성에 의해 필요한 모든 `gem`을 설치한다.

### <span stlye="color:red;">bundle install 설치시 error</span>

bundle 설치시 아래와 같은 에러가 발생하면 


    C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)


{% highlight ruby %}
bundle add webrick 
bundle install
{% endhighlight %}

이렇게 하면 해결할 수 있습니다!!     


### 로컬에서 구동시키기


{% highlight ruby %}
bundle exec jekyll serve 
{% endhighlight %}


위 명령어를 사용하면 jekyll이 구동된다.

실행결과 :
![이미지](/assets/images/jekyll_serve.PNG)


<span style="color:red;">빨간밑줄</span> 친 로컬 서버로 들어가면 아래와 같은 화면이 나온다!

실행결과:
![이미지](/assets/images/jekyll_serve_complete.JPG)


이제 반왔다 나머지 는 다음글에서!

다음글
[[Github Blog] 블로그 생성 with Jekyll(2)]()
