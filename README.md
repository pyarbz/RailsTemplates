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
$ bundle install --path .bundle --without test production
$ bundle exec rails new . --skip-bundle --skip-test-unit
$ bundle exec rails generate rspec:install
$ bundle exec spring s
```

##Herokuの設定

view画面を作成しないとエラーが出るので、以下のコマンドでview画面を作成する。

```sh
$ bundle exec rails g controller Top index
```

ルーティング先をTopControllerにする

`config/routes.rb`

```ruby
Rails.application.routes.draw do
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

```sh
$ heroku auth:token
```

Travis用のTokenを生成したので、travis.ymlに以下のコマンドで書き込む

```sh
$ bundle exec travis setup heroku
```

##NewRelicでHerokuアプリをスリープさせなくする

```sh
$ heroku addons:add newrelic:stark
$ heroku config:set NEW_RELIC_APP_NAME=heroku上でのアプリ名
$ heroku config:set NEW_RELIC_LICENSE_KEY=NewRelicのToken
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
