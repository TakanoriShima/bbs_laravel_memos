# 12_Laravel Collectiveの導入とviewの変形

<p style='text-align: right;'> &copy; 20210907 by Takanori Shima </p>

```
* 以下、Cloud9上でターミナルを起動してコマンドを打つ *
最終的には3つのターミナルを同時に開いておく
- MySQL用(占有)
- Laravelサーバ起動用(占有)
- artisanコマンドを随時実行用
```

## 1. Laravel Collective のインストール
```
composer require "laravelcollective/html":"5.8.*"
```

## 2. index.blade.php の変形
### resources/views/messages/index.blade.php
```html
@extends('layouts.app')
@section('title', '簡易掲示板')
@section('header', '投稿一覧')
@section('content')
            <div class="row mt-2">
                @if(count($messages) !== 0) 
                <table class="col-sm-12 table table-bordered table-striped">
                    <tr>
                        <th>ID</th>
                        <th>ユーザ名</th>
                        <th>タイトル</th>
                        <th>内容</th>
                        <th>投稿時間</th>
                    </tr>
                    </tr>
                    @foreach($messages as $message)
                    <tr>
                        <td>{!! link_to_route('messages.show', $message->id, ['id' => $message->id], []) !!}</td>
                        <td>{{ $message->name }}</td>
                        <td>{{ $message->title }}</td>
                        <td>{{ $message->body }}</td>
                        <td>{{ $message->created_at }}</td>
                    </tr>
                    @endforeach
                </table>
                
                <!-- ページネーションのために追記 -->
                <div class="row">
                    <div class="col-sm-12">
                        {{ $messages->links() }}
                    </div>
                </div>
                
                @else
                <p>データ一件もありません。</p>
                @endif
            </div>
            <div class="row mt-5">
                {!! link_to_route('messages.create', '新規投稿', [], ['class' => 'btn btn-primary']) !!}
            </div>
            <div class="row mt-5">
                
            </div>
        </div>
@endsection
```

### link_to_route(第一引数, 第二引数, 第三引数, 第四引数) の書式
aタグを生成
```
第一引数: 名前付きルーティング名
第二引数: クリックする文字
第三引数: 引き連れれていくクエリパラメータ一覧を連想配列でセット
第四引数: タグの属性を連想配列でセット
```
## 3. show.blade.php の変形
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
                        <td><img src="{{ asset('uploads/' . $message->image) }}" alt="表示する画像がありません。"></td>
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

### Form::model(第一引数, 第二引数) の書式
formタグを作成
```
第一引数: 操作するインスタンス
第二引数: 各種設定値を連想配列で指定
削除には　'method' => 'delete' が必要
```
第二引数の連想配列の route というキー値に関しては
```
第一要素: 名前付きルーティング名
第二要素以降: コントローラに引き連れていくデータを並べる
```

### Form::submit('投稿', ['class' => 'btn btn-primary']) の書式
ボタンを作成 
```
第一引数: ボタンのラベル
第二引数: タグの属性値を連想配列でセット
```

## 4. create.blade.php の変形
### resources/views/messages/create.blade.php
```html
@extends('layouts.app')
@section('title', '新規投稿')
@section('header', '新規投稿')
@section('content')
            <div class="row mt-2">
                
                
                {!! Form::model($message, ['route' => 'messages.store', 'enctype' => 'multipart/form-data', 'class' => 'col-sm-12']) !!}
                
                    <div class="form-group row">
                        {!! Form::label('name', '名前', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('name', old('name') ? old('name') : $message->name, ['class' => 'form-control']) !!}
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        {!! Form::label('title', 'タイトル', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('title', old('title') ? old('title') : $message->title, ['class' => 'form-control']) !!}
                        </div>
                    </div>
                    
                    <div class="form-group row">
                        {!! Form::label('body', '内容', ['class' => 'col-2 col-form-label']) !!}
                        <div class="col-10">
                            {!! Form::text('body', old('body') ? old('body') : $message->body, ['class' => 'form-control']) !!}
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
                            {!! Form::submit('投稿', ['class' => 'btn btn-primary']) !!}
                        </div>
                    </div>

                {!! Form::close() !!}
                
            </div>
             <div class="row mt-5">
                {!! link_to_route('messages.index', '投稿一覧', [], ['class' => 'btn btn-primary']) !!}
            </div>
@endsection
```

### Form::label(第一引数, 第二引数, 第三引数) の書式
labelタグを生成
```
第一引数: id属性値を設定
第二引数: ラベルの文字
第三引数: タグの各種属性を連想配列でセット
```

### Form::text(第一引数, 第二引数, 第三引数) の書式
```
第一引数: name属性値をセット
第二引数: デフォルトで表示するデータ
第三引数: タグの各種属性を連想配列でセット
```

### Form::file(第一引数, 第二引数) の書式
```
第一引数: name属性値をセット
第二引数: タグの各種属性を連想配列でセット
```

## 5. edit.blade.php の変形
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
                            <img src="{{ asset('uploads/' . $message->image) }}" alt="表示する画像がありません。">
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

## 6. Git/Github
```
git add .
git commit -m "Laravel Collectiveの導入とviewの変形"
git push origin main
```




