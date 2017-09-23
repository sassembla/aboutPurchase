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

1. 各プラットフォームにアイテムをセット
1. URLをPurchaseSettings.csに記述
1. アイテムを売る画面を作る
1. Autoya経由で使う

---

## 1.各PFにアイテムをセット
がんばってください。  
指定するアイテム名については  
PF識別子(_iOSとか_Androidとか)を  
つけとくと管理が楽。

---

## 2.URLをPurchaseSettings.csに記述
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
次の2段階がある。

* 準備完了待ち
* 購入

(C/S全体のフローは後で)

+++

### 準備完了待ち
 
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

+++/Users/passepied/Desktop/remote purchaseについて.md

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

ここで、

* サーバ最新のアイテム所持情報問い合わせ
* 終わったらローディングを停める

などの処理を書く想定。

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
@[7-18]
失敗後のコールバック

課金が失敗した時に呼ばれる。  
ユーザーに理由を表示したりすると良い。


---
## C/S全体のフロー

実際のゲーム中での購買処理は、

* 準備完了待ち(クライアント)
* 購入可能アイテム一覧配布(サーバ)
* 購入チケット配布(サーバ)
* 購入(クライアント)
* 購入確認(サーバ)

の5段階に分かれる。

+++

## クライアント側：
* 準備完了待ち(クライアント)
* 購入(クライアント)


## サーバ側：
* 購入可能アイテム一覧配布(サーバ)
* 購入チケット配布(サーバ)
* 購入確認(サーバ)


+++

## 準備完了待ち(クライアント)



+++



全全全全全全全全全全全全全全全全全全全全  
全角でだいたい20文字/行