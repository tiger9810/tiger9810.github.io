# 作ったアプリの公開方法

<!--
date = "2025-07-19"
-->

## 使ったもの
- [Google Fonts](https://fonts.google.com/icons?icon.size=60&icon.color=%23e3e3e3)
- [Canva](https://www.canva.com/)
- [AppIcon Generator](https://www.appicon.co/)
- [AppLaunchpad](https://theapplaunchpad.com/projects)


## やり方
### 1. アプリアイコンの作成と適用

[Google Fonts](https://fonts.google.com/icons?icon.size=60&icon.color=%23e3e3e3)から使えそうなアイコンを見つけて、[Canva](https://www.canva.com/)で色をつける。
その後、[AppIcon Generator](https://www.appicon.co/)でAndroid, iOSに対応した画像セットを作成し、出力された画像データを適切なフォルダに配置する。
##### Android
```
your_flutter_project/
└── android/
    └── app/
        └── src/
            └── main/
                └── res/
                    ├── mipmap-hdpi/
                    ├── mipmap-mdpi/
                    ├── mipmap-xhdpi/
                    ├── mipmap-xxhdpi/
                    └── mipmap-xxxhdpi/

```
##### iOS
```
your_flutter_project/
└── ios/
    └── Runner/
        └── Assets.xcassets/
            └── AppIcon.appiconset/

```

### 3. Google playに登録する
[Google Play StoreにAndroidアプリをリリースする手順](https://qiita.com/ike04/items/fe1296854524aacbc141)を参考に登録する。VScodeで開発したので、Android Studioの機能が使えなかったため、以下の対応をした。
##### VScodeで.aab ファイルを作る
Flutter プロジェクトのルートディレクトリで
```flutter build appbundle```
コマンドを実行する。
```
✓ Built build/app/outputs/bundle/release/app-release.aab (24.3MB)
```
このメッセージが表示されていれば、Flutterは .aab ファイルを正常に生成できている。






---
<details><summary>履歴</summary>

- [2025-06-21 Fri] [重力に負けずに下ろした方が体をコントロールした方が筋肉に効果があるとの研究結果](https://yuchrszk.blogspot.com/2025/06/vs.html?m=0)を見て、ゆっくり動作に変更。10回×5セットで限界を迎えるようになり、残り50回は膝つきで行う。

</details>