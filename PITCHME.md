全全全全全全全全全全全全全全全全全全全全  
全角でだいたい20文字/行

# local purchaseについて

---

## って何？

UnityIAPの機構を使った課金処理。

ゲームサーバと一切通信せずに  
課金処理を実装できる。  

(もちろんプラットフォームとは通信する。)

--- 

## メリット
* サーバがいらない子

---

## デメリット
* アップデートでしか商品内容を更新できない
* ハックされ放題++
* 防ぐべき不正はまだある

---

## 準備と動作のフロー

1. 各プラットフォームにアイテムをセット
1. エディタでRVObfuscatorに情報をセット
1. 商品情報をPurchaseSettings.csに記述
1. アイテムを売る画面を作る
1. LocalPurchaseRouterをnewして使う

---

## 1.各PFにアイテムをセット
がんばってください。  
指定するアイテム名については  
PF識別子(_iOSとか_Androidとか)を  
つけとくと管理が楽。

具体例は3.あたりを。

---

## 2.RVObfに情報をセット
[Unity Documentに情報があるので](https://docs.unity3d.com/Manual/UnityIAPValidatingReceipts.html)  

がんばってください。

この工程で暗号化コードが生成される。

---

## 3.商品情報をP~.csに記述
**PurchaseSettings.cs**  
ここに書いたアイテムだけが売買される。

```C#
/*
	immutable purchasable item infos.
*/
public static readonly ProductInfos IMMUTABLE_PURCHASE_ITEM_INFOS = new ProductInfos {
	productInfos = new ProductInfo[] {
		new ProductInfo("100_gold_coins", "100_gold_coins_iOS", true, "one hundled of coins."),
		new ProductInfo("1000_gold_coins", "1000_gold_coins_iOS", true, "one ton of coins."),
		new ProductInfo("10000_gold_coins", "10000_gold_coins_iOS", false, "ten tons of coins."),// this product setting is example of not allow to buy for this player, disable to buy but need to be displayed.
	}
};
```

---

+++

iOS/Androidの両方で出す場合：


```C#
/*
	immutable purchasable item infos.
*/
public static readonly ProductInfos IMMUTABLE_PURCHASE_ITEM_INFOS = new ProductInfos {
	productInfos = new ProductInfo[] {
#if UNITY_IOS
		new ProductInfo("100_gold_coins", "100_gold_coins_iOS", true, "one hundled of coins."),
#elif UNITY_ANDROID
		new ProductInfo("100_gold_coins", "100_gold_coins_Android", true, "one hundled of coins."),
#endif

```		
とかやっとくと楽。


---

## 4.アイテムを売る画面を作る

がんばってくれ。

## 5. LocPurcRoutをnewして使う

さてここからコードの話になる。  
ここまでくれば、アイテムを買う場所、買う内容、その暗号化などが揃っている。  

で、実際にゲーム中での購買処理をどのようにセットしていくかを書く。

**LocalPurchaseRouterの初期化**

```C#
localPurchaseRouter = new LocalPurchaseRouter(
	PurchaseSettings.IMMUTABLE_PURCHASE_ITEM_INFOS.productInfos,
	() => {
		Debug.Log("ready purchase.");
	}, 
	(err, reason) => {
		Debug.LogError("failed to ready purchase. error:" + err + " reason:" + reason);
	}, 
	alreadyPurchasedProductId => {
		/*
			this action will be called when 
				the IAP feature found non-completed purchase record
					&&
				the validate result of that is OK.

			need to deploy product to user.
		 */
		
		// deploy purchased product to user here.
	}
);
```
解説してくよ。

+++


```C#
localPurchaseRouter = new LocalPurchaseRouter(
	PurchaseSettings.IMMUTABLE_PURCHASE_ITEM_INFOS.productInfos,
	() => {
		Debug.Log("ready purchase.");
	}, 
	(err, reason) => {
		Debug.LogError("failed to ready purchase. error:" + err + " reason:" + reason);
	}, 
	alreadyPurchasedProductId => {
		/*
			this action will be called when 
				the IAP feature found non-completed purchase record
					&&
				the validate result of that is OK.

			need to deploy product to user.
		 */
		
		// deploy purchased product to user here.
	}
);
```
@[0]アプリケーションの起動時にインスタンスを生成

+++


```C#
localPurchaseRouter = new LocalPurchaseRouter(
	PurchaseSettings.IMMUTABLE_PURCHASE_ITEM_INFOS.productInfos,
	() => {
		Debug.Log("ready purchase.");
	}, 
	(err, reason) => {
		Debug.LogError("failed to ready purchase. error:" + err + " reason:" + reason);
	}, 
	alreadyPurchasedProductId => {
		/*
			this action will be called when 
				the IAP feature found non-completed purchase record
					&&
				the validate result of that is OK.

			need to deploy product to user.
		 */
		
		// deploy purchased product to user here.
	}
);
```
@[1]設定ファイルに書かれているプロダクトを買えるようにする。



