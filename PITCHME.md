# remote purchaseについて

---

## って何？

UnityIAPの機構を使った課金処理。

ゲームサーバと通信する   
課金処理を実装できる。  

--- 

## メリット
* 誰が何持ってるか管理可能
* この人にはこれ売ろができる
* アプデ無しでアイテムの追加
* お問い合わせに対応しやすい

---

## デメリット
* 管理が面倒
* ハックされないわけではない
* つまりお問い合わせが来る

---

## 準備と動作のフロー

先ずはクライアント側のみ。

1. 各プラットフォームにアイテムをセット
1. PurchaseSettings.csのURL調整
1. アイテムを売る画面を作る
1. Autoya経由で使う

---

## 1.各PFにアイテムをセット
がんばってください。  
指定するアイテム名については  
PF識別子(_iOSとか_Androidとか)を  
つけとくと管理が楽。

---

## 2.PurchaseSettings.csのURL調整
**PurchaseSettings.cs**  

```C#
public const string PURCHASE_URL_READY = "https://httpbin.org/get";
public const string PURCHASE_CONNECTIONID_READY_PREFIX = "purchase_ready_";

public const string PURCHASE_URL_TICKET = "https://httpbin.org/post";
public const string PURCHASE_CONNECTIONID_TICKET_PREFIX = "purchase_start_";

public const string PURCHASE_URL_PURCHASE = "https://httpbin.org/post";
public const string PURCHASE_CONNECTIONID_PURCHASE_PREFIX = "purchase_succeeded_";

public const string PURCHASE_URL_PAID = "https://httpbin.org/post";
public const string PURCHASE_CONNECTIONID_PAID_PREFIX = "purchase_paid_";

public const string PURCHASE_URL_CANCEL = "https://httpbin.org/post";
public const string PURCHASE_CONNECTIONID_CANCEL_PREFIX = "purchase_cancelled_";
```


---

## 3.アイテムを売る画面を作る

がんばってくれ。

---

## 4.Autoya経由で使う
次の3段階がある。

* 購入機構準備待ち
* 購入
* 購入後処理

(C/S全体のフローは後で)

+++

### 購入機構準備待ち
 
```C#
IEnumerator Start () {
	while (!Autoya.Purchase_IsReady()) {
		Debug.Log("log:" + Autoya.Auth_IsAuthenticated());
		Debug.Log("puc:" + Autoya.Purchase_IsReady());
		yield return null;
	}
```
@[2]

Autoya.Purchase_IsReady()が  
trueを返せば、準備完了。

初期化、起動時購入可能アイテムの取得は  
Autoyaのほうで完備されている。

+++


### 購入

クライアント中のどこでも、次のコード一発。

```C#
Autoya.Purchase(
	purchaseId, 
	"100_gold_coins", 
	pId => {
		Debug.Log("succeeded to purchase. id:" + pId);
	}, 
	(pId, err, reason, autoyaStatus) => {
		if (autoyaStatus.isAuthFailed) {
			Debug.LogError("failed to auth.");
			return;
		} 
		if (autoyaStatus.inMaintenance) {
			Debug.LogError("failed, service is under maintenance.");
			return;
		}
		Debug.LogError("failed to purchase, id:" + pId + " err:" + err + " reason:" + reason);
	}
);
```

ちょっとでかいので解説していく。

+++

```C#
Autoya.Purchase(
	purchaseId, 
	"100_gold_coins", 
	pId => {
		Debug.Log("succeeded to purchase. id:" + pId);
	}, 
	(pId, err, reason, autoyaStatus) => {
		if (autoyaStatus.isAuthFailed) {
			Debug.LogError("failed to auth.");
			return;
		} 
		if (autoyaStatus.inMaintenance) {
			Debug.LogError("failed, service is under maintenance.");
			return;
		}
		Debug.LogError("failed to purchase, id:" + pId + " err:" + err + " reason:" + reason);
	}
);
```
@[1](課金開始)
@[2](課金ごとに好きなidをつけられる。)
@[3](購入対象の名前をセット)
+++

```C#
Autoya.Purchase(
	purchaseId, 
	"100_gold_coins", 
	pId => {
		Debug.Log("succeeded to purchase. id:" + pId);
	}, 
	(pId, err, reason, autoyaStatus) => {
		if (autoyaStatus.isAuthFailed) {
			Debug.LogError("failed to auth.");
			return;
		} 
		if (autoyaStatus.inMaintenance) {
			Debug.LogError("failed, service is under maintenance.");
			return;
		}
		Debug.LogError("failed to purchase, id:" + pId + " err:" + err + " reason:" + reason);
	}
);
```

@[4-6]
アイテム付与後のコールバック

* サーバ最新のアイテム所持情報問い合わせ
* 終わったらローディングを停める とか想定

+++

```C#
Autoya.Purchase(
	purchaseId, 
	"100_gold_coins", 
	pId => {
		Debug.Log("succeeded to purchase. id:" + pId);
	}, 
	(pId, err, reason, autoyaStatus) => {
		if (autoyaStatus.isAuthFailed) {
			Debug.LogError("failed to auth.");
			return;
		} 
		if (autoyaStatus.inMaintenance) {
			Debug.LogError("failed, service is under maintenance.");
			return;
		}
		Debug.LogError("failed to purchase, id:" + pId + " err:" + err + " reason:" + reason);
	}
);
```
@[7-17]
失敗後のコールバック

課金が失敗した時に呼ばれる。  
ユーザーに理由を表示したりすると良い。

+++

## 購入後処理

アイテム付与後のコールバック

* サーバ最新のアイテム所持情報問い合わせ
* 終わったらローディングを停める とか想定

ここのこと。  
要はユーザーへと購入後の情報を見えるように  
しましょうねという処理を書く必要がある。 


---

ここまでで購入と配布までが終わり。

というのはクライアント側だけ見た場合で、  
実際にはサーバとのやりとりが一杯ある。  

つまりサーバが動いてないといけない。

---
## C/S全体のフロー


```
1. 購入機構初期化+リクエスト(クライアント)
2. ☆購入機構準備待ち(クライアント)
3. 購入可能アイテム一覧配布(サーバ)
4. ☆購入(クライアント)
5. 購入チケットリクエスト(クライアント)
6. 購入チケット記録(サーバ)
7. 購入チケットレスポンス(サーバ)
8. 購入確認リクエスト(クライアント)
9. 購入確認処理(サーバ)
10. 購入確認レスポンス(サーバ)
11. 購入後事前処理(クライアント)
12. ☆購入後処理(クライアント)
```

の、えーっと、全12段階に分かれる。

+++

![seq](doc/RemotePurchase.png)

+++ 

順に。

---

## 1. 購入機構初期化+リクエスト(クライアント)

クライアントが起動すると、サーバに  
「このユーザーに提示するアイテム一覧くれ」  
ってリクエストがいく。

+++

課金機構の初期化には、「どんな商品があるか」  
プラットフォームと確認する必要がある。  

アイテム情報をサーバから受け取れると、  
柔軟性が出る。


---
## 2. ☆購入機構準備待ち(クライアント)

クライアントは準備完了まで待つ。

---


## 3. 購入可能アイテム一覧配布(サーバ)

サーバはリクエストを受け取って、  
アイテム一覧を返す。  

クライアントは一覧を使って課金機構を初期化。  
準備完了となる。

+++

サーバが返すべき購入可能アイテム一覧は  
[ProductInfos](https://github.com/sassembla/Autoya/blob/master/Assets/Autoya/Purchase/PurchaseRouter.cs#L36)型

+++

json例  

```json
{
	"productInfos": [{
		"productId": "100_gold_coins",
		"platformProductId": "100_gold_coins_iOS",
		"isAvailableToThisPlayer": true,
		"info": "one hundled of coins."
	}, {
		"productId": "1000_gold_coins",
		"platformProductId": "1000_gold_coins_iOS",
		"isAvailableToThisPlayer": true,
		"info": "one ton of coins."
	}]
}
```

+++

OverridePoint.csを書き換えて、  
サーバから受け取ったアイテムを使うように  
変更。  

[書き換えてね](https://github.com/sassembla/Autoya/blob/7e69c19da3af364b03ff2a49dcc1d3f512af17b5/Assets/Autoya/Backyard/OverridePoints.cs#L204)

---

## 4. ☆購入(クライアント)

買いたいものを指定して、購入処理を開始。

---

## 5. 購入チケットリクエスト(クライアント)
購入情報をサーバに生成してもらうため、  
リクエストを送る。

---

## 6. 購入チケット記録(サーバ)

サーバはリクエストを受け取り、  
購入チケットを生成。購入を記録開始。

---

## 7. 購入チケットレスポンス(サーバ)

ユーザーに購入を許可する場合は  
200とチケットを返す。  

それ以外ならエラーを返す。

+++

チケットの要素は、

* GUID
* productId
* 日時
* もちろんuserId
* 状態(new, cancelled, done)

など。

+++


クライアントに返すチケットの情報は  
GUIDだけでOK。

どんな何を買おうとしてるか、  
ここでサーバでしっかり保存しておくと  
かなりいい感じになる。

+++

クライアント側は、200とチケットを  
受け取ったら処理継続、  

それ以外ならFailハンドラで停まる。

---

7 購入チケットレスポンス(サーバ)  

と  

8 購入確認リクエスト(クライアント)  
には隙間があって、  

ここでプラットフォームの  
「買う? Y/N」画面が出る。  

買うなら継続、買わないなら中止。

+++

図にはないけれど、  
購入キャンセルの場合、  
キャンセルの通信がサーバへと飛び、  
処理はFailで中止される。

---

## 8. 購入確認リクエスト(クライアント)

購入処理が終わったので、  
チケットとレシートをサーバへと送る。

---

## 9. 購入確認処理(サーバ)

チケットは本物か？ 記録と照合。  
レシートは本物か？ PFに問い合わせたり。  
本物だとして過去に重複したものはないか？  
[まとめてある](http://sassembla.github.io/Public/2016:06:03%2018-10-20/2016:06:03%2018-10-20.html)


etc.

+++

Androidの場合はpayloadを  
入れればいいんだけど、  
Unity IAPでどう入るのかが未確認。  


---

## 10. 購入確認レスポンス(サーバ)

チェックがOKだったら、200を返す。  
それ以外ならそれ以外を。

---

## 11. 購入後事前処理(クライアント)

200が来た場合は、PFに問い合わせて  
権利の解決を行う。

それ以外の場合は、独自に動作を書く。

+++

ここはサーバ側チェックによって  
色々実装を変えるべき場所で、  

200以外で来た場合このコードならこう、  
とかを書く必要がある。


---

## 12. ☆購入後処理(クライアント)

というわけでここまでで一切の契約処理は  
完了した状態で、コールバックが呼ばれる。

---

## イレギュラーケースについて

ある。そりゃもう滅茶苦茶ある。  

なんたって通信が多い上に、  
状態を保持している地点が3箇所も存在する。

しかもクライアントはまあ落ちる。

+++

## クライアントが落ちると？

大きく3つ、落ちパターンがある。  

1. PFとの契約が発生する前に落ちるか、  
2. PFとの契約が発生した後解決前に落ちるか、  
3. PFとの契約が解決した後に落ちるか。

+++

## 1. 契約発生前

チケットが無駄になるだけで済む。  
サーバは気にせず、クライアントが来たら  
新しいチケットを生成してあげるといい。  

+++

## 3. 契約解決後
さきに3を紹介。  

こちらは、すでに契約解決がなされているので  
問題ない。

むしろなんで落ちたし。

+++
## 2. 契約発生後解決前
問題のところ。  
ここで落ちるパターンは、これまた2つある。  

1. アイテム配布前
2. アイテム配布後

---


## 2-1 アイテム配布前

9の前、8の最中に落ちたらどうなるか。

購入は完了していて契約が残っていて、  
アプリが落ちた、と。

+++

ストアを初期化する時、  
未完了の契約がPFから現れて、  
クライアントからサーバへと送付される。

+++


![seq2](doc/RemotePurchase2.png)


+++

通常のケースとの違いとしては、


* そもそもURLが違う
* チケットが送られてこない

あたり。

+++

チケットは送られてこないので、  
レシートの検証だけして、  
200を返せばいい、、のか？  

まあ次を見てみよう。

---



## 2-2 アイテム配布後

10のあと、11完了までに落ちたらどうなるか。  

アイテム配布後、契約が残った状態、になる。

+++

ストアを初期化する時、  
未完了の契約がPFから現れて、  
クライアントからサーバへと送付される。

+++

![seq3](doc/RemotePurchase3.png)

+++

通常のケースとの違いとしては、


* そもそもURLが違う
* チケットが送られてこない

あたり。そして、

+++

2-1との違いが本当に一箇所だけあって！  
もうアイテム付与されてるんだよね(赤字)。  

というわけで、200を返すが、別にアイテムを  
さらに付与はしない、という感じになる。

---

2-1,2-2のケースは、同じルートで  
状況の異なる課金処理が発生する。  

2-1はまだアイテム配布してない。
2-2はもう配布が済んでる。  

差はどこで見る？

+++

1. 未完了Tiketの有無
2. レシート検証時色々チェック

+++

## 1. 未完了Tiketの有無

あれ、お客さん未済のチケットないんすか？  
おかしいっすね〜〜

or

チケットとレシート内容が違うっすね〜〜

などで対応。

+++


## 2. レシート検証時色々チェック

頑張ってチケットまで偽造して来る場合、  
[まとめてある](http://sassembla.github.io/Public/2016:06:03%2018-10-20/2016:06:03%2018-10-20.html)


---


ちなみにこの2-1, 2-2の課金ルートは  
なんというか不正にも使われやすい。  
というか使いやすい。

あんまりにこれが多い人は面白い人なので  
**優しく**してあげよう。
