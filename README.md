#Rails -> Travis -> Heroku までの環境構築

##必要なもの
- Unix/Linux 環境(OS Xも含む)
- [rbenv](https://github.com/pyar6329/.rbenv)
- [ndenv](https://github.com/riywo/ndenv)
  - `therubyracer`ではなく、`node.js`で動作するため。
- [heroku Toolbelt](https://toolbelt.heroku.com/)
- 各種アカウント
  - [github](https://github.com/)
  - [heroku](https://www.heroku.com/)
  - Travis
    - [無償](https://travis-ci.org/)
    - [有償](https://travis-ci.com/)
  - [NewRelic](http://newrelic.com/)

##RoRのひな形の作成

`Overwrite Gemfile?`と聞かれたら`n`と入力

```sh
$ git clone git@github.com:pyarbz/RailsTemplates.git アプリ名
$ cd アプリ名
$ rm -rf README.md
$ rm -rf .git
$ bundle install --path .bundle --without production --binstubs .bin
$ bundle exec rails new . --skip-bundle --skip-test-unit
$ bundle exec rails generate rspec:install
$ bundle exec spring binstub --all
$ bundle exec guard init spring rspec livereload
```

`app/assets/javascripts/application.js`


`//= require turbolinks`を消す。

```javascript
//= require jquery
//= require jquery_ujs
//= require_tree .
```

`app/views/layouts/application.html.erb`

- 変更前

```html
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
```

- 変更後

```html
  <%= stylesheet_link_tag    'application', media: 'all' %>
  <%= javascript_include_tag 'application' %>
```

Railsの起動
```sh
$ bundle exec spring rails s
```

##Githubの設定

リポジトリの作成をする
- [https://github.com/new](https://github.com/new)

gitの設定をする

```sh
$ git init
$ git add .
$ git commit -m "first commit"
$ git remote add origin git@github.com:ユーザー名/リポジトリ名.git
$ git push origin master
```


##Herokuの設定

アプリケーションの作成をする

- [https://dashboard-next.heroku.com/new](https://dashboard-next.heroku.com/new)

view画面を作成しないとエラーが出るので、以下のコマンドでview画面を作成する。

```sh
$ bundle exec rails g controller Top index
```

ルーティング先をTopControllerにする

`config/routes.rb`

```ruby
Rails.application.routes.draw do
  # get 'top/index'
  root 'top#index'
end
```

###環境変数を読み込むよう変更する

`/config/database.yml`

```yaml
production:
  adapter: postgresql
  encoding: utf8
  port: 5432
  host: <%= ENV['DB_HOST'] %>
  database: <%= ENV['DB_DATABASE'] %>
  pool: 5
  username: <%= ENV['DB_USERNAME'] %>
  password: <%= ENV['DB_PASSWORD'] %>
  timeout: 5000
```

`/config/secrets.yml`

```yaml
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

```sh
$ git remote add heroku git@heroku.com:アプリ名.git
$ heroku login
$ heroku config
```

以下のように出てくる。5432はport番号。

`postgres://ユーザー名:パスワード@ホスト名:5432/データベース名`

以下のコマンドを入力して環境変数を設定

```sh
$ heroku config:set DB_HOST=ホスト名
$ heroku config:set DB_DATABASE=データベース名
$ heroku config:set DB_PASSWORD=パスワード
$ heroku config:set DB_USERNAME=ユーザー名
$ heroku config:set LANG=ja_JP.UTF-8
$ heroku config:set TZ=Asia/Tokyo
$ heroku config:set SECRET_KEY_BASE=secrets.ymlに書いてあった文字列
$ heroku config:set RACK_ENV=production
```

##Travisとの連携

githubから取得できるようにしておく。

 - [https://travis-ci.org/profile](https://travis-ci.org/profile)

```sh
$ heroku auth:token
```

Travis用のTokenを生成したので、travis.ymlに以下のコマンドで書き込む

```sh
$ bundle exec travis setup heroku
```

必要であれば、Travisに環境変数を設定する

```sh
$ bundle exec travis login --org
$ bundle exec travis env set DB_HOST ホスト名
$ bundle exec travis env set DB_DATABASE データベース名
$ bundle exec travis env set DB_PASSWORD パスワード
$ bundle exec travis env set DB_USERNAME ユーザー名
$ bundle exec travis env set DB_HOST ホスト名
$ bundle exec travis env list
```

`/config/database.yml`

```yaml
test:
  adapter: postgresql
  encoding: utf8
  port: 5432
  host: <%= ENV['DB_HOST'] %>
  database: <%= ENV['DB_DATABASE'] %>
  pool: 5
  username: <%= ENV['DB_USERNAME'] %>
  password: <%= ENV['DB_PASSWORD'] %>
  timeout: 5000
```

##NewRelicでHerokuアプリをスリープさせなくする

ライセンスキーを取得しておく

- [https://rpm.newrelic.com/applications/setup](https://rpm.newrelic.com/applications/setup)

```sh
$ heroku addons:add newrelic:stark
$ heroku config:set NEW_RELIC_APP_NAME=heroku上でのアプリ名
$ heroku config:set NEW_RELIC_LICENSE_KEY=NewRelicのライセンスキー
```

URLにアクセスして、Pingを飛ばすようにする

1. `newRelicのURL/アプリ名/ping_targets`にアクセス
2. `URL to monitor`にherokuの起動URLを入力

##Rubyのバージョンの指定

2.1.2の場合

`Gemfile`

```sh
ruby '2.1.2'
```

`.travis.yml`

```yaml
rvm:
- 2.1.2
```

`.ruby-version`

```sh
$ rbenv local 2.1.2
```

##その他参考URL
###Herokuのテンプレート
[https://github.com/heroku/ruby-getting-started](https://github.com/heroku/ruby-getting-started)

###Heroku Tutorial
[https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction)
