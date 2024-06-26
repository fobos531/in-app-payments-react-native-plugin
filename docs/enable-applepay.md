# Enable Apple Pay with the React Native Plugin for In-App Payments SDK

This guide walks you through the process of enabling the Apple Pay digital wallet
for an app that uses the **React Native plugin for the Square [In-App Payments SDK]**. See the [React Native plugin Technical Reference](reference.md)
for more detailed information about the [Apple Pay] methods available.

**Apple Pay** can only be used on iOS devices. You must [set up Apple Pay requirements] and [enable Apple Pay] in the `ios/myRNInAppPaymentsSample` React Native project before using Apple Pay in this app.

Alternatively, if you're using Expo, you can utilize the built-in config plugin to automatically configure the Apple Pay Merchant ID in your iOS project.

Add the following to your `app.json`:

```json
{
  "expo": {
    ...
    "plugins": [
      [
        "react-native-square-in-app-payments",
        {
          "merchantIdentifier": string | string [],
        }
      ]
    ],
  }
}
```

## Before you start

- If you haven't already created a React Native project with In-App Payments SDK, use the [Getting Started with the React Native Plugin for In-App Payments SDK](get-started.md) to
  set up a React Native project .

## Process overview

- [Enable Apple Pay with the React Native Plugin for In-App Payments SDK](#enable-apple-pay-with-the-react-native-plugin-for-in-app-payments-sdk)
  - [Before you start](#before-you-start)
  - [Process overview](#process-overview)
  - [Step 1: Upload an Apple Pay certificate to the Square Developer Portal](#step-1-upload-an-apple-pay-certificate-to-the-square-developer-portal)
  - [Step 2: Initialize Apple Pay and verify Apple Pay support](#step-2-initialize-apple-pay-and-verify-apple-pay-support)
  - [Step 3: Authorize payment with Apple Pay and get payment authorization result](#step-3-authorize-payment-with-apple-pay-and-get-payment-authorization-result)

## Step 1: Upload an Apple Pay certificate to the Square Developer Portal

To [add an Apple Pay payment processing certificate] in the **Apple Developer Portal**,
you must first obtain a Certificate Signing
Request (CSR) from Square. The [Square Application Dashboard]
provides a CSR file that can be submitted to Apple:

1. On the application dashboard, select the application you created for your React Native project.
1. Click the **Apple Pay** tab.
1. Click **Add Certificate** and follow the instructions to generate an Apple
   Pay certificate from Apple and upload it to your Square account.

## Step 2: Initialize Apple Pay and verify Apple Pay support

Add code to initialize Apple Pay in your application State class.

```javascript
import {
  ...
  Platform,
} from 'react-native';
import {
  SQIPApplePay,
} from 'react-native-square-in-app-payments';

export default class App extends Component {
  ...
  async componentDidMount() {
    ...
    let digitalWalletEnabled = false;
    if (Platform.OS === 'ios') {
      await SQIPApplePay.initializeApplePay('REPLACE_WITH_APPLE_PAY_MERCHANT_ID');
      digitalWalletEnabled = await SQIPApplePay.canUseApplePay();
    } else if (Platform.OS === 'android') {
      ...
    }

    this.setState({
      canUseDigitalWallet: digitalWalletEnabled,
    });
  }
  ...
}
```

- Replace `REPLACE_WITH_APPLE_PAY_MERCHANT_ID` in this example with a valid apple pay merchant ID.

## Step 3: Authorize payment with Apple Pay and get payment authorization result

Open the Apple Pay sheet and request the user's authorization of the payment. On authorization, a
payment nonce is returned in `onApplePayNonceRequestSuccess`.

```javascript

export default class App extends Component {
  constructor() {
    super();
    ...
    // bind 'this' to the method's context
    this.onStartDigitalWallet = this.onStartDigitalWallet.bind(this);
  }

  ...
  /**
   * Callback when successfully get the card nonce details for processig
   * apple pay sheet is still open and waiting for processing card nonce details
   * @param cardDetails
   */
  async onApplePayNonceRequestSuccess(cardDetails) {
    try {
      // take payment with the card nonce details
      // you can take a charge
      // await chargeCard(cardDetails);

      // you must call completeApplePayAuthorization to close apple pay sheet
      await SQIPApplePay.completeApplePayAuthorization(true);
    } catch (ex) {
      // handle card nonce processing failure

      // you must call completeApplePayAuthorization to close apple pay sheet
      await SQIPApplePay.completeApplePayAuthorization(false, ex.message);
    }
  }

  /**
   * Callback when failed to get the card nonce
   * apple pay sheet is still open and waiting for processing error information
   */
  async onApplePayNonceRequestFailure(errorInfo) {
    // handle this error before close the apple pay sheet

    // you must call completeApplePayAuthorization to close apple pay sheet
    await SQIPApplePay.completeApplePayAuthorization(false, errorInfo.message);
  }

  /**
   * Callback when the apple pay sheet is closed after
   * call completeApplePayAuthorization or user tap to close apple pay sheet manually
   */
  onApplePayEntryComplete() {
    // handle the apple pay sheet closed event
  }

  /**
   * An event listener to start digital wallet flow
   */
  async onStartDigitalWallet() {
    if (Platform.OS === 'ios') {
      const applePayConfig = {
        price: '1.00',
        summaryLabel: 'Test Item',
        countryCode: 'US',
        currencyCode: 'USD',
        paymentType: SQIPApplePay.PaymentTypeFinal,
      };
      try {
        await SQIPApplePay.requestApplePayNonce(
          applePayConfig,
          this.onApplePayNonceRequestSuccess,
          this.onApplePayNonceRequestFailure,
          this.onApplePayEntryComplete,
        );
      } catch (ex) {
        // Handle InAppPaymentsException
      }
    } else if (Platform.OS === 'android') {
      ...
    }
  }

  render() {
    return (
      <View style={styles.container}>
        ...
        <Button
          onPress={this.onStartDigitalWallet}
          title="Start Digital Wallet"
          disabled={!this.state.canUseDigitalWallet}
        />
      </View>
    );
  }
}
```

---

**Note:** the `chargeCard` method in this example shows a typical REST request on a backend process
that uses the **Payments API** to take a payment with the supplied nonce.
See [BackendQuickStart Sample] to learn about building an app that processes payment nonces on a server.

[//]: # "Link anchor definitions"
[in-app payments sdk]: https://developer.squareup.com/docs/in-app-payments-sdk/what-it-does
[root readme]: ../README.md
[apple pay]: https://developer.apple.com/documentation/passkit/apple_pay
[add an apple pay payment processing certificate]: https://help.apple.com/developer-account/#/devb2e62b839?sub=devf31990e3f
[square application dashboard]: https://connect.squareup.com/apps/
[set up apple pay requirements]: https://developer.apple.com/documentation/passkit/apple_pay/setting_up_apple_pay_requirements
[enable apple pay]: https://help.apple.com/xcode/mac/9.3/#/deva43983eb7?sub=dev44ce8ef13
[backendquickstart sample]: https://github.com/square/in-app-payments-server-quickstart
