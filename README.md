## バージョン
- Ruby version
    - 2.7.0
- Ruby on Rails version
    - 6.0.2.2
- Mysql
    - 8.0

## 環境構築
```
・ 新規作成する場合はsrcディレクトリを空にした状態で「プロジェクト作成」から開始  
※ Gemfile（src外にある内容を写した状態）とGemfile.lock（空）は残す
・ 既存のソースを利用する場合は「コンテナをビルド&起動」から開始
```
- プロジェクト作成   
通常モードの場合
`docker-compose run web rails new . --force --database=mysql --skip-bundle`  
APIモードの場合
`docker-compose run web rails new . --api --force --database=mysql --skip-bundle`  
※ イメージ作成とホスト側にrailsの基盤を作ることがメインなので、コンテナがダウンしてても問題なし
- database.yml を修正  
```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root <-変更
  password: password <-変更
  host: db <-変更
```
- コンテナをビルド&起動  
`docker-compose -f docker-compose.yml build`  
`docker-compose -f docker-compose.yml up -d`  
※ 恐らくwebコンテナが落ちるので下記を実行
- Rails6からwebpackerが必要になったのでインストール  
`docker-compose run web rails webpacker:install`  
※ runで起動したコンテナは不要なので docker rm で削除しておく
- 再度、コンテナを起動  
`docker-compose -f docker-compose.yml up -d`
- MySQL V8で認証プラグイン「caching_sha2_password」をロードできないため下記で回避  
`docker-compose exec db bash`  
`mysql -uroot -ppassward`  
`ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'password';`  
`exit`
- DBを作成  
`docker-compose exec web rails db:create`   
- docker ps でコンテナが正常に起動していることを確認したら下記にアクセス  
http://localhost:3000/

※ gem系のエラーが発生してコンテナが起動しなかった場合  
- ログを確認  
`docker logs コンテナID`
- コンテナ・ネットワーク・ボリュームを停止・削除  
`docker-compose down -v`
- bundle install を実行  
`docker-compose run web bundle install`
- コンテナをビルド&再起動  
`docker-compose -f docker-compose.yml build`  
`docker-compose -f docker-compose.yml up -d`

## 基本操作まとめ
※ コンテナの中で実行する想定  
※ コンテナ外であれば `docker-compose exec web` を接頭に付ける
- モデル作成  
`rails g model モデル名（単数形） カラム名:型 カラム名:型`
- コントローラー作成  
`rails g controller コントローラー名（複数形）`
- マイグレーション  
`rake db:migrate`
- ルーティング設定  
```
[config/routes.rb]
※【controllers/階層_1/階層_2/コントローラー名.rb】の場合
Rails.application.routes.draw do
  namespace '階層_1' do
    namespace '階層_2' do
      resources :ルーティング名(コントローラー名に合わせる)
    end
  end
end
```
- ルーティング設定確認  
`rake routes`
- railsコンソール操作  
`rails c`  
```
例）データ挿入
モデル名.create(カラム名:'データ')
```
- scaffoldを使用する場合  
`rails g scaffold モデル名 カラム名1:データ型1 カラム名2:データ型 2`  
`rake db:migrate`
