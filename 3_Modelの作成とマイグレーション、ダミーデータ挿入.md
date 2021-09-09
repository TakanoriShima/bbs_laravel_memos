# 3_Modelの作成とマイグレーション、ダミーデータ挿入

<p style='text-align: right;'> &copy; 20210907 by Takanori Shima </p>

```
* 以下、Cloud9上でターミナルを起動してコマンドを打つ *
最終的には3つのターミナルを同時に開いておく
- MySQL用(占有)
- Laravelサーバ起動用(占有)
- artisanコマンドを随時実行用
```

## 1. MySQL専用の設定
### app/Providers/AppServiceProvider.php 24行目附近を変更
```
    public function boot()
    {
        \Schema::defaultStringLength(191);
        \URL::forceScheme('https');
    }
```

## 2. Messageモデルの作成
-mはデータベースへのマイグレーションファイルを同時に作成するオプション
```
php artisan make:model -m Message
```
>Model created successfully.<br>
>Created Migration: 2021_09_01_160022_create_messages_table

## 3. マイグレーションファイル修正
### database/migrations/xxxx_create_messages_table.php 14行目附近
```
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('title');
            $table->text('body');
            $table->text('image');
            $table->timestamps();
        });
    }
```

## 4. マイグレーション実行
```
php artisan migrate
```
>Migration table created successfully.<br>
>Migrating: 2014_10_12_000000_create_users_table<br>
>Migrated:  2014_10_12_000000_create_users_table (0.03 seconds)<br>
>Migrating: 2014_10_12_100000_create_password_resets_table<br>
>Migrated:  2014_10_12_100000_create_password_resets_table (0.02 seconds)<br>
>Migrating: 2021_09_01_160022_create_messages_table<br>
>Migrated:  2021_09_01_160022_create_messages_table (0.01 seconds)

## 5. MySQLにテーブルが作成されたか確認
```
mysql> use bbs_laravel
mysql> show tables;
+-----------------------+
| Tables_in_bbs_laravel |
+-----------------------+
| messages              |
| migrations            |
| password_resets       |
| users                 |
+-----------------------+
mysql> desc messages;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| name       | varchar(191)        | NO   |     | NULL    |                |
| title      | varchar(191)        | NO   |     | NULL    |                |
| body       | text                | NO   |     | NULL    |                |
| image      | text                | NO   |     | NULL    |                |
| created_at | timestamp           | YES  |     | NULL    |                |
| updated_at | timestamp           | YES  |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
```
## 6. Laravelとの対話モードで、モデルを使ってCRUD処理
### tinker 起動
```
php artisan tinker
```
### テーブルに新規データ挿入(INSERT系)
- 実行コマンド
```
use App\Message
$message = new Message()
$message->name = 'shima'
$message->title = 'hello!'
$message->body = 'enjoy Laravel'
$message->image = '1.jpg'
$message
$message->save()

$message = new Message()
$message->name = 'yamada'
$message->title = 'good morning!'
$message->body = 'enjoy PHP'
$message->image = '2.jpg'
$message
$message->save()
```

- 実行結果例
```
>>> use App\Message
>>> $message = new Message()
=> App\Message {#3239}
>>> $message->name = 'shima'
=> "shima"
>>> $message->title = 'hello!'
=> "hello!"
>>> $message->body = 'enjoy Laravel'
=> "enjoy Laravel"
>>> $message->image = '1.jpg'
=> "1.jpg"
>>> $message
=> App\Message {#3239
     name: "shima",
     title: "hello!",
     body: "enjoy Laravel",
     image: "1.jpg",
   }
>>> $message->save()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> true
>>> $message = new Message()
=> App\Message {#3241}
>>> $message->name = 'yamada'
=> "yamada"
>>> $message->title = 'good morning!'
=> "good morning!"
>>> $message->body = 'enjoy PHP'
=> "enjoy PHP"
>>> $message->image = '2.jpg'
=> "2.jpg"
>>> $message
=> App\Message {#3241
     name: "yamada",
     title: "good morning!",
     body: "enjoy PHP",
     image: "2.jpg",
   }
>>> $message->save()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> true
```

### テーブルデータを表示(SELECT系)
- 実行コマンド
```
Message::all()
Message::first()
Message::find(2)
Message::where('name', 'shima')->get()
Message::where('image', '2.jpg')->get()
Message::where('id', '>=', '2')->get()
Message::where('id', '>=', '2')->where('name', 'yamada')->get()

```
- 実行結果例
```
>>> Message::all()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#4174
     all: [
       App\Message {#3242
         id: 1,
         name: "shima",
         title: "hello!",
         body: "enjoy Laravel",
         image: "1.jpg",
         created_at: "2021-09-01 16:05:48",
         updated_at: "2021-09-01 16:05:48",
       },
       App\Message {#4172
         id: 2,
         name: "yamada",
         title: "good morning!",
         body: "enjoy PHP",
         image: "2.jpg",
         created_at: "2021-09-01 16:06:53",
         updated_at: "2021-09-01 16:06:53",
       },
     ],
   }
>>> Message::first()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> App\Message {#4176
     id: 1,
     name: "shima",
     title: "hello!",
     body: "enjoy Laravel",
     image: "1.jpg",
     created_at: "2021-09-01 16:05:48",
     updated_at: "2021-09-01 16:05:48",
   }
>>> Message::find(2)
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> App\Message {#4167
     id: 2,
     name: "yamada",
     title: "good morning!",
     body: "enjoy PHP",
     image: "2.jpg",
     created_at: "2021-09-01 16:06:53",
     updated_at: "2021-09-01 16:06:53",
   }
>>> Message::where('name', 'shima')->get()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#4170
     all: [
       App\Message {#4179
         id: 1,
         name: "shima",
         title: "hello!",
         body: "enjoy Laravel",
         image: "1.jpg",
         created_at: "2021-09-01 16:05:48",
         updated_at: "2021-09-01 16:05:48",
       },
     ],
   }
>>> Message::where('image', '2.jpg')->get()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#4172
     all: [
       App\Message {#4167
         id: 2,
         name: "yamada",
         title: "good morning!",
         body: "enjoy PHP",
         image: "2.jpg",
         created_at: "2021-09-01 16:06:53",
         updated_at: "2021-09-01 16:06:53",
       },
     ],
   }
>>> Message::where('id', '>=', '2')->get()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#4114
     all: [
       App\Message {#4168
         id: 2,
         name: "yamada",
         title: "good morning!",
         body: "enjoy PHP",
         image: "2.jpg",
         created_at: "2021-09-01 16:06:53",
         updated_at: "2021-09-01 16:06:53",
       },
     ],
   }
>>> Message::where('id', '>=', '2')->where('name', 'yamada')->get()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#4186
     all: [
       App\Message {#4188
         id: 2,
         name: "yamada",
         title: "good morning!",
         body: "enjoy PHP",
         image: "2.jpg",
         created_at: "2021-09-01 16:06:53",
         updated_at: "2021-09-01 16:06:53",
       },
     ],
   }
>>> 
```
### MySQLでテーブルデータを確認
```
mysql> select * from messages;
+----+--------+---------------+---------------+-------+---------------------+---------------------+
| id | name   | title         | body          | image | created_at          | updated_at          |
+----+--------+---------------+---------------+-------+---------------------+---------------------+
|  1 | shima  | hello!        | enjoy Laravel | 1.jpg | 2021-09-01 16:05:48 | 2021-09-01 16:05:48 |
|  2 | yamada | good morning! | enjoy PHP     | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
+----+--------+---------------+---------------+-------+---------------------+---------------------+
2 rows in set (0.00 sec)
mysql> select * from messages where id=2;
+----+--------+---------------+-----------+-------+---------------------+---------------------+
| id | name   | title         | body      | image | created_at          | updated_at          |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
|  2 | yamada | good morning! | enjoy PHP | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
mysql> select * from messages where name='shima';
+----+-------+--------+---------------+-------+---------------------+---------------------+
| id | name  | title  | body          | image | created_at          | updated_at          |
+----+-------+--------+---------------+-------+---------------------+---------------------+
|  1 | shima | hello! | enjoy Laravel | 1.jpg | 2021-09-01 16:05:48 | 2021-09-01 16:05:48 |
+----+-------+--------+---------------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
mysql> select * from messages where image='2.jpg';
+----+--------+---------------+-----------+-------+---------------------+---------------------+
| id | name   | title         | body      | image | created_at          | updated_at          |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
|  2 | yamada | good morning! | enjoy PHP | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
mysql> select * from messages where id >= 2;                                                                                                                                                                  
+----+--------+---------------+-----------+-------+---------------------+---------------------+
| id | name   | title         | body      | image | created_at          | updated_at          |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
|  2 | yamada | good morning! | enjoy PHP | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
mysql> select * from messages where id >= 2 AND name='yamada';
+----+--------+---------------+-----------+-------+---------------------+---------------------+
| id | name   | title         | body      | image | created_at          | updated_at          |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
|  2 | yamada | good morning! | enjoy PHP | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
+----+--------+---------------+-----------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
```

### テーブル更新(UPDATE系)

- 実行コマンド
```
$message = new Message()
$message->name = 'watanabe'
$message->title = 'sleepy'
$message->body = 'enjoy Java'
$message->image = '3.jpg'
$message
$message->save()

$message->name = 'kubota'
$message->save()
Message::all()
```

- 実行結果例
```
>>> $message = new Message()
=> App\Message {#4173}
>>> $message->name = 'watanabe'
=> "watanabe"
>>> $message->title = 'sleepy'
=> "sleepy"
>>> $message->body = 'enjoy Java'
=> "enjoy Java"
>>> $message->image = '3.jpg'
=> "3.jpg"
>>> $message
=> App\Message {#4173
     name: "watanabe",
     title: "sleepy",
     body: "enjoy Java",
     image: "3.jpg",
   }
>>> $message->save()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> true
>>> $message->name = 'kubota'
=> "kubota"
>>> $message->save()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> true
>>> Message::all()
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> Illuminate\Database\Eloquent\Collection {#3244
     all: [
       App\Message {#3241
         id: 1,
         name: "shima",
         title: "hello!",
         body: "enjoy Laravel",
         image: "1.jpg",
         created_at: "2021-09-01 16:05:48",
         updated_at: "2021-09-01 16:05:48",
       },
       App\Message {#142
         id: 2,
         name: "yamada",
         title: "good morning!",
         body: "enjoy PHP",
         image: "2.jpg",
         created_at: "2021-09-01 16:06:53",
         updated_at: "2021-09-01 16:06:53",
       },
       App\Message {#4189
         id: 3,
         name: "kubota",
         title: "sleepy",
         body: "enjoy Java",
         image: "3.jpg",
         created_at: "2021-09-01 19:16:37",
         updated_at: "2021-09-01 19:17:34",
       },
     ],
   }
>>>
```

#### MySQLでデータ確認
```
mysql> select * from messages;
+----+--------+---------------+---------------+-------+---------------------+---------------------+
| id | name   | title         | body          | image | created_at          | updated_at          |
+----+--------+---------------+---------------+-------+---------------------+---------------------+
|  1 | shima  | hello!        | enjoy Laravel | 1.jpg | 2021-09-01 16:05:48 | 2021-09-01 16:05:48 |
|  2 | yamada | good morning! | enjoy PHP     | 2.jpg | 2021-09-01 16:06:53 | 2021-09-01 16:06:53 |
|  3 | kubota | sleepy        | enjoy Java    | 3.jpg | 2021-09-01 19:16:37 | 2021-09-01 19:17:34 |
+----+--------+---------------+---------------+-------+---------------------+---------------------+
3 rows in set (0.00 sec)
```

### テーブルデータ削除(DELETE系)
- 実行コマンド
```
Message::destroy(3)
$message = Message::find(2)
$message->delete()
```

- 実行結果例
```
>>> Message::destroy(3)
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> 1
>>> $message = Message::find(2)
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
PHP Deprecated:  The "Doctrine/Common/Inflector/Inflector::pluralize" method is deprecated and will be dropped in doctrine/inflector 2.0. Please update to the new Inflector API. in /home/ec2-user/environment/bbs_laravel/vendor/doctrine/inflector/lib/Doctrine/Common/Inflector/Inflector.php on line 264
=> App\Message {#3560
     id: 2,
     name: "yamada",
     title: "good morning!",
     body: "enjoy PHP",
     image: "2.jpg",
     created_at: "2021-09-01 16:06:53",
     updated_at: "2021-09-01 16:06:53",
   }
>>> $message->delete()
=> true
>>> exit
```
exitでtinker終了

#### MySQLでデータ確認
```
mysql> select * from messages;
mysql> select * from messages;
+----+-------+--------+---------------+-------+---------------------+---------------------+
| id | name  | title  | body          | image | created_at          | updated_at          |
+----+-------+--------+---------------+-------+---------------------+---------------------+
|  1 | shima | hello! | enjoy Laravel | 1.jpg | 2021-09-01 16:05:48 | 2021-09-01 16:05:48 |
+----+-------+--------+---------------+-------+---------------------+---------------------+
1 row in set (0.00 sec)
```

## 7. Git/Github
```
git add .
git commit -m "Modelの作成とマイグレーション、ダミーデータ挿入"
git log
git push origin main
```
