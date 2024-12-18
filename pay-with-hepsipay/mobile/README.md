<p align="center" style="margin-bottom: 0;padding-bottom: 0;"><img src="https://images.hepsiburada.net/assets/sardes/wallet/redesign/svg/logo_hepsipay.svg" alt="project-image"></p>

# Pay with Hepsipay

Hepsipay checkout deneyimi ve avantajları her yerde!

Bu dökümantasyon Hepsipay deneyimini JavaScript SDK çözümümüz olmadan kullanımına yöneliktir.

JavaScript SDK'in kullanılması durumunda [Frame JS Events](#frame-js-event) bölümü ve sonrasına ihtiyaç duymayacaktır

## 🚀 Demo

[https://pf-ui-pwh-qa.hepsipay.com/?token=YOUR\_MERCHANT\_SESSION\_TOKEN](https://pf-ui-pwh-qa.hepsipay.com/?token=YOUR_MERCHANT_SESSION_TOKEN)

<img src="https://images.hepsiburada.net/hepsipay/img/demo/pwh-demo-view.png" alt="project-screenshot" width="700" height="auto">

## 🧐 Özellikleri

Projenin bazı harika özellikleri:

*   Kartla öde
*   Hepsipay hesabımla öde
*   Hızlı alışveriş kredisi ile öde
*   Hepsifinans ile öde

## 🛠️ Kullanım/Kurulum:

1. Hepsipay Backend Entegrasyonu
   Hepsipay UI kullanım/testleri öncesinde Backend entegrasyonu tamamlanmalıdır

```
<*!-- Hepsipay Backend dökümantasyonu ayrıca iletilir -->
```

2. Backend entegrasyonu tamamlandığında API'den aşağıdaki gibi response alınır;

```json 
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
Hepsipay frame [6 farklı event](#messagetype-listesi) gönderir. Bunların tamamı, ihtiyaca bağlı entegre olunabilecek eventlerdir;

### JavaScript eventleri nasıl kontrol edilebilir?
*  WEB Platform
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
* Android Platform (Kotlin)
```kotlin
class HepsipayFrameCommunicator(){
    @JavascriptInterface
    fun postMessage(message:String){
        //Received message from webview in native, process data
    }
}
merchantWebview.addJavascriptInterface(HepsipayFrameCommunicator(),"HepsipayFrameCommunicator")
```

### messageType listesi;
#### - hp-payment-success
- ```event.data = { messageType: 'hp-payment-success', merchantOrderNumber: string; merchantCallBackUrl: string; token: string }```
- Müşteri ödeme sürecini 3Ds ve/veya non 3Ds ile başarıyla tamamladığı bildirilir
- *Bu event handle edildiği durumda frame'in kapatılması beklenir, yoksa event atıldıktan ön tanımlı bir süre kadar sonrasında [Kullanım/Kurulum 4. adımda](#-kullanımkurulum-) anlatılan aksiyon alınır
#### - hp-restart-frame *(handle edilmesi önerilir)*
- `event.data = { messageType: 'hp-restart-frame' }`
- Session ile ilgili devam edilemeyecek kritik bir hata oluşmuştur. Bu durumda müşteri işlemine devam edemeyecektir.Yeniden token üretilip iframe yeniden render edilmelidir
- Bu hata çoğunlukla üretilen SessionToken'ın artık geçerli olmadığı durumda gönderilir
#### - hp-payment-available-status
- `event.data = { messageType: 'hp-payment-available-status', isPaymentAllowed: boolean }`
- `isPaymentAllowed: true` Ödemeyi başlatabilir durumdadır
- `isPaymentAllowed: false` Ödeme başlatamayacak durumdadır. Çoğunlukla yeni kart form ekranı açtığı için atılır veya alışveriş kredisi sürecindedir
#### - hp-resize-frame
- `event.data = { messageType: 'hp-resize-frame', height: number }`
- Bu event frame işgal ettiği yükseklik alanı değiştikçe atılır.
- Burada gelen `height` değeri direkt olarak <iframe ... height={height}/> şeklinde kullanılarak frame'in yüksekliği dinamik kullanılması sağlanabilir
#### - hp-jwt-token
- `event.data = { messageType: 'hp-jwt-token', token: string }`
- Bu event hepsipay bakiyesini kullanarak ödeme yapmak isteyen kullanıcılardan Hepsipay şifreleri başarılı giriş yaptıktan sonra gönderilir
- Event'in atılması native iOS ve Android tarafında token'ın alıp storage üzerinde saklanması içindir. Bir sonraki Hepsipay webview açılırken, bu değer, aynı isimle tekrar Webview cookie üzerine yazılması içindir.
- Bu şekilde kullanıldığı zaman; müşterinin JWT token değer hâlâ geçerli olduğu sürece tekrar Hepsipay bakiyesi ile ödeme yapmak istediğinde yeniden şifre sorulmayacaktır.
#### - hp-selected-payment-info
- `event.data = { messageType: 'hp-selected-payment-info', paymentType: string, amount: number, binNumber?: string, actualInstallmentNumber?: number, displayInstallmentNumber?: number }`
* Payload object for relevant payment types;
* When "Kart İle Öde" selected
  * `{ amount: number, actualInstallmentNumber: number, displayInstallmentNumber: number, binNumber: string, paymentType: string }`
* When "Hepsipay Bakiyem İle Öde" selected
  * `{ amount: number, binNumber: string, paymentType: string }`
* When "Hızlı alışveriş kredisi ile öde" selected
  * `{ amount: number, paymentType: string }`
* When "Hepsifinans ile öde" selected
  * `{ amount: number, paymentType: string }`
#### - hp-redirect-deeplink
- `event.data = { messageType: 'hp-redirect-deeplink', deeplinkUrl: '{BROWSER_URL}', appDeeplinkUr: '{APP_DEEPLINK}', packageId: '{PLATFORM_PACKAGE_ID}', storeUrl: '{PLATFORM_STORE_BROWSER_URL}' }`
- [Event detaylı açıklama için tıklayın](#hp-redirect-deeplink)


## (ONLY-APP) Uygulama içerisinden WebView açılırken istenenler;
WebView açılırken cookie listesine 2 adet değer tanımlanması kullanıcı deneyimini iyileştirecektir
#### - unique-device-id
Bu bilgi kullanıcılarının ödeme akışlarında fraud ve 3Ds veya non-3Ds akışa mı girmesi gerektiği kurallarında parametre olarak çalışacaktır.
#### - hp-jwt-token
Bu bilginin kullanım amacı [messageType listesinde](#--hp-jwt-token) belirtilmişti. Uygulama ile cihazda saklanan bu bilgi kullanıcının tekrardan bakiyeli ödemelerde login akışına mâruz kalmaması için talep edilmektedir

## hp-redirect-deeplink
Bu event, "**Alışveriş Kredisi ile Öde**" yöntemi için **zorunlu** bir adımdır.
### Event Verisi
```javascript
event.data = {
  "messageType": "hp-redirect-deeplink",
  "deeplinkUrl": "{BROWSER_URL}",
  "appDeeplinkUrl": "{APP_DEEPLINK}",
  "packageId": "{PLATFORM_PACKAGE_ID}",
  "storeUrl": "{PLATFORM_STORE_BROWSER_URL}"
}
```
#### Açıklama
iOS Safari tarayıcısının iframe güvenlik politikaları, yeni bir sekme açma veya deeplink protokollerini çalıştırma yeteneğini kısıtlar. Bu nedenle, Alışveriş Kredisi ile Hepsipay native ekranlarına yönlendirme işlemi sadece **hp-redirect-deeplink** eventi kullanılarak gerçekleştirilebilir.
#### Kullanım Koşulları
- Event'in kontrol edilmesi için ilk olarak frame tarafına **hp-redirect-deeplink-handled** eventi geri döndürülmesi gereklidir. Bu sayede frame, ek bir aksiyon almadan statü kontrol aşamasına geçecektir.
- Web platformunda, event içindeki deeplinkUrl, yeni bir sekme içinde açılmalıdır.
- Native Application Webview kullanımında ise öncelikle appDeeplinkUrl, native seviyede açılmaya çalışılmalıdır. Eğer bu başarısız olursa, packageId ile belirtilen uygulama, ilgili app/play store üzerinde açılmalıdır.
#### Web Platform Örnek Kullanım
```js
// Event kontrolü ve geri bildirim
window.addEventListener('message', function (event) {
  if (event.data.messageType === 'hp-redirect-deeplink') {
    // hp-redirect-deeplink-handled eventi ile kontrolü geri bildir
    iframeElement.contentWindow.postMessage({ messageType: 'hp-redirect-deeplink-handled' }, '*');

    // Web platformunda yeni sekme açma
    window.open(event.data.deeplinkUrl, '_blank');
  }
});

```
