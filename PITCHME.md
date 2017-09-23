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
がんばってください
---

## 2.RVObfに情報をセット
[Unity Doc](https://docs.unity3d.com/Manual/UnityIAPValidatingReceipts.html)  
がんばってください

---

## 3.商品情報をP~.csに記述
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


