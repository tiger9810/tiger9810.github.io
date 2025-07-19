# 作ったアプリの公開方法

<!--
date = "2025-07-19"
-->

## 使ったもの
- [Google Fonts](https://fonts.google.com/icons?icon.size=60&icon.color=%23e3e3e3)
- [Canva](https://www.canva.com/)
- [AppIcon Generator](https://www.appicon.co/)


## やり方
#### 1. アプリアイコンの作成と適用
[Google Fonts](https://fonts.google.com/icons?icon.size=60&icon.color=%23e3e3e3)から使えそうなアイコンを見つけて、[Canva](https://www.canva.com/)で色をつける。そのあと、[AppIcon Generator](https://www.appicon.co/#app-icon)でApp Store / Google Play の規格に対応した複数サイズのアイコン画像を一度に作成する。

#### 2. ストア向けアセット
アプリ画面のスクリーンショットを撮る。エミュレータを起動して、スクリーンショットを撮る。


#### 3. Google play, Apple storeに登録する
[Google Play StoreにAndroidアプリをリリースする手順](https://qiita.com/ike04/items/fe1296854524aacbc141)を参考に登録する。VScodeで開発したので、Android Studioの機能が使えなかったため、以下の対応をした。
##### VScodeで.aab ファイルを作る
Flutter プロジェクトのルートディレクトリで`flutter build appbundle`コマンドを実行する。
```
✓ Built build/app/outputs/bundle/release/app-release.aab (24.3MB)
```
このメッセージが表示されていれば、Flutterは .aab ファイルを正常に生成できている。
##### Flutter用の keystore を手動で作成する
keystoreとは、アプリを開発者本人が作成したことを証明するための「デジタル署名情報」を安全に保存するファイル。ファイル名は通常 .jks（Java KeyStore）という拡張子。Androidアプリ（特にGoogle Play公開用）は、この署名付きでビルドされていないとリリースできない。

##### アプリの名前を決める




---
<details><summary>履歴</summary>

- [2025-06-21 Fri] [重力に負けずに下ろした方が体をコントロールした方が筋肉に効果があるとの研究結果](https://yuchrszk.blogspot.com/2025/06/vs.html?m=0)を見て、ゆっくり動作に変更。10回×5セットで限界を迎えるようになり、残り50回は膝つきで行う。

</details>