---
layout: post
title: Jekyllを使ってGithub.ioでブログを作る
subtitle: Ruby全く知らないけど
tags: [github, jekyll, ruby]
comments: false
---

## Jekill（ジキル）
- 個人、プロジェクト又は組織のサイト向けの、簡単で、ブログのような静的サイトジェネレーターである（[wikipedia](https://ja.wikipedia.org/wiki/Jekyll)より)
- Rubyで開発されている
	- のでJekyllを利用するのに必要な「Bundler」や「Gemfile」などの単語が全く分かりません．でもやる！

## 開発環境の用意
### on Ubuntu 20.04
1. Jekyllを利用するのに必要なツールがインストールされていることを以下のコマンドで確認します
```
$ ruby -v
$ gem -v
$ gcc -v
$ g++ -v
$ make -v
```
自分の環境ではすべて揃っていました．
いずれかのコマンドの実行に失敗した場合はインストールする必要があります．


2. JekyllとBundlerをインストールします
(注意) 以下のコマンドは一般ユーザで実行してください
```
$ sudo apt-get install ruby-full build-essential zlib1g-dev
$ echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
$ echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
$ echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
gem install jekyll bundler
```

### on Windows
（注意）私の環境はGitBashやVSCodeを利用して，Linux系コマンドが実行できるターミナルがあります
#### Rubyのインストール
1. https://rubyinstaller.org/downloads/へアクセス
2. 「RubyInstallers」セクションの「WITH DEVKIT」から「⇒」マークが付いているリンクを選択します
	- 例）⇒ Ruby+Devkit 3.0.2-1 (x64)
3. ダウンロードしたファイルを管理者として実行します
	- 「インストール先とオプションの指定」と書かれた画面が表示されたら、「Rubyの実行ファイルへ環境変数PATHを設定する」を必ずチェック
4. インストールが完了するとコマンドプロンプトが起動し入力を求められるので，すべて[Enter]を押下します
5. PCを再起動します


## ブログ用リポジトリの作成
2通りの方法を紹介します．
1つは「JekyllテンプレートのGithubリポジトリを自分のリポジトリへForkする」方法です．
もう1つは「自分のリポジトリを新規作成して，そのリポジトリにJekyllテンプレートを設定する」方法です．

### JekyllテンプレートのGithubリポジトリを自分のリポジトリへForkする
1. [Free Jekyll Themes](https://jekyllthemes.io/free)などのサイトから気になるJekyllテンプレートを見つけます
2. そのテンプレートのGithubページへ移動します
	- READMEで使い方の説明があると思うので，その指示に従います
	- 以下からは，私がお借りした[Beautiful Jekyll](https://github.com/daattali/beautiful-jekyll)で推奨されているはForkを使った方法を説明します
3. 右上の[Fork]ボタンを押し，自分のリポジトリへコピーします
4. 自分のリポジトリへ移動し，[Settings]から[Repository名]を変更します
	- Repository nameとして「アカウント名.github.io」または「適当な名前」を指定します
		- 「アカウント名.github.io」を指定した場合，ブログのURLがhttps:<アカウント名>.github.io/になります
		- 「適当な名前」を指定した場合，ブログのURLがhttps:<アカウント名>.github.io/<適当な名前>になります


### 自分のリポジトリを新規作成して，そのリポジトリにJekyllテンプレートを設定する
1. 自分のGithubで新規リポジトリを作成します
	- Repository nameとして「アカウント名.github.io」または「適当な名前」を指定します
		- 「アカウント名.github.io」を指定した場合，ブログのURLがhttps:<アカウント名>.github.io/になります
		- 「適当な名前」を指定した場合，ブログのURLがhttps:<アカウント名>.github.io/<適当な名前>になります
	- 公開範囲としてPublicを選択します
	- [Create repository]ボタンを押します
2. リポジトリの[Settings]へ移動し，[Pages]の[Theme Chooser]からテーマを変更します


## ローカルでの開発
1. ブログ用リポジトリをローカル環境へクローンして，そのディレクトリに移動します
2. Gemfileという名前のファイルを作成し，以下を記述します
```
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
```
3. 次のコマンドを実行
```
$ bundle install
```
4. 次のコマンドを実行すると[http://127.0.0.1:4000](http://127.0.0.1:4000)でブログを確認できるようになります
```
$ bundle exec jekyll serve
```
