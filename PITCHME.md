# purchaseについて

---

## remoteとlocal
Autoyaには2種の課金機構が搭載されてる。

サーバありのやつ  
[remote](https://gitpitch.com/sassembla/aboutPurchase/remotePurchase#)  

[コード](https://github.com/sassembla/Autoya/blob/master/Assets/Autoya/Purchase/PurchaseRouter.cs)

サーバなしのやつ  
[local](https://gitpitch.com/sassembla/aboutPurchase/localPurchase#)

[コード](https://github.com/sassembla/Autoya/blob/master/Assets/Autoya/Purchase/LocalPurchaseRouter.cs)


+++

サーバあり版は認証とかがっつり絡んでいる。

サーバなしのやつはコード単体で動かせる。

どちらも、動くサンプルが[ここ](https://github.com/sassembla/Autoya/tree/master/Assets/AutoyaSample/3_Purchase)にあるんで、  
Autoyaを落としてPurchase.unityシーンを  
動かしてみるといいと思う。