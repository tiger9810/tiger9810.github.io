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

### 4. Google play アプリ署名の登録方法
Google Play アプリ署名（Play App Signing）とは、Google に署名鍵（リリースキー）を安全に管理してもらい、開発者は「アップロード鍵」で署名して .aab を提出するだけでよくなる仕組みです。
##### アップロード鍵を用意する
アップロード用キーストア（upload keystore）は、自分で作成・保管する必要があるファイルです。Google はそれを自動的に作成・提供は しません。下記のコマンドを実行して質問に答えることで、作成できた。

```python
# アップロード用キーストア
keytool -genkey -v \
 -keystore upload-keystore.jks \
 -keyalg RSA -keysize 2048 -validity 10000 \
 -alias upload
```

```python
# 各コマンドの意味
keytool -genkey -v \
  -keystore upload-keystore.jks \         # ← 作成するキーストアのファイル名
  -keyalg RSA \                           # ← 鍵のアルゴリズム（RSA）
  -keysize 2048 \                         # ← 鍵のビット数（2048bit）
  -validity 10000 \                       # ← 有効期間（日数）→ 約27年
  -alias upload                           # ← この鍵の名前（別名）
```
コマンドを実行すると、以下のような質問(情報入力)が求められます：
```
Keystore password: ●●●●●●
Your name and surname: 
Your organizational unit: 
Your organization: 
Your City or Locality: 
Your State or Province: 
Your Country Code (XX): 
```

```
| 入力画面での質問                       | `key.properties` での設定 |
| ----------------------------------- | --------------------- |
| **Enter keystore password:**        | ✅ `storePassword`     |
| **Enter key password for <alias>:** | ✅ `keyPassword`       |
| **-alias upload**                   | ✅ `keyAlias`          |

```
実行場所はどこでも良い。
```
| 条件.    | 内容                                                                      |
| ------- | ------------------------------------------------------------------------- |
| 実行場所 | 任意のディレクトリでOK（例：ホームディレクトリ）                                  |
| 出力先   | コマンドを実行した場所に `upload-keystore.jks` が作成されます                   |
| 次の作業 | Flutter プロジェクトの `android/key.properties` にそのファイルパスを指定します    |
```
keytool -genkey コマンドを実行すると、キーストア（署名鍵）を作成するための対話形式の質問が表示されます。以下、それぞれの入力項目の意味と入力例を説明します。
```python
# 入力項目の意味と入力例
| 質問                           | 意味                       | 入力例                             |
| ---------------------------- | --------------------------- | --------------------------------- |
| **Keystore password**        | `.jks` ファイル全体のパスワード | `MySecureStore123!`（自分で決める）  |
| **Your name and surname**    | 開発者の名前（実名または会社名） | `Taro Yamada` または `MyApp Inc.`   |
| **Your organizational unit** | 部署名など（空欄でもOK）       | `Development`                      |
| **Your organization**        | 会社名または個人事業名         | `YamadaWorks`                      |
| **Your City or Locality**    | 市区町村                     | `Shibuya`                          |
| **Your State or Province**   | 都道府県                     | `Tokyo`                            |
| **Your Country Code (XX)**   | 国コード（2文字）             | `JP`（日本の場合）                    |

```

##### プロジェクトルート/androidフォルダにkey.propertiesを作成する
key.properties ファイルを作成して、下記内容を記述する。
```
storePassword=your_store_password
keyPassword=your_key_password
keyAlias=upload
storeFile=/Users/yourname/path/to/upload-keystore.jks
```
##### VScodeで.aab ファイルを作る
Flutter プロジェクトのルートディレクトリで、下記のコマンドを実行する。
```
flutter build appbundle
```

`✓Build`メッセージが表示されていれば、Flutterは .aab ファイルを正常に生成できている。
```
✓ Built build/app/outputs/bundle/release/app-release.aab (24.3MB)
```

aabファイルをアップロードした際に、`アップロードされた APK または Android App Bundle がデバッグモードで署名されています。APK または Android App Bundle はリリースモードで署名する必要があります。詳しくは、署名についての説明をご覧ください。`と表示される場合は、android/app/build.gradleでリリースビルドでデバッグキーを使用している設定になっている可能性があります。
```
signingConfig = signingConfigs.debug
```
また、私はandroid/app/build.gradleファイルのandroid{}の先頭（pluginsブロックの前）に以下のコードが必要だったので、追記しました。
```
  def keystorePropertiesFile = rootProject.file("key.properties")
  def keystoreProperties = new Properties()
  keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
```

修正箇所まとめ
`android/app/build.gradle`で先頭に下記を追加
```
def keystorePropertiesFile = rootProject.file("key.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
```

buildTypesの中身も`signingConfig = signingConfigs.debug`を削除して、下記に書き換え。
```
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
```
また、`signingConfigs`も追記。
```
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
        }
    }
```

### 発生したトラブル

##### アプリは Android 15（API レベル 35） 以降を対象とする
FlutterでターゲットSDKを更新するには、次のように`targetSdk`の設定を変更
```python
defaultConfig {
    # 他の定義
    targetSdk = 35
}
```
アプリバージョンを上げてから再び下記コマンドを実行し、aabファイルを作成する。
```
flutter clean
flutter build appbundle
```
作成されたaabファイルをplayコンソールにアップロードし、新しいリリースを作成することによって、トラブルを解消できる。


---
<details><summary>履歴</summary>

- [2025-07-26 Sat] keystoreについて追記。トラブルに関しては以下のようなプロンプトをGemini-cliに入力して原因を見つけてもらった。
```
flutter build appbundleを実行した際に作られたaabファイルをアップロードすると、アップロードされた APK または Android App 
  Bundle がデバッグモードで署名されています。APK または Android App Bundle 
  はリリースモードで署名する必要があります。詳しくは、署名についての説明をご覧ください。と言われてしまい、リリースすることがで
  きませんでした。key.propertiesは正しく設定できていると思うので、原因がよくわかりません。関連するファイルを確認して原因を見つけてください。
```

</details>