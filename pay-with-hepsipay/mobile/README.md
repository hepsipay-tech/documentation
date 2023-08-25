<p align="center" style="margin-bottom: 0;padding-bottom: 0;"><img src="https://images.hepsiburada.net/assets/sardes/wallet/redesign/svg/logo_hepsipay.svg" alt="project-image"></p>

# Pay with Hepsipay

Hepsipay checkout deneyimi ve avantajları her yerde!

Bu dökümantasyon Hepsipay deneyimini JavaScript SDK çözümümüz olmadan kullanımına yöneliktir. 

JavaScript SDK'in kullanılması durumunda [Frame JS Events](#frame-js-event) bölümü ve sonrasına ihtiyaç duymayacaktır

## 🚀 Demo

[https://pf-ui-pwh-qa.hepsipay.com/?token=YOUR\_MERCHANT\_SESSION\_TOKEN](https://pf-ui-pwh-qa.hepsipay.com/?token=YOUR_MERCHANT_SESSION_TOKEN)

### Hepsipay Frame Ekran Görüntüleri:

- <img src="https://images.hepsiburada.net/hepsipay/img/demo/pwh-use-stored-card-in-co.jpg" alt="project-screenshot" width="300" height="auto/">
- <img src="https://images.hepsiburada.net/hepsipay/img/demo/pwh-use-wallet-in-co.jpg" alt="project-screenshot" width="300" height="auto/">
- <img src="https://images.hepsiburada.net/hepsipay/img/demo/pwh-use-wallet.jpg" alt="project-screenshot" width="300" height="auto/">

## 🧐 Özellikleri

Projenin bazı harika özellikleri:

*   Kartla öde
*   Hepsipay hesabımla öde
*   Alışveriş kredisi ile öde

## 🛠️ Kullanım/Kurulum:

1. Hepsipay Backend Entegrasyonu
Hepsipay UI kullanım/testleri öncesinde Backend entegrasyonu tamamlanmalıdır

```
<*!-- Hepsipay Backend dökümantasyonu ayrıca iletilir -->
```

2. Backend entegrasyonu tamamlandığında API'den aşağıdaki gibi response alınır;

```JSON 
{
    "RequestUrl": "https://{{HEPSIPAY_GATEWAY_DNS}}/hepsipayframe/init",
    "Response": {
        "FrameUrl": "https://pf-ui-pwh-qa.hepsipay.com/?token=MERCHANT_SESSION_TOKEN",
        "Token": "MERCHANT_SESSION_TOKEN",
        "Success": true,
        "MessageCode": "0000",
        "Message": "İşlem Başarıyla Gerçekleştirildi.",
        "UserMessageTitle": null,
        "UserMessage": "İşlem Başarıyla Gerçekleştirildi."
    }
}
```

3. Hepsipay frame çözümünü kendi checkout iframe ile kullanmak  
```html
<iframe id="hepsipayFrame" src="{{FrameUrl}}" height="450" width="100%" frameborder="0"></iframe>

<!--
iframe "height" tanımı için önerilen boyutlar;
mobil çözünürlükte: en az 430px
desktop çözünürlükte: en az 550px 
-->
```

4. Ödeme başarılı olduğu durumda `MerchantCallBackUrl` ile tanımlanan adrese `POST` methodu ile içinde form-data tipinde `token` input taşıyan değer ile birlikte gelinir;
```jsx
<form action="{MerchantCallBackUrl}" method="POST">
    <input type="hidden" name="token" value="{MERCHANT_SESSION_TOKEN}" />
    <input type="hidden" name="MerchantOrderNumber" value="{MERCHANT_ORDER_NUMBER}" />
</form>
```

# Frame JS Event
Hepsipay frame [4 farklı event](#messagetype-listesi) gönderir. Bunların tamamı, ihtiyaca bağlı entegre olunabilecek eventlerdir;

### Eventler, JavaScript şekilde kontrol edilebilir
```js
// Tüm hepsipay frame eventleri event.data.messageType şeklinde `messageType: string` değeri taşır 
window.addEventListener('message', handleMessageEvents);
/**
 * iOS ve Android için Webview oluşturulurken `HepsipayFrameCommunicator` adında bir sınıf Webview'e bind edilmeli.
 * Bu sınıf üzerinden "postMessage" yöntemiyle bu eventler kontrol edilebilir olacaktır.
 */


function handleMessageEvents(event) {
    const {messageType, ...data} = evet.data;
    if (messageType === ONE_OF_HEPSIPAY_MESSAGE_TYPE) {
        // Your business logic goes here
    } 
}

```
### messageType listesi;
#### - hp-payment-success
- ```event.data = { messageType: 'hp-payment-success' }```
- Müşteri ödeme sürecini 3Ds ve/veya non 3Ds ile başarıyla tamamladığı bildirilir
- *Bu event handle edildiği durumda frame'in kapatılması beklenir, yoksa event atıldıktan ön tanımlı bir süre kadar sonrasında [Kullanım/Kurulum 4. adımda](#-kullanımkurulum-) anlatılan aksiyon alınır 
#### - hp-restart-frame *(handle edilmesi önerilir)*
- `event.data = {messageType: 'hp-restart-frame'}`
- Session ile ilgili devam edilemeyecek kritik bir hata oluşmuştur. Bu durumda müşteri işlemine devam edemeyecektir.Yeniden token üretilip iframe yeniden render edilmelidir 
- Bu hata çoğunlukla üretilen SessionToken'ın artık geçerli olmadığı durumda gönderilir
#### - hp-payment-available-status
- `event.data = { messageType: 'hp-payment-available-status', isPaymentAllowed: boolean }`
- `isPaymentAllowed: true` Ödemeyi başlatabilir durumdadır
- `isPaymentAllowed: false` Ödeme başlatamayacak durumdadır. Çoğunlukla yeni kart form ekranı açtığı için atılır veya alışveriş kredisi sürecindedir
#### - hp-resize-frame
- `event.data = { messageType: 'hp-payment-available-status', height: number }`
- Bu event frame işgal ettiği yükseklik alanı değiştikçe atılır. 
- Burada gelen `height` değeri direkt olarak <iframe ... height={height}/> şeklinde kullanılarak frame'in yüksekliği dinamik kullanılması sağlanabilir
#### - hp-jwt-token
- `event.data = { messageType: 'hp-jwt-token', data: { token: "JWT_TOKEN" } }`
- Bu event hepsipay bakiyesini kullanarak ödeme yapmak isteyen kullanıcılardan Hepsipay şifreleri başarılı giriş yaptıktan sonra gönderilir
- Event'in atılması native iOS ve Android tarafında token'ın alıp storage üzerinde saklanması içindir. Bir sonraki Hepsipay webview açılırken, bu değer, aynı isimle tekrar Webview cookie üzerine yazılması içindir.
- Bu şekilde kullanıldığı zaman; müşterinin JWT token değer hâlâ geçerli olduğu sürece tekrar Hepsipay bakiyesi ile ödeme yapmak istediğinde yeniden şifre sorulmayacaktır.
