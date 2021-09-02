# Blade Helper 記法

## 変数の埋め込み
```
<?= $name ?>

=>

{{ $name }}
```
## if文
```
<?php if(condition): ?>
...
<?php endif; ?>

=>

@if(condition)
...
@endif
```

## foreach
```
<?php foreach($messages as $message): ?>
...
<?php endforeach; ?>

=>

@foreach($messages as $message)
...
@endforeach
```
## a タグ作成
```
{!! link_to_route('messages.create', '新規メッセージの投稿', [], ['class' => 'btn btn-primary']) !!}
```

## Form タグ作成
```
{{, }} で囲った場合は、htmlspecialchars 関数に通したものが出力されます。
{!!, !!} で囲った場合は、そのまま出力されます

{!! Form::model($message, ['route' => 'messages.store']) !!}

    {!! Form::label('content', 'メッセージ:') !!}
    {!! Form::text('content', null, ['class' => 'form-control']) !!}
    
    {!! Form::submit('投稿') !!}

{!! Form::close() !!}
```

## 埋め込み
### 埋め込み先: layout.blade.php
```
@yield('content')
```
### 表示本体の blade
```
@extends('layouts.app')
@section('content')
<!-- ここにページ毎のコンテンツを書く -->
@endsection
```

### 部分テンプレート（パーシャル）組み込み
```
@include('commons.navbar')
@component('components.header')
@endcomponent
```
### 使い分け
```
ファイルを丸ごといれるなら@include
一つのテンプレートをつかうなら@exclude
要素ごとに分けてテンプレートをつくなら components
```