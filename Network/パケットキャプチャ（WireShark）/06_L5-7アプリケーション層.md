
# HTTP
- HTTP/1.1とHTTP/2が混在。主要なブラウザ等はすでに対応済み。サーバが対応していない場合もある。
![image](https://user-images.githubusercontent.com/60077121/99460605-7e303b00-2973-11eb-8d37-5f73bc44b0d4.png)

## HTTPメッセージ
- HTTPメッセージ：HTTPでやり取りする情報全般。以下2種類
  - リクエストメッセージ
  - レスポンスメッセージ
- どちらも`スタートライン`、`メッセージヘッダ`、`メッセージボディ`で構成
![image](https://user-images.githubusercontent.com/60077121/99460844-f72f9280-2973-11eb-81f0-764dc12d08c1.png)

### リクエストメッセージ
<img width="475" alt="スクリーンショット 2020-11-18 8 00 46" src="https://user-images.githubusercontent.com/60077121/99461064-65745500-2974-11eb-9d1b-eb1f189d36e7.png">

- リクエストライン：以下で構成
  - メソッド
  - リクエストURI
    - ![image](https://user-images.githubusercontent.com/60077121/99461797-c6505d00-2975-11eb-8e84-a5b53b09bb32.png)
  - HTTPバージョン
 
 ### レスポンスメッセージ
 - 以下で構成
 ![image](https://user-images.githubusercontent.com/60077121/99462046-3e1e8780-2976-11eb-9d55-8899e0001831.png)

- ステータスライン：以下で構成
  - HTTPバージョン
  - ステータスコード（200とか）
  - リーズンフレーズ

### HTTPヘッダー
- メッセージヘッダーのこと。以下で構成
  - リクエストヘッダー
  - レスポンスヘッダー
  - 一般ヘッダー
  - エンティティヘッダー
  - その他のヘッダー
  
#### リクエストヘッダー(RFC2616)**クライアントがサーバに送る**
![image](https://user-images.githubusercontent.com/60077121/99462580-55aa4000-2977-11eb-8d21-0d3599b5d688.png)
![image](https://user-images.githubusercontent.com/60077121/99462615-6b1f6a00-2977-11eb-9fef-ebb03603d25f.png)

- Hostヘッダ：HTTP/1.1で唯一必須のヘッダ
  - リクエスト先のホスト（URL）とポート番号。ひとつのIPで複数ドメインを運用するバーチャルホストで活躍
  ![image](https://user-images.githubusercontent.com/60077121/99463043-411a7780-2978-11eb-90ac-1573cfd56f2d.png)

- Referヘッダー
  - どこのリンクから飛んできた示す
  - 信頼できるリンク元のみ受け入れることによって`CSRF（クロスサイトリクエストフォージリ）`に有効
  - **ただしツールで簡単に改変可能**
  - サーバAのセッション管理にCookie方式やHiddenフィールド方式ではなくURI埋め込み方式を利用している場合、ReferヘッダにセッションIDが入ってしまうので他サーバBに飛んだ時にサーバBからセッションID見える危険。

#### 一般ヘッダ（RFC2616）行き帰りどちらのメッセージにも利用される
![image](https://user-images.githubusercontent.com/60077121/99463939-4082e080-297a-11eb-8e15-3b10fc1cae34.png)

##### Cache-Controlヘッダー
ディレクティブという値をフィールドにセットする
- `no-store`：相手にキャッシュさせない
- `no-cache`：相手に最新の情報かどうか確認してから使用する。確認時は相手から304NotModifiedが返ってきたらキャッシュを利用
- `max-age`：キャッシュ保持の時間（秒）。86400なら1日

![image](https://user-images.githubusercontent.com/60077121/99464194-e0d90500-297a-11eb-9854-30320e2b6a84.png)
![image](https://user-images.githubusercontent.com/60077121/99464217-f1897b00-297a-11eb-8821-4eda3ad9a15e.png)

#### エンティティヘッダー
![image](https://user-images.githubusercontent.com/60077121/99597898-0c6ff400-2a3c-11eb-8d5b-8589e52e5ad6.png)

- `Content-Encoding`,`Accept-Encoding`
<img width="398" alt="スクリーンショット 2020-11-19 7 52 54" src="https://user-images.githubusercontent.com/60077121/99598082-578a0700-2a3c-11eb-8311-cfc3afbe2cc2.png">

- `Content-length`：HTTP/1.1以降はキープアライブによってひとつのコネクションを使い回すのでメッセージボディの長さを伝える必要がある

#### レスポンスヘッダー
![image](https://user-images.githubusercontent.com/60077121/99598903-a8e6c600-2a3d-11eb-9302-3ed5cbb4a71e.png)

- `E-Tag`：サーバの持つファイルに割り当てられた一意の文字列。ファイルが更新されるとE-Tagも更新。キャッシュに使われていそう。
  - リクエストヘッダーの`If-Match``If-None-Match`に使用される
![image](https://user-images.githubusercontent.com/60077121/99599117-0844d600-2a3e-11eb-8734-65f57308c108.png)

#### その他のヘッダー
- `Set-Cookie`、`Cookie`ヘッダー
<img width="481" alt="スクリーンショット 2020-11-19 8 07 45" src="https://user-images.githubusercontent.com/60077121/99599327-7b4e4c80-2a3e-11eb-9564-d468578f2c14.png">

## HTTPの解析に役立つWireshark機能
割愛

## HTTP解析
割愛
