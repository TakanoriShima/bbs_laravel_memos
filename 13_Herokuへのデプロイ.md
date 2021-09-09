# 13_Herokuへのデプロイ

<p style='text-align: right;'> &copy; 20210907 by Takanori Shima </p>

```
* 以下、Cloud9上でターミナルを起動してコマンドを打つ *
最終的には3つのターミナルを同時に開いておく
- MySQL用(占有)
- Laravelサーバ起動用(占有)
- artisanコマンドを随時実行用
```

## 1. コマンドラインツールのインストール
### ref) https://qiita.com/sasasoni/items/d93224e9b62b524b1dc6

```
wget https://cli-assets.heroku.com/heroku-linux-x64.tar.gz -O heroku.tar.gz
sudo mkdir -p /usr/local/lib/heroku
sudo tar --strip-components 1 -zxvf heroku.tar.gz -C /usr/local/lib/heroku
sudo ln -s /usr/local/lib/heroku/bin/heroku /usr/local/bin/heroku
```

## 2. Heroku にログイン
```
heroku login -i
```

## 3. Herokuアプリ作成
```
heroku create quark2galaxy-bbs-laravel
```

## 4. Herokuアプリ一覧取得
```
heroku apps
```

表示例
```
ec2-user:~/environment/bbs_laravel (master) $ heroku apps
=== quark2galaxy@gmail.com Apps
quark2galaxy-bbs-laravel
```

## 5. リモートリポジトリ heroku の確認
```
git remote -v
```
> heroku  https://git.heroku.com/quark2galaxy-bbs-laravel.git (fetch)<br>
> heroku  https://git.heroku.com/quark2galaxy-bbs-laravel.git (push)<br>
> origin  https://github.com/TakanoriShima/bbs_laravel.git (fetch)<br>
> origin  https://github.com/TakanoriShima/bbs_laravel.git (push)

## 6. Heroku 用設定ファイルを新規作成
```
echo "web: vendor/bin/heroku-php-apache2 public/" > Procfile
```

## 7. Heroku アプリの環境変数の設定
```
heroku config:set APP_KEY=$(php artisan --no-ansi key:generate --show)
```
>Setting APP_KEY and restarting ⬢ quark2galaxy-bbs-laravel... done, v3<br>
>APP_KEY: base64:Nl9uPLdkE20+6OLEYT80mEvLlHehUlkqMaM3lHhmPkA=

## 8. 環境変数の確認方法
```
heroku config
```
>=== quark2galaxy-bbs-laravel Config Vars<br>
>APP_KEY: base64:Nl9uPLdkE20+6OLEYT80mEvLlHehUlkqMaM3lHhmPkA=

## 9. Heroku 上でPostgreSQLデータベースを作成
Heroku 上のpostgreSQL v13.4
```
heroku addons:create heroku-postgresql:hobby-dev
```
>Creating heroku-postgresql:hobby-dev on ⬢ quark2galaxy-bbs-laravel.. free
>Database has been created and is available
> ! This database is empty. If upgrading, you can transfer<br>
> ! data from another database with pg:copy<br>
>Created postgresql-perpendicular-61502 as DATABASE_URL<br>
>Use heroku addons:docs heroku-postgresql to view documentation<br>

## 10. 再度、環境変数の確認方法
```
heroku config
```
>=== quark2galaxy-bbs-laravel Config Vars<br>
>APP_KEY:      base64:Nl9uPLdkE20+6OLEYT80mEvLlHehUlkqMaM3lHhmPkA=<br>
>DATABASE_URL: postgres://iyyvdalowlxgjp:d58e9a7a0c4a529ed72fbe82c763d14dfc3db9c48fda82cb1ee4a7be9ecde484@ec2-54-156-24-159.compute-1.amazonaws.com:5432/d4vf665pmc8g33

## 11. データベース用環境設定
```
一般書式)
DATABASE_URL: postgres://ユーザ名:パスワード@ホスト名:5432/データベース名
今の設定)
DATABASE_URL: postgres://iyyvdalowlxgjp:d58e9a7a0c4a529ed72fbe82c763d14dfc3db9c48fda82cb1ee4a7be9ecde484@ec2-54-156-24-159.compute-1.amazonaws.com:5432/d4vf665pmc8g33

見比べて以下の対応が付く
ユーザ名: iyyvdalowlxgjp
パスワード: d58e9a7a0c4a529ed72fbe82c763d14dfc3db9c48fda82cb1ee4a7be9ecde484
ホスト名: ec2-54-156-24-159.compute-1.amazonaws.com
データベース名: 4vf665pmc8g33

そして
一般書式)
heroku config:set DB_CONNECTION=pgsql DB_USERNAME=ユーザ名 DB_PASSWORD=パスワード DB_HOST=ホスト名 DB_DATABASE=データベース名

に当てはめて以下の作成し、実行する

heroku config:set DB_CONNECTION=pgsql DB_USERNAME=iyyvdalowlxgjp DB_PASSWORD=d58e9a7a0c4a529ed72fbe82c763d14dfc3db9c48fda82cb1ee4a7be9ecde484 DB_HOST=ec2-54-156-24-159.compute-1.amazonaws.com DB_DATABASE=4vf665pmc8g33

```
> Setting DB_CONNECTION, DB_USERNAME, DB_PASSWORD, DB_HOST, DB_DATABASE and restarting ⬢ quark2galaxy-bbs-laravel... done, v6<br>
> DB_CONNECTION: pgsql<br>
> DB_DATABASE:   dctg5apf3kva5c<br>
> DB_HOST:       ec2-52-72-125-94.compute-1.amazonaws.com<br>
> DB_PASSWORD:   32d35519690b4a91f35fffda64c8b3748bdda8dd8153fe04da013b5483ad1356<br>
> DB_USERNAME:   ygtjmxlenghsoc

## 12. Git
```
git add .
git commit -m 'Heroku用にProcfileを追加、PostgreSQLのアドオン、環境設定完了'
git push origin main
git push heroku main
```

## 13. Heroku上でマイグレーション実施
```
heroku run php artisan migrate
```
> Do you really wish to run this command? (yes/no) [no]:<br>
> > yes<br>
><br>
>Migration table created successfully.<br>
>Migrating: 2014_10_12_000000_create_users_table<br>
>Migrated:  2014_10_12_000000_create_users_table (0.03 seconds)<br>
>Migrating: 2014_10_12_100000_create_password_resets_table<br>
>Migrated:  2014_10_12_100000_create_password_resets_table (0.02 seconds)<br>
>Migrating: 2021_08_30_152643_create_profiles_table<br>
>Migrated:  2021_08_30_152643_create_profiles_table (0.06 seconds)<br>
>Migrating: 2021_08_30_162219_create_posts_table<br>
>Migrated:  2021_08_30_162219_create_posts_table (0.02 seconds)<br>
>Migrating: 2021_08_30_172053_create_comments_table<br>
>Migrated:  2021_08_30_172053_create_comments_table (0.03 seconds)<br>
>Migrating: 2021_08_30_185847_create_favorites_table<br>
>Migrated:  2021_08_30_185847_create_favorites_table (0.06 seconds)<br>
>Migrating: 2021_08_31_110656_create_user_follow_table<br>
>Migrated:  2021_08_31_110656_create_user_follow_table (0.03 seconds)<br>

## 14. Cloud9 に PostgreSQLをインストール
```
sudo yum install postgresql95-devel postgresql95-server postgresql95-contrib
psql --version

heroku pg:psql
\l
\c d4vf665pmc8g33
\dt;
# \d messages;
select * from messages;
\q
```

## 15. S3 設定
S3用のパッケージをインストール
```
composer remove league/flysystem
composer require league/flysystem-aws-s3-v3 ~1.0
composer license
```

## 16. Heroku側にAWSアクセスキー設定環境設定

```
heroku config:set AWS_ACCESS_KEY_ID="xxxx"
heroku config:set AWS_SECRET_ACCESS_KEY="xxxx"
heroku config:set AWS_DEFAULT_REGION="ap-northeast-1"
heroku config:set AWS_BUCKET="quark2galaxy-bbs"
```

## 17. 最新の環境変数確認
- 実行コマンド
```
heroku config
```
- 実行結果例
```
APP_KEY:               base64:Dfc7bH0fcnHox9IMIDkUYZh8qNRLrmk/qkfaDqodZhE=
AWS_ACCESS_KEY_ID:     xxxx
AWS_BUCKET:            quark2galaxy-instagram
AWS_DEFAULT_REGION:    ap-northeast-1
AWS_SECRET_ACCESS_KEY: xxxx
DATABASE_URL:          postgres://seythccoltjtuw:1fd378b1377fee5ee14bbaa93e4b43034d0e0ae741b42ae9a8ba31568ff5bd3d@ec2-52-203-74-38.compute-1.amazonaws.com:5432/d3mi8lvt2pitsj
DB_CONNECTION:         pgsql
DB_DATABASE:           d3mi8lvt2pitsj
DB_HOST:               ec2-52-203-74-38.compute-1.amazonaws.com
DB_PASSWORD:           1fd378b1377fee5ee14bbaa93e4b43034d0e0ae741b42ae9a8ba31568ff5bd3d
DB_USERNAME:           seythccoltjtuw
```

## 18. storeアクションの編集
### app/Http/Controllers/MessagesController.php@store
```
use Illuminate\Support\Facades\Storage;

/**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        
        // validation
        $this->validate($request, [
            'name' => 'required',
            'title' => 'required',
            'body' => 'required',
            'image' => [
                'required',
                'file',
                'mimes:jpeg,jpg,png'
            ]
        ]);
        
        // 入力された値を取得
        $name = $request->input('name');
        $title = $request->input('title');
        $body = $request->input('body');
        // 画像ファイル情報の取得だけ特殊
        $file =  $request->image;
        
        // // 現在時刻ともともとのファイル名を組み合わせてランダムなファイル名作成
        // $image = time() . $file->getClientOriginalName();
        // // アップロードするフォルダ名取得
        // $target_path = public_path('uploads/');
        // // 画像アップロード処理
        // $file->move($target_path, $image);

        // S3用
        $path = Storage::disk('s3')->putFile('/uploads', $file, 'public');
        // パスから、最後の「ファイル名.拡張子」の部分だけ取得
        $image = basename($path);

        // 空のメッセージインスタンスを作成
        $message = new Message();
        
        // 入力された値をセット
        $message->name = $name;
        $message->title = $title;
        $message->body = $body;
        $message->image = $image;

        // メッセージインスタンスをデータベースに保存
        $message->save();
        
        // セッションにフラッシュメッセージを保存しながら、indexアクションにリダイレクト
        return redirect('/')->with('flash_message', '新規投稿が成功しました');
        
    }
```

## 19. updateアクションの編集
### app/Http/Controllers/MessagesController.php@update
```

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Message  $message
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Message $message)
    {
        
        // validation
        $this->validate($request, [
            'name' => 'required',
            'title' => 'required',
            'body' => 'required',
        ]);
        
        // 入力された値を取得
        $name = $request->input('name');
        $title = $request->input('title');
        $body = $request->input('body');
        // 画像ファイル情報の取得だけ特殊
        $file =  $request->image;
        
        // 画像ファイルが選択されていれば
        if ($file) { 
        
            // // 現在時刻ともともとのファイル名を組み合わせてランダムなファイル名作成
            // $image = time() . $file->getClientOriginalName();
            // // アップロードするフォルダ名取得
            // $target_path = public_path('uploads/');
            // // 画像アップロード処理
            // $file->move($target_path, $image);
            
            // S3用
            $path = Storage::disk('s3')->putFile('/uploads', $file, 'public');
            // パスから、最後の「ファイル名.拡張子」の部分だけ取得
            $image = basename($path);

        } else { // ファイルが選択されていなければ元の値を保持
            $image = $message->image;
        }
        
        // 入力されたでインスタンス情報を更新
        $message->name = $name;
        $message->title = $title;
        $message->body = $body;
        $message->image = $image;

        // データベースを更新
        $message->save();
        
        // フラッシュメッセージを保存しながらshowアクションにリダイレクト
        return redirect('/messages/' . $message->id)->with('flash_message', 'id: ' . $message->id . 'の投稿の更新が成功しました');

    }
```

## 20. show.blade.php の編集
### resources/views/messages/show.blade.php
```html
@extends('layouts.app')
@section('title', '投稿詳細')
@section('header', 'id: ' .  $message->id . 'の投稿詳細')
@section('content')
            <div class="row mt-2">
                <table class="table table-bordered table-striped">
                    <tr>
                        <th>id</th>
                        <td>{{ $message->id }}</td>
                    </tr>
                    <tr>
                        <th>名前</th>
                        <td>{{ $message->name }}</td>
                    </tr>
                    <tr>
                        <th>タイトル</th>
                        <td>{{ $message->title }}</td>
                    </tr>
                    <tr>
                        <th>内容</th>
                        <td>{{ $message->body }}</td>
                    </tr>
                    <tr>
                        <th>画像</th>
                        <td><img src="{{ Storage::disk('s3')->url('uploads/' . $message->image) }}" alt="表示する画像がありません。"></td>
                    </tr>
                </table>
            </div> 
            
            <div class="row">
                {!! link_to_route('messages.edit', '編集', ['id' => $message->id], ['class' => 'col-sm-6  btn btn-primary']) !!}
                
                {!! Form::model($message, ['route' => ['messages.destroy', $message->id], 'method' => 'delete', 'class' => 'col-sm-6']) !!}
                    {!! Form::submit('削除', ['class' => 'btn btn-danger col-sm-12', 'onclick' => 'return confirm("投稿を削除します。よろしいですか？")']) !!}
                {!! Form::close() !!}
            </div>       
        
             <div class="row mt-5">
                {!! link_to_route('messages.index', '投稿一覧', [], ['class' => 'btn btn-primary']) !!}
            </div>
@endsection

```

## 21. edit.blade.php の編集
### resources/views/messages/edit.blade.php
```html
@extends('layouts.app')
@section('title', '投稿編集')
@section('header', 'id: ' .  $message->id . 'の投稿編集')
@section('content')
            <div class="row mt-2">
                
                {!! Form::model($message, ['route' => ['messages.update', $message->id], 'method' => 'put', 'enctype' => 'multipart/form-data', 'class' => 'col-sm-12']) !!}
                
                    <div class="form-group row">
                        {!! Form::label('name', '名前', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('name', $message->name, ['class' => 'form-control']) !!}
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        {!! Form::label('title', 'タイトル', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('title', $message->title, ['class' => 'form-control']) !!}
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        {!! Form::label('body', '内容', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('body', $message->body, ['class' => 'form-control']) !!}
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        <label class="col-2 col-form-label">現在の画像</label>
                        <div class="col-10">
                            <img src="{{ Storage::disk('s3')->url('uploads/' . $message->image) }}" alt="表示する画像がありません。">
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        {!! Form::label('image', '画像アップロード', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-3">
                            {!! Form::file('image', ['accept' => 'image/*', 'onchange' => "previewImage(this)"]) !!}
                        </div>
                        <div class="col-7">
                            <img id="preview" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==" style="max-width:200px;">
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        <div class="offset-2 col-10">
                            {!! Form::submit('更新', ['class' => 'btn btn-primary']) !!}
                        </div>
                    </div>

                {!! Form::close() !!}
                
            </div>                
            <div class="row mt-5">
                {!! link_to_route('messages.index', '投稿一覧', [], ['class' => 'btn btn-primary']) !!}
            </div>
@endsection
```

## 22. 設定ファイルのキャッシュクリアとサーバの再起動
```
php artisan config:cache
```

## 23. Git
```
git add .
git commit -m 'AWS S3の画像をアップロードする設定完成'
git push origin main
git push heroku main
```

## 24. Herokuアプリの起動(OK)
以下のURLにアクセス
```
https://quark2galaxy-bbs-laravel.herokuapp.com/
```


