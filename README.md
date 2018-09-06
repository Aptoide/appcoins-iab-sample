# AppCoins IAB
AppCoins IAB lets you sell digital content from inside Android apps.


## Architecture
AppCoinsWallet exposes a Android Service which your app should bind with. Once bound to AppCoinsWallet service, your app can start communicating over IPC using an AIDL inteface.

## Google Play IAB to AppCoins IAB Migration Guide


### AIDL

Like Google Play IAB, AppCoins IAB uses a AIDL file in order to communicate with AppCoins service. The package for your AIDL must be **com.appcoins.billing** instead of **com.android.vending.billing**. Both AppCoins and Google AIDL files are identical, but you need to rename **InAppBillingService.aild** to **AppcoinsBilling.aidl**.

![Migration](docs/aidl-migration.png)

### Permissions

Your app needs a permission to allow it to perform billing actions with AppCoins IAB. The permission is declared in the **AndroidManifest.xml** file of your app. Since Google Play IAB already declares a permission with name **com.android.vending.BILLING**, you should rename it to **com.appcoins.BILLING**.


**Google Play IAB**

	<uses-permission android:name="com.android.vending.BILLING" />

**AppCoins IAB**

	<uses-permission android:name="com.appcoins.BILLING" />

### Service Connection

In order to communicate with AppCoins IAB, your app must bind to a service in the same way as in Google Play IAB. Google Play IAB Intent action and package must be updated from **com.android.vending.billing.InAppBillingService.BIND** and **com.android.vending** to **com.appcoins.wallet.iab.action.BIND** and **com.appcoins.wallet**, respectively.


**Google IAB Service Intent**

	Intent serviceIntent = new Intent("com.android.vending.billing.InAppBillingService.BIND");
	serviceIntent.setPackage("com.android.vending");

**AppCoins IAB Service Intent**

	Intent serviceIntent = new Intent("com.appcoins.wallet.iab.action.BIND");
	serviceIntent.setPackage("com.appcoins.wallet");

**Payment Intent**	

When calling [getPaymentIntent method](https://github.com/Aptoide/appcoins-iab-sample/blob/feature/APPC-541-documentation/app/src/appcoinsiab/aidl/com/appcoins/billing/AppcoinsBilling.aidl#L96), you must send your developer's Ethereum wallet address. This can be done by using the helper provided [here](app/src/main/java/com/aptoide/iabexample/util/PayloadHelper.java).

```java
...
String payload = PayloadHelper.buildIntentPayload("developer_ethereum_address","developer_payload")
Bundle buyIntentBundle = mService.getBuyIntent(3, mContext.getPackageName(), sku, itemType, payload);
```
In order to get your developer's Ethereum wallet address, go to [BDS Back Office -> Wallet -> Deposit APPC](https://blockchainds.com/wallet/depositAppc) and click on "copy to clipboard button".


### BDS Public Key

Just like Google Play IAB, AppCoins IAB also exposes a public key. You should use AppCoins IAB public key to verify your purchases. It works exactly like Google Play IAB key, so you just need to replace one for the other.

To find your BDS public key, you should be registered in BDS Back Office. You should head to [Add App section](https://developers.blockchainds.com/myApps/upload/publicKey) of the BO and add the package name of your app (e.g. com.appcoins.wallet). Please make sure it's the same package name of the APK you'll upload later. Once you insert the package name and click "Next", you should already see your public key.

### Purchase Broadcast

Google Play IAB broadcasts and Intent with action **com.android.vending.billing.PURCHASES_UPDATED**. Since AppCoins IAB does not implement this, any code related with listening to that Intent can be removed.

### Wallet Support

In order to use AppCoins, you will need a compliant wallet installed. The following snippets shows how you can prompt the user to install the AppCoins BDS Wallet when not present.

To do that, first validate if an EIP 681 compatible wallet is installed.
```
if (!hasWalletInstalled(this)) {
    showWalletInstallDialog(this,
    this.getString(R.string.install_wallet_from_iab));
}
```
**Please note** that other EIP 681 compatible wallets can be available on the users device and in that case no dialog is prompted and the bind with billing service fails. In this case the installed wallet is not compatible with the BDS billing system and also in that case the dialog should be prompted suggesting to the user to install the compatible wallet.

To check if the wallet is installed, build the EIP 681 compatible URI and validate if there is an application that can handle the intent.
It can be done with the following.
```
public static boolean hasWalletInstalled(Context context) {
    Intent intent = new Intent(Intent.ACTION_VIEW);
    Uri uri = Uri.parse("ethereum:");
    intent.setData(uri);

    return hasHandlerAvailable(intent, context);
  }
```

The list of applications that can listen to the intent with the EIP 681 compatible URI is checked to be empty. If it is empty, it means that no application to handle the purchase flow is available.
```
public static boolean hasHandlerAvailable(Intent intent, Context context) {
    PackageManager manager = context.getPackageManager();
    List<ResolveInfo> infos = manager.queryIntentActivities(intent, 0);
    return !infos.isEmpty();
  }
```

Showing a basic dialog that informs the user that a compatible wallet needs to be installed can be done with following method. Notice that the dialog generated by this method is basic and provides the minimum required information. The creation of a new dialog is advised, to be compatible with the layout of your application.

```
private static void showWalletInstallDialog(Context context, String message) {
    AlertDialog.Builder builder;
    builder = new AlertDialog.Builder(context);
    builder.setTitle(R.string.wallet_missing)
        .setMessage(message)
        .setPositiveButton(R.string.install, (dialog, which) -> gotoStore(context))
        .setNegativeButton(R.string.skip, (DialogInterface dialog, int which) -> dialog.dismiss())
        .setIcon(android.R.drawable.ic_dialog_alert)
        .show();
}
```
If you follow with the use of the code above the resources below are required.
```
  <string name="install">INSTALL</string>
  <string name="skip">SKIP</string>
  <string name="install_wallet_from_iab">To pay for in-app items, you need to install the AppCoins BDS Wallet.</string>
  <string name="wallet_missing">AppCoins Wallet Missing</string>
```

Finally, the following method is the one used to redirect the used to AppCoins BDS Wallet store page. 
```
@NonNull private static void gotoStore(Context activity) {
    String appPackageName = "com.appcoins.wallet";
    try {
      activity.startActivity(
          new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + appPackageName)));
    } catch (android.content.ActivityNotFoundException anfe) {
      activity.startActivity(new Intent(Intent.ACTION_VIEW,
          Uri.parse("https://play.google.com/store/apps/details?id=" + appPackageName)));
    }
  }
```
### Testing

For testing purposes you can use the following data to test the billing of you application.
* **applicationId**: *com.appcoins.sample*
* **APPCOINS_IAB_KEY**: *MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyEt94j9rt0UvpkZ2jPMZZ16yUrBOtjpIQCWi/F3HN0+iwSAeEJyDw7xIKfNTEc0msm+m6ud1kJpLK3oCsK61syZ8bYQlNZkUxTaWNof1nMnbw3Xu5nuYMuowmzDqNMWg5jNooy6oxwIgVcdvbyGi5RIlxqbo2vSAwpbAAZE2HbUrysKhLME7IOrdRR8MQbSbKEy/9MtfKz0uZCJGi9h+dQb0b69H7Yo+/BN/ayBSJzOPlaqmiHK5lZsnZhK+ixpB883fr+PgSczU7qGoktqoe6Fs+nhk9bLElljCs5ZIl9/NmOSteipkbplhqLY7KwapDmhrtBgrTetmnW9PU/eCWQIDAQAB* 

# Known Issues

* AppCoins IAB is not compliant with [Google Play IAB v5](https://developer.android.com/google/play/billing/versions.html). Calls to **getBuyIntentToReplaceSkus** method will always fail.
