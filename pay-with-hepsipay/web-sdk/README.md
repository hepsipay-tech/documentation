# Hepsipay SDK

## Usage

```html

<body>
  <!-- ... -->
  <!-- Latest version -->
  <script src='https://images.hepsiburada.net/hepsipay/packages/pf/hepsipay-latest.min.js'></script>

  <!-- Specific version (recommended) -->
  <script src='https://images.hepsiburada.net/hepsipay/packages/pf/hepsipay-0.3.1.min.js'></script>
</body>

```

## Methods

1. **init**


```js

HepsipaySdk.init(initOptions)

```

*init* method needs an argument which is a JavaScript object and it is consists of four properties.

- frameUrl
- frameDivId
- paymentStatusCallback
- frameFailureCallback
- onPaymentFailureLogCallback
- onPaymentSuccessCallback
- maxHeight
- disableDynamicHeight
- showFramePaymentButton
- useApiCallOnSuccessCallback
- overrideFontFamily
- hideHeader

```js
const arg = {
  frameUrl: 'frameUrl',
  frameDivId: 'divIdForShowingFrame',
  /* Handle status of checkout button */
  paymentStatusCallback: callback(isPaymentAllowed),
  /* Handle frame related errors */
  frameFailureCallback: callback(),
 /* Handle payment failures */
  onPaymentFailureLogCallback: callback(payload: {userMessage: string, userMessageTitle: string, message: string, messageCode: string, isBankError: boolean}),
  /* Handle successful payment */
  onPaymentSuccessCallback: callback(payload: {MerchantOrderNumber: string, merchantCallBackUrl: string, token: string}),
  /* Max height value for frame div, default is set to 800px */
  maxHeight: "800",
  /* Resize frame wether is dynamic or static, default is set to false */
  disableDynamicHeight: false,
  /* Show in-frame payment button, default is set to false */
  showFramePaymentButton: false,
  /**
  * In the BE frame-init integration, the `MerchantCallbackUrl` address defined;
  * If `true`, the API call uses the POST request method (application/json).
  * If `false`, the form-post method (application/x-www-form-urlencoded) is used.
  * Default: false
  */
  useApiCallOnSuccessCallback: false,
  /**
  * Payload object for relevant payment types;
  * When "Kart İle Öde" selected
  * @param { amount: number, actualInstallmentNumber: number, displayInstallmentNumber: number, binNumber: string, paymentType: string }
  * When "Hepsipay Bakiyem İle Öde" selected
  * @param { amount: number, binNumber: string, paymentType: string }
  * When "Hızlı alışveriş kredisi ile öde" selected
  * @param { amount: number, paymentType: string }
  * When "Hepsifinans ile öde" selected
  * @param { amount: number, paymentType: string }
  */
  selectedPaymentInfoCallback: callback(payload),
  /* You can define the google font you want to use. Default: Inter */
  overrideFontFamily: "Inter",
  /* Allows hide/show control of the Hepsipay logo header area: Default: false */
  hideHeader: false,
}
```

*init* method uses two callbacks for checking payment status and handling frame failures.

```js

 const paymentStatusCallback = (isPaymentAllowed) => {
    if (isPaymentAllowed) {
      // Payment allowed logic by merchant
    } else {
     // Payment not allowed logic by merchant
    }
  }

  const frameFailureCallback = () => {
     // Merchant will manage
  }

  HepsipaySdk.init({frameUrl, frameDivId, paymentStatusCallback, frameFailureCallback})

```


It will initialize frame with relevant frame source url and create in a div which id is given id.

2. **initPayment**


```js

HepsipaySdk.initPayment()

```

It is called in click event of payment button in merchant page.

```js

  const onPayment = () => {
    HepsipaySdk.initPayment();
  };

```

It is a method that posts *message event* to the frame. Frame will listen and start the payment process with relevant message type in this case: *hp-start-payment*


```js

  const message = { messageType: 'hp-start-payment' };
  document
    .getElementById('hepsipayFrame')
    .contentWindow.postMessage(message, '*');

```
