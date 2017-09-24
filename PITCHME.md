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

---

## デメリット
* 管理が面倒
* ハックされないわけではない

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

1. 購入機構初期化+リクエスト(クライアント)
1. ☆購入機構準備待ち(クライアント)
1. 購入可能アイテム一覧配布(サーバ)
1. 購入チケットリクエスト(クライアント)
1. 購入チケット配布(サーバ)
1. ☆購入(クライアント)
1. 購入確認処理(サーバ)
1. 購入確認レスポンス(サーバ)
1. ☆購入後処理(クライアント)

の、えーっと、全9段階に分かれる。

+++

1. 購入機構初期化+リクエスト(クライアント)
1. **☆購入機構準備待ち(クライアント)**
1. 購入可能アイテム一覧配布(サーバ)
1. 購入チケットリクエスト(クライアント)
1. 購入チケット配布(サーバ)
1. **☆購入(クライアント)**
1. 購入確認処理(サーバ)
1. 購入確認レスポンス(サーバ)
1. **☆購入後処理(クライアント)**

@[2]
@[6]
@[9]

☆がついてるのは**クライアント側で明示的に**  
何かする必要があるところ。

+++

1. 購入機構初期化+リクエスト(クライアント)
1. ☆購入機構準備待ち(クライアント)
1. **購入可能アイテム一覧配布(サーバ)**
1. 購入チケットリクエスト(クライアント)
1. **購入チケット配布(サーバ)**
1. ☆購入(クライアント)
1. **購入確認処理(サーバ)**
1. **購入確認レスポンス(サーバ)**
1. ☆購入後処理(クライアント)

@[3]
@[5]
@[7-8]


サーバ側処理。


+++

シーケンス図だとこんな感じ。



全全全全全全全全全全全全全全全全全全全全  
全角でだいたい20文字/行