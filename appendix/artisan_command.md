# artisanコマンド集

## キャッシュクリア
```
php artisan cache:clear
```
## APP KEY 自動生成
```
php artisan key:generate
```
## サーバ起動
```
php artisan serve --host=$IP --port=$PORT
```

## ルーティング一覧表示
```
php artisan route:list
```

## REPL 使用
```
php artisan tinker
```

## マイグレーション
```
php artisan migrate
```

## マイグレーションのリフレッシュ
```
php artisan migrate:fresh --seed
```

## マイグレーションのロールバック
```
php artisan migrate:rollback
```

## model 作成
```
php artisan make:model -m Message
```

## controller 作成
```
php artisan make:controller MessagesController --resource --model=Message
```

