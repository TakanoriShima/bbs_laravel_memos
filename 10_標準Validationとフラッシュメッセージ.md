# 10_標準Validationとフラッシュメッセージ、old関数

<p style='text-align: right;'> &copy; 20210907 by Takanori Shima </p>

```
* 以下、Cloud9上でターミナルを起動してコマンドを打つ *
最終的には3つのターミナルを同時に開いておく
- MySQL用(占有)
- Laravelサーバ起動用(占有)
- artisanコマンドを随時実行用
```

## 1. storeアクションの変形
ref) https://qiita.com/maejima_f/items/7691aa9385970ba7e3ed
### app/Http/Controllers/MessagesController.php@store
```php
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
        
        // 現在時刻ともともとのファイル名を組み合わせてランダムなファイル名作成
        $image = time() . $file->getClientOriginalName();
        // アップロードするフォルダ名取得
        $target_path = public_path('uploads/');
        // 画像アップロード処理
        $file->move($target_path, $image);

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
## 2. indexアクションの変形
### app/Http/Controllers/MessagesController.php@index
```php
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        // Messageモデルを使って、MySQLのmessagesテーブルから全データ取得
        $messages = Message::all();
        
        // 連想配列のデータを1セット（viewで引き出すキーワードと値のセット）引き連れてviewを呼び出す
        return view('messages.index', compact('messages'));
    }
``` 

## 3. flash_message.blade.php の変形
### resources/views/commons/flash_message.blade.php
```html
            @if(session('flash_message'))
            <div class="row mt-2">
                <h2 class="text-center col-sm-12">{{ session('flash_message') }}</h1>
            </div>
            @endif
``` 

## 4. errors.blade.php の変形
### resources/views/commons/errors.blade.php
```html
            @if($errors->any())
            <ul class="row mt-2">
            @foreach($errors->all() as $error)  
                <li class="text-center col-sm-12">{{ $error }}</li>
            @endforeach
            </ul>
            @endif
``` 

## 5. createアクションの変形
### app/Http/Controllers/MessagesController.php@create
```php
    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        // 空のメッセージインスタンスを作成
        $message = new Message();
        
        // 連想配列のデータを1セット（viewで引き出すキーワードと値のセット）引き連れてviewを呼び出す
        return view('messages.create', compact('message'));
    }
``` 


## 6. create.blade.phpの変形
### resources/views/messages/create.blade.php
```html
@extends('layouts.app')
@section('title', '新規投稿')
@section('header', '新規投稿')
@section('content')
            <div class="row mt-2">
                <form class="col-sm-12" action="/messages" method="POST" enctype="multipart/form-data">
                    <!-- CSRF 対策(トークン作成) -->
                    {{ csrf_field() }}
                    <!-- 1行 -->
                    <div class="form-group row">
                        <label class="col-2 col-form-label">名前</label>
                        <div class="col-10">
                            <input type="text" class="form-control" name="name" value="{{ old('name') ? old('name') : $message->name }}">
                        </div>
                    </div>
                
                    <!-- 1行 -->
                    <div class="form-group row">
                        <label class="col-2 col-form-label">タイトル</label>
                        <div class="col-10">
                            <input type="text" class="form-control" name="title" value="{{ old('title') ? old('title') : $message->title }}">
                        </div>
                    </div>
                    
                    <!-- 1行 -->
                    <div class="form-group row">
                        <label class="col-2 col-form-label">内容</label>
                        <div class="col-10">
                            <input type="text" class="form-control" name="body" value="{{ old('body') ? old('body') : $message->body }}">
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        <label class="col-2 col-form-label">画像アップロード</label>
                        <div class="col-3">
                            <input type="file" name="image" accept='image/*' onchange="previewImage(this);">
                        </div>
                        <div class="col-7">
                            <img id="preview" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==" style="max-width:200px;">
                        </div>
                    </div>
                    
                    <!-- 1行 -->
                    <div class="form-group row">
                        <div class="offset-2 col-10">
                            <button type="submit" class="btn btn-primary">投稿</button>
                        </div>
                    </div>
                </form>
            </div>
             <div class="row mt-5">
                <a href="/" class="btn btn-primary">投稿一覧</a>
            </div>
@endsection
``` 


## 7. showアクションの変形
### app/Http/Controllers/MessagesController.php@show
```php
    /**
     * Display the specified resource.
     *
     * @param  \App\Message  $message
     * @return \Illuminate\Http\Response
     */
    public function show(Message $message)
    {
        // 連想配列のデータを1セット（viewで引き出すキーワードと値のセット）引き連れてviewを呼び出す
        return view('messages.show', compact('message'));
    }
``` 

## 8. editアクションの変形
### app/Http/Controllers/MessagesController.php@edit
```php
    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Message  $message
     * @return \Illuminate\Http\Response
     */
    public function edit(Message $message)
    {
        // 連想配列のデータを1セット（viewで引き出すキーワードと値のセット）引き連れてviewを呼び出す
        return view('messages.edit', compact('message'));
    }
``` 

## 9. updateアクションの変形
### app/Http/Controllers/MessagesController.php@update
```php
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
        
            // 現在時刻ともともとのファイル名を組み合わせてランダムなファイル名作成
            $image = time() . $file->getClientOriginalName();
            // アップロードするフォルダ名取得
            $target_path = public_path('uploads/');
            // 画像アップロード処理
            $file->move($target_path, $image);

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

## 10. destroyアクションの変形
### app/Http/Controllers/MessagesController.php@destroy
```php
    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Message  $message
     * @return \Illuminate\Http\Response
     */
    public function destroy(Message $message)
    {
        // 該当メッセージをデータベースから削除
        $message->delete();
        // フラッシュメッセージを保存しながらshowアクションにリダイレクト
        return redirect('/')->with('flash_message', 'id: ' . $message->id . 'の投稿を削除しました');
    }
``` 

## 11. エラーメッセージの日本語化

ref) https://qiita.com/samuraibrass/items/d71c08e144dbbf98e348
```
手順1) resources/lang/en フォルダをコピーして langフォルダに張り付け。resources/lang/jaに名前変更
```
```
手順2) resources/lang/ja/validation.php が英語で書かれていることを確認
```
```
手順3) resources/lang/ja フォルダを右クリックし、 Open Terminal Here でターミナルを起動
```
```
手順4) 日本語化ファイルのダウンロード
開いたターミナルで順に以下の4つのコマンドを実行

curl -OL https://raw.githubusercontent.com/rito-nishino/Laravel-Japanese-Language-fileset/master/auth.php
curl -OL https://raw.githubusercontent.com/rito-nishino/Laravel-Japanese-Language-fileset/master/pagination.php
curl -OL https://raw.githubusercontent.com/rito-nishino/Laravel-Japanese-Language-fileset/master/passwords.php
curl -OL https://raw.githubusercontent.com/rito-nishino/Laravel-Japanese-Language-fileset/master/validation.php
```
```
手順5) resources/lang/ja/validation.php が日本語に書き換わっていることを確認
```

```
手順6) resources/lang/ja/validation.php の 157行目附近を書き換え

    'attributes' => [
        'name'          => '名前',
        'title'       => 'タイトル',
        'body'       => '内容',
        'image'      => '画像ファイル',
    ],
```
```
手順7) config/app.php の83行目附近を書き換え

'locale' => 'ja',
```
```
手順8) サーバ再起動

一旦 コントロール+ C で Laravelサーバを停止し、

php artisan serve --host=$IP --port=$PORT
```

## 12. Git/Github
```
git add .
git commit -m "標準validationのカスタマイズ、withメソッドを用いたフラッシュメッセージ使用とold関数を用いた入力値の再代入完了"
git push origin main
```




