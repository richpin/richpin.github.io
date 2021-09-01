---
title: "jekyll serve 명령 실행 시 crash-abort 이슈"
excerpt: "jekyll serve 명령 실행이 bug report에 대한 내용과 함께 abort 될 때"

last_modified_at: 20201-09-01
---

이제는 나도 개발 블로그를 하나 해야겠다는 생각으로 Github Pages로 지금 이 블로그를 만들기 시작하던 중이었다.  

###jekyll 테마를 다운받아 로컬 호스팅 해보기

많은 개발자들이 그러하듯 `jekyll` 테마를 이용하기 위해 `homebrew`로 버전 관리자인 `rbenv` 먼저 설치해주고  
```
$ brew install rbenv
```
rbenv로 가능한 `Ruby`버전들을 확인해본다.
```
$ rbenv install --list

2.6.8
2.7.4
3.0.2
jruby-9.2.19.0
mruby-3.0.0
rbx-5.0
truffleruby-21.2.0.1
truffleruby+graalvm-21.2.0

Only latest stable releases for each Ruby implementation are shown.
Use 'rbenv install --list-all / -L' to show all local versions.
```
3.0.2가 가장 최근이니 거침없이 다운했다.
```
$ rbenv install 3.0.2
```
Ruby 패키지 관리 프레임워크인 `RubyGems`는 [공식 홈페이지](https://rubygems.org/pages/download/)에서 받고, 그 gem을 이용해서 사용하려는 `jekyll`과 `bunlder`를 설치한다.
```
$ gem install jekyll bundler
```
원하는 jekyll 테마를 다운받아야 했는데 나는 그냥 가장 무난하고 사용이 많은 [minimal mistakes theme](https://github.com/mmistakes/minimal-mistakes)을 받았다. 다운받은 테마 디렉토리로 이동 후 bundle을 실행해준다. 정말 컴퓨터는 한 가지 하려면 너무 여러 가지를 시킨다. ~~(명령어 한 번에 다 쭈루룩 되면 안되나...)~~
```
$ bundle
```
이제 로컬 호스팅을 하기 위해 `serve`명령어 실행!!! 
```
$ bundle exec jekyll serve
```
잘되나 싶었지만......  

```
497 /Library/Ruby/Gems/2.6.0/gems/sassc-2.4.0/lib/sassc/native/sass_input_style.rb
  498 /Library/Ruby/Gems/2.6.0/gems/sassc-2.4.0/lib/sassc/native/sass_output_style.rb
  499 /Library/Ruby/Gems/2.6.0/gems/sassc-2.4.0/lib/sassc/native/string_list.rb

[NOTE]
You may have encountered a bug in the Ruby interpreter or extension libraries.
Bug reports are welcome.
For details: https://www.ruby-lang.org/bugreport.html

[IMPORTANT]
Don't forget to include the Crash Report log file under
DiagnosticReports directory in bug reports.

zsh: abort      bundle exec jekyll serve
```
어째 한 번에 착!하고 잘 될리가 없지...:flushed:
Crash Report하더니 `abort`라.., 명령어가 씹혔다는 얘기다...
해결방법을 찾아보려 열심히 구글링했다.

###Crash Report 후 abort 문제 해결해보기

이것저것 해결해보았지만...방법이 보이지가 않았다.
언제나 그랬듯이 열심히 나오는 명령이들 다 때려 넣어보았지만 언제나 그랬듯이 바로 해결되지 않는다.:sweat_smile:
이 문제를 다루고 있는 가장 최근 커뮤니티를 살피던 중 `Apple m1`의 이슈라는 내용들이 있었다. (정말 여러모로 아직은 m1으로 살아가기 걸림돌이 많은 것 같다...) 

어떤 m1 유저분은 이제 업데이트 `v3`이후로는 해결됬기 때문에 이것저것 업데이트 업데이트를 해보면 된다했다. 물론 해봤는데 안되서 **gem으로 현재 상태를 확인해보고 있던 중이었다.**
```
$ gem env

RubyGems Environment:
  - RUBYGEMS VERSION: 3.2.26
  - RUBY VERSION: 2.6.3 (2019-04-16 patchlevel 62) [universal.arm64e-darwin20]
  - INSTALLATION DIRECTORY: /Library/Ruby/Gems/2.6.0
```
뭔가 이상한데? 직관적으로 RubyGems와 Ruby의 버전 차이가 심함을 알았고 자세히 보니 2019년 버전??? 그러고 보니 아까 난 3.0.2버전을 다운받았는데???

```
$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.arm64e-darwin20]
```
정말 2.6.3이라는 뚱딴지 같은 버전으로 설정이 되어있었다.:rage:
바로 rbenv를 통해 3.0.2로 버전 스위치에 들어갔다.
```
$ rbenv shell 3.0.2

rbenv: shell integration not enabled. Run `rbenv init' for instructions.
```
음...당황하지 않고, 써있는 대로 명령어를 실행해보았다.
```
$ rbenv init

# Load rbenv automatically by appending
# the following to ~/.zshrc:

eval "$(rbenv init -)"
```
후...다시 한 번 당황하지 않고, 써있는 대로 실행
````
$ eval "$(rbenv init -)"
````
그리고 다시 `$ rbenv shell 3.0.2`을 실행하니 드디어 되었다. 그리고 빠르게 디폴트 버전으로 설정해주었다. (나중에야 알았지만 이미 위 명령어로 Ruby 버전이 바뀌어있었다.)
```
$ rbenv global 3.0.2
```
다시 `$ bundle exec jekyll serve` 실행해보니 겁나 잘된다!!! 적어도 내가 찾아봤을 때는 이런 경우에 관해 자세히 쓴 블로그는 못 봐서 이렇게 글을 쓰게 되었다. 따라서 혹시 이런 문제가 있다면 
##결론 : gem env로 버전 확인 먼저 해보세요
.
.
.
.
.
###또 다른 이슈
하...끝내고 싶었는데 또 후속 문제가 찾아왔다. 터미널을 끄고 다시 실행하면 또 Ruby 버전이 2.6.3이 되어있다.
```
$ rbenv global

3.0.2
```
`global` 버전은 문제 없는데????

PATH가 잘못 들여져하고 확인해봤더니
```
$ env | grep PATH

PATH=/Users/yooyoungseok/.gem/ruby/2.6.0/bin:/Users/yooyoungseok/.gem/ruby/2.6.0/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin
MANPATH=/opt/homebrew/share/man::
INFOPATH=/opt/homebrew/share/info:
```
2.6.0??? 도대체 이게 왜 환경 변수에 왜 있는 거지?

그 이후로도 이것저것 다 해보다가 결국 초심으로 돌아가서 Ruby 버전 관리에 대한 문서로 돌아갔다. 그러다 보니 하나 걸리는 게  
`Pre-installed macOS system Ruby`  
MAC OS에서는 기본적으로 시스템에 Ruby가 있다고 한다...
버전은 2.6.3이란다. 이마를 탁! 쳤다.
```
$ which ruby

/usr/bin/ruby
```
이렇게 보인다면 이미 깔려있는 거...문서에서는 그냥 무시하고 다른 버전 관리 시스템(asdf, chruby, rbenv, or rvm)들을 이용해서 사용하면 된다는 데 이것이 환경 변수에 영향을 끼쳐서 터미널을 새로 만들 시 계속해서 영향을 주고 있는 가보다.MAC 사용자인 이상 안고갈 수 밖에 없는 가 보다...:sob:

그래서 나같은 경우는 그나마 편하게 `~/.bash_profile`안에
```
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```
적어놓고 serve 해야될 시에
```
source ~/.bash_profile
````
해주기로 했다.