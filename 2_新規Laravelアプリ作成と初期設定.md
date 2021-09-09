# 2_新規Laravelアプリ作成と初期設定

<p style='text-align: right;'> &copy; 20210907 by Takanori Shima </p>

```
* 以下、Cloud9上でターミナルを起動してコマンドを打つ *
最終的には3つのターミナルを同時に開いておく
- MySQL用(占有)
- Laravelサーバ起動用(占有)
- artisanコマンドを随時実行用
```

## 1. 新規データべース作成 
```
sudo service mysqld start
mysql -u root
mysql> create database bbs_laravel default character set utf8;
mysql> use bbs_laravel;
```
このターミナルはこのままMySQL専用にする
## 2. 新規Laravelプロジェクト作成(bbs_laravel)
もう一つターミナルを起動
```
composer create-project --prefer-dist laravel/laravel bbs_laravel "5.8.*"
cd bbs_laravel
```
## 3. 初期環境設定確認
```
php artisan tinker
>>> print_r($_ENV);
Array
(
    [APP_NAME] => Laravel
    [APP_ENV] => local
    [APP_KEY] => base64:EINdb2HY5b5kBvFISO2JYZc92P5oF7wGDCEt/bT5G3I=
    [APP_DEBUG] => true
    [APP_URL] => http://localhost
    [LOG_CHANNEL] => stack
    [DB_CONNECTION] => mysql
    [DB_HOST] => 127.0.0.1
    [DB_PORT] => 3306
    [DB_DATABASE] => laravel
    [DB_USERNAME] => root
    [DB_PASSWORD] => 
    [BROADCAST_DRIVER] => log
    [CACHE_DRIVER] => file
    [QUEUE_CONNECTION] => sync
    [SESSION_DRIVER] => file
    [SESSION_LIFETIME] => 120
    [REDIS_HOST] => 127.0.0.1
    [REDIS_PASSWORD] => null
    [REDIS_PORT] => 6379
    [MAIL_DRIVER] => smtp
    [MAIL_HOST] => smtp.mailtrap.io
    [MAIL_PORT] => 2525
    [MAIL_USERNAME] => null
    [MAIL_PASSWORD] => null
    [MAIL_ENCRYPTION] => null
    [AWS_ACCESS_KEY_ID] => 
    [AWS_SECRET_ACCESS_KEY] => 
    [AWS_DEFAULT_REGION] => us-east-1
    [AWS_BUCKET] => 
    [PUSHER_APP_ID] => 
    [PUSHER_APP_KEY] => 
    [PUSHER_APP_SECRET] => 
    [PUSHER_APP_CLUSTER] => mt1
    [MIX_PUSHER_APP_KEY] => 
    [MIX_PUSHER_APP_CLUSTER] => mt1
    [SHELL_VERBOSITY] => 0
)
=> true
>>> exit
```
## 4. プロキシ設定 
### app/Http/Middleware/TrustProxies.php 15行目附近を変更
Cloud9の環境ではAWSのロードバランサが使用されているため、LaravelがURLをうまく生成できない問題があるための対策
```
protected $proxies = '**'; // 全プロキシを信用
```

## 5. Laravel サーバの起動とプレビュー
```
php artisan serve --host=$IP --port=$PORT
```
以後、このターミナルはLaravelサーバ起動専用とする

## 6. 使用するデータベースの設定
###  config/database.php の18行目附近に以下の行があることを確認
```
'default' => env('DB_CONNECTION', 'mysql'),
```
### .env 9-14行目附近を書き換え
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=bbs_laravel
DB_USERNAME=root
DB_PASSWORD=
```

### LaravelがMySQLのデータベースと接続に成功しているか確認
bbs_laravelフォルダでもう一つターミナルを起動
```
php artisan tinker
>>> DB::reconnect()
=> Illuminate\Database\MySqlConnection {#3234}
>>> exit
```

## 7. Laravelアプリのタイムゾーン設定
### config/app.php の70行目附近を変更
```
    'timezone' => 'Asia/Tokyo',
```

## 8. Git/Github
初回だけ以下を実行
```
git config --global user.name "TakanorShima"
git config --global user.email "quark2galaxy@gmail.com"
git init
```
続けて以下の3行を実行(毎回の繰り返し単位)
```
git add .
git commit -m "新規Laravelアプリの作成と環境設定完了"
git log
```
Githubに bbs_laravel という名前のリモートリポジトリを作成後
```
git remote add origin https://github.com/TakanoriShima/bbs_laravel.git
git branch -M main
git push -u origin main
```
