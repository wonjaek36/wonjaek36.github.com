---
layout: post
title: "Install zsh on Mac"
date: 2020-10-09 18:00 +0900
tags: [shell]
---

Mac에서 많이 사용하는 iTerm2와 zsh 사용

* iTerm2
  * Download: https://www.iterm2.com/downloads.html
  * 별다른 이유가 없다면, Stable 버전 다운로드

* Zsh
  * zsh은 zsh 뿐만 아니라 oh-my-zsh과 다수의 플러그인을 같이 사용
  * 여기서는 두 플러그인(zsh-syntax-higlighting & zsh-autosuggestions)을 설치
  * Theme는 Powerlevel10k을 적용

{% highlight shell %}
# Install zsh
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# Install zsh-autosuggestions, zsh-syntax-highlighting
$ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

# Install powerlevel10K theme
$ git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
{% endhighlight %}

  * 다 설치하고 난 뒤에 ~/.zshrc에서 두 부분을 변경

{% highlight shell %}
...
ZSH_THEME="powerlevel10k/powerlevel10k"
...

plugins=( ...  zsh-syntax-higlighting zsh-autosuggestions  ...) # separator가 띄어쓰기

...
{% endhighlight %}


* Solarized(번외)
  * 개인적으로 좋아하는 스타일
  * Solarized: https://ethanschoonover.com/solarized/

{% highlight shell %}
$ git clone https://github.com/wonjaek36/solarized
$ cd solarized/iterm2-colors-solarized
{% endhighlight %}

  * 이 파일을 iterm2 Perference -> Profile -> Color -> Import(Color Presets...)에 적용

  