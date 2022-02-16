---
layout: post
title:  "[Github Blog] 블로그 생성 with Jekyll(2)"
date:   2022-02-10 22:56:53 +0900
categories: Git
---

자 이번에는 지난 포스트에 이어서  `Jekyll theme`e와 `admin page`를 생성해서 나중에 쉽게 작업할 수 있게 해보자!!


## 설치과정
---
  ~~1. `Github repository` 생성~~

  ~~2. local 실행환경(`Ruby`) 설치~~
	  
  [3. `Jekyll` 테마 적용](#Jekyll-테마-적용)
  
  [4. `Jekyll` admin 페이지 적용](#Jekyll-amdin-페이지-적용)
  
## Jekyll 테마 적용
---
jekyll 테마를 적용하는 방법은 크게 두가지로 나뉜다.
  1. [원하는 theme를 jekyll 프로젝트에 Copy & Paste](#원하는-theme를-jekyll-프로젝트에-Copy&Paste)
  2. [Gem을 통한 설치](#Gem을-통한-설치)
 
위 두가지 방법 중... 솔직히 1번을 강력추천한다.
2번을 쓰는 것은 내가 완전히 Jekyll에 대한 구조를 이해해서 Customizing이 자신 있다 라고 생각되면 하길 바란다.

하지만 두가지 방법다 포스팅 하려고 한다!

### 원하는 theme를 jekyll 프로젝트에 Copy&Paste
---
먼저 무료로 배보중인 theme 사이트 링크에 접속 한다
   * Lanyon 테마 - [https://jekyll-themes.com/lanyon/](https://jekyll-themes.com/lanyon/)
   * Jekyll 무료 테마 - [https://jekyll-themes.com/free/](https://jekyll-themes.com/free/)
   * 쓴이의 테마 -[https://jekyll-themes.com/jekyll-theme-yat/](https://jekyll-themes.com/jekyll-theme-yat/)
   
솔직히.. ~~저는 테마 잘못고른거 같아서 후회...~~ 본 포스팅에선 Lanyon테마로 진행합니다

```cmd
git clone https://github.com/poole/lanyon
```

  위 명령어로 theme를 다운 받고 생성된 `lanyon`파일에 있는 내용을 전부 새로 생성한 jekyll 프로젝트에 `Gemfile`, `Gemfile.lock`을 제외한 **프로젝트에 내용물을 전부 삭제** 후 복사 붙여넣기 합니다

```ruby
  bundle install
  bundle exec jekyll serve
```

실행결과 :
![이미지](/assets/images/lanyon_complete.jpg)

이렇게하면 끝!!!!!!!! 야 너두? 야 나두!

#### <span stlye="color:red;">bundle install error</span>
---

```
C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/external.rb:60:in `require': cannot load such file -- jekyll-paginate (LoadError)
```

위와 같은 오류가 발생했을 때에는 bundle에 해당 package가 설치가 안된것임으로

```ruby
bundle add jekyll-paginate
bundle install
bundle exec jekyll serve
```
위와 같이 실행하면 정상 작동 한다!!

#### Gem을 통한 설치
---

jekyll 프로젝트내에서 해당명령어를 실행한다
```ruby
gem install jekyll-theme-lanyon 
```


설치가 완료되면 `Gemfile`에 수정을 해줘야한다

```
source "https://rubygems.org"

gem "jekyll", "~> 3.7.0"
gem "jekyll-theme-lanyon", "~> 1.1"
```

수정을 완료했으면 동일하게

```ruby
bundle install
bundle exec jekyl server 
```

실행하면 동일한 화면이 나온다!!


## Lanyon 구조도
---
먼저 `jekyll`의  매커니즘을 알아보자
<ul>
<li>post마다 md 파일이나 html 파일을 생성한다.</li>
<li>글을 작성하고 배포하기 위한 빌드를 jekyll에서 진행하면 응답할 html 화면을 만들고 파일로 저장해서 준비한다.</li>
<li>유저가 특정 화면을 요청하면 미리 생성한 html 파일을 찾아 꺼내준다.</li>
<li><strong>DB를 조회하고 HTML 양식으로 응답하는 과정과 같다.

</strong><h4>모든 화면을 미리 만든다.</h4>
</li>
<li>유저가 요청할 수 있는 모든 화면을 미리 빌드해 두는 방식을 상용</li>
<li>따라서 글이 많을 수록 길어지는 글 목록 화면이 많아지고 미리 만들어야하는 페이지 수도 많아진다.


<h4>검색 기능</h4>
</li>
<li>클라이언트 스크립트를 사용해서 작성된 모든 글의 제목과 내용에 키워드를 조회한다.</li>
<li>최상위 경로에 검색에 필요한 정보인 search.json을 생성하고 자바스크립트를 이용해 검색한다.</li>
</ul>

<pre class="code ruby"><code class="ruby"><span class="id identifier rubyid_├──">├──</span> <span class="id identifier rubyid__posts">_posts</span>        <span class="comment"># 포스트 할때 markdown 파일 올리는곳!
</span><span class="id identifier rubyid_├──">├──</span> <span class="id identifier rubyid__site">_site</span>         <span class="comment"># build 되고 나서 생성되는 파일들!
</span><span class="id identifier rubyid_├──">├──</span> <span class="id identifier rubyid__config">_config</span><span class="period">.</span><span class="id identifier rubyid_yml">yml</span>   <span class="comment"># 기본 설정값들 바꾸는 곳
</span><span class="id identifier rubyid_├──">├──</span> <span class="int">404</span><span class="period">.</span><span class="id identifier rubyid_html">html</span>      <span class="comment"># default 404 page template
</span><span class="id identifier rubyid_├──">├──</span> <span class="id identifier rubyid_about">about</span><span class="period">.</span><span class="id identifier rubyid_md">md</span>      <span class="comment"># default example page
</span><span class="id identifier rubyid_├──">├──</span> <span class="const">Gemfile</span>       <span class="comment"># bundler configuration
</span><span class="id identifier rubyid_├──">├──</span> <span class="const">Gemfile</span><span class="period">.</span><span class="id identifier rubyid_lock">lock</span>  <span class="comment"># bundler version lock
</span><span class="id identifier rubyid_└──">└──</span> <span class="id identifier rubyid_index">index</span><span class="period">.</span><span class="id identifier rubyid_html">html</span>    <span class="comment"># list all the posts on the homepage
</span></code></pre>



## Jekyll admin 페이지 적용
---
결국 기본 template는 수정소요가 없고
post에서 게시되는 `.md` 파일들을 잘 수정하면 된다.


근데 이게 생각보다 쓰고 하는게 귀찮다.. 해서
찾은 방법이 `jekyll-admin` package를 다운 받아서 수정하고 유지하는 것이다


방법은 `Gemfile`에 아래를 추가한다!!


```
gem 'jekyll-admin', group: :jekyll_plugins
gem 'jekyll-sitemap'
```


추가후에 다시 아래 루비 명령어를 실행한다.
```ruby
bundle install 
bundle exec jekyll serve
```

실행하고 `http://127.0.0.1:4000/admin`을 접속 하면 아래와 같은 화면이 나올꺼다!

실행결과 :
![이미지](/assets/images/lanyon_admin_complete.JPG)
