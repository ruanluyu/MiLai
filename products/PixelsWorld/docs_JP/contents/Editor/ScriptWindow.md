# 世界ルール　エディター
-------

## エディター
プラグインパネルのWorld formulaとなりのEditボタンを押すと、世界ルール　エディターが開かれます。

![World formula editor](formulaEditor.png)

ここで、記述便宜上、異なるエリアを次のようにハイライトしておきます。

![World formula editor(colored)](FE_AreaInfo.png)

## <span style="color:rgb(255,0,0)">◆</span>コードエリア

コードエリアで、コード入力はできます。

> このエリアの`Language`設定は、プラグインパネルにある`Language`設定より優先度が高いです。`(default)`とは「パネルのほうの設定を変えずに維持する」という意味です。


## <span style="color:rgb(56,148,228)">◆</span>参照リスト

ここで、よく使われているコードが挙げられています。現在、このリストは読み取り専用です。


## <span style="color:rgb(243,27,243)">◆</span>パラメーター設定

ここで、パラメーター名とパラメーター初期値を一覧することができます。`Parameter manager`ボタンからパラメーター管理パネルが開けます。[Parameter manager](ParameterWindow.md)も参照してください。



> パラメーター設定はコードと一緒にプリセットに保存できます。

## <span style="color:rgb(153,204,0)">◆</span>プリセットリスト

すべてのローカルプリセットファイルはここのポップアップメニューにあります。

プリセットに関するの定義をここでここで述べておきます。

- ファイル名の拡張子が `.pwp` で、 `PixelsWorld.aex`の隣にある**JSON** 様式のファイルを「プリセットコラム」と我々は呼びます。
- 一つのプリセットコラムの中に若干の「プリセット」が入っています。
- 「プリセット」はポップアップメニューの下のリストに列挙されています。

## <span style="color:rgb(128,0,0)">◆</span>コードプレビュー

選択されているプリセットの内容をここでチェックすることができます。`Replace`ボタンを押すと、選択されたプリセットのコードを左のコードエリアに既存のコードを置き換えられます。


> コードのみを置換し、設定を置き換えたくない場合は、<span style="color:rgb(247,136,58)">パラメーター設定プレビュー</span>の`Apply when replace`のチェックを外してから`Replace`ボタンを押してください。 

プリセットコードを今のコードを上書きせず、先頭につなげたい場合は`<<Top`ボタンを、後ろにつなげたい場合は`Bottom>>`ボタンを押してください。

## <span style="color:rgb(247,136,58)">◆</span>パラメーター設定プレビュー

選択されているプリセットのパラメーター設定をここでチェックできます。プリセットの設定を適用するには`Apply`ボタンを押してください。
> `Apply when replace`は「コードエリアの`Replace`ボタンを押すと自動的にこのエリアの`Apply`ボタンを押してくれる」という意味です。


<br>
<br>
<br>
<br>
<br>
<br>