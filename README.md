# AppCoins IAB
AppCoins IAB lets you sell digital content from inside applications.


## Architecture
AppCoinsWallet exposes a Android Service which your application should bind with. Once bound to AppCoinsWallet service your application can start communicating over IPC using an AIDL inteface.

## Google Play IAB to AppCoins IAB Migration


### AIDL

Like Google Play IAB, AppCoins IAB uses a AIDL file in order to communicate with AppCoins service. The package for your AIDL must be **com.appcoins.billing** instead of **com.android.vending.billing**. Both AppCoins and Google AIDL files are identical, but you need to rename **InAppBillingService.aild** to **AppcoinsBilling.aidl**.

![Migration](docs/aidl-migration.png)

### Permissions

Your application needs a permission to allow it to perform billing actions with AppCoins IAB. The permission is declared in **AndroidManifest.xml** of your application. Google Play IAB already declares a permision with name **com.android.vending.BILLING** you should rename it to **com.appcoins.BILLING**.


**Google Play IAB**

	<uses-permission android:name="com.android.vending.BILLING" />

**AppCoins IAB**

	<uses-permission android:name="com.appcoins.BILLING" />

### Service Connection

In order to communicate with AppCoins IAB your application must bind to a service the same way Google Play IAB. Google Play IAB Intent action and package must be updated from **com.android.vending.billing.InAppBillingService.BIND** to **com.appcoins.wallet.iab.action.BIND** and from **com.android.vending** to **com.appcoins.wallet** respectively.


**Google IAB Service Intent**

	Intent serviceIntent = new Intent("com.android.vending.billing.InAppBillingService.BIND");
	serviceIntent.setPackage("com.android.vending");

**AppCoins IAB Service Intent**

	Intent serviceIntent = new Intent("com.appcoins.wallet.iab.action.BIND");
	serviceIntent.setPackage("com.appcoins.wallet");

**Payment Intent**	

When calling [getPaymentIntent method](https://github.com/Aptoide/appcoins-iab-sample/blob/feature/APPC-541-documentation/app/src/appcoinsiab/aidl/com/appcoins/billing/AppcoinsBilling.aidl#L96), you must send the developer's ethereum wallet address. This can be done by using the helper provided [here](app/src/main/java/com/aptoide/iabexample/util/PayloadHelper.java).

```java
...
String payload = PayloadHelper.buildIntentPayload("developer_ethereum_address","developer_payload")
Bundle buyIntentBundle = mService.getBuyIntent(3, mContext.getPackageName(), sku, itemType, payload);
```
In order to get the Developer Ethereum Address go to [BDS Back Office -> Wallet -> Deposit APPC](https://developers-dev.blockchainds.com/wallet/depositAppc) and click on "copy to clipboard button".


### AppCoins Public Key

Just like Google Play IAB, AppCoins IAB also exposes a public key. You should use AppCoins IAB public key to verify your purchases. It works exactly like Google Play IAB key so you just need to replace each other.

To find your AppCoins public key go to [BDS Back Office -> Manage Apps -> Apps List -> Open Your App](https://developers-dev.blockchainds.com/myApps/appsList). Scroll down to Monetisation card, create a product, refresh the page and click "get key" button.


### Purchase Broadcast

Google Play IAB broadcasts and Intent with action **com.android.vending.billing.PURCHASES_UPDATED**. AppCoins IAB does not do that therefore any code related with listening to that Intent can be removed.


# Known Issues


* AppCoins IAB is not compliant with [Google Play IAB v5](https://developer.android.com/google/play/billing/versions.html). Calls to **getBuyIntentToReplaceSkus** method will always fail.
