# **Pay With HepsiPay iOS Documentation**

# Contents
- [Integration](#integration)
  - [Option 1 - Swift Package Manager - Package.swift](#spmpackage)
  - [Option 2 - Swift Package Manager - Project Package Dependencies](#spmproject)
  - [Option 3 - Manual](#manual)
- [Initialization](#initialization)
- [Usage](#usage)

# <a name="integration"> Integration </a>

### <a name="spmpackage">Option 1 - Swift Package Manager - Package.swift</a>
Projenizde `Package.swift` kullanıyorsanız Package dependencies kısmına aşağıdaki satırı ekleyin.
```swift
dependencies: [
    ...
    .package(url: "https://gitea.com/hepsipay/PayWithHPSPM.git", from: "1.2.2")
]
```
- Kullanıcı adı:`hepsipay`
- Şifre: Size özel ilettiğimiz access token

Kullanıcı adı şifre bilgilerini giremediğiniz bir durum oluşursa (Örneğin pipeline makineleri için) kullanıcı adı ve şifre bilgisini URL içerisine aşağıdaki gibi yazabilirsiniz.
```swift
dependencies: [
    ...
    .package(url: "https://hepsipay:YourAccessToken@gitea.com/hepsipay/PayWithHPSPM.git", from: "1.2.2")
]
```

Kullanacağınız target için dependencies kısmına ekleme yapın.
```swift
.target(
    name: "YourTarget",
    dependencies: [
        "YourDependencies",
        ...
        "PayWithHPSPM"
    ]
)
```
### <a name="spmproject">Option 2 - Swift Package Manager - Project Package Dependencies</a>
`Package.swift` kullanmıyorsanız `Project -> Settings -> Package Dependencies` sekmesinde + butonuna basarak `Search or Enter Package URL` kısmına aşağıdaki URL'i girip ekleyin.
```
https://gitea.com/hepsipay/PayWithHPSPM.git
```
`Project -> Target Settings -> General -> Frameworks, Libraries, and Embedded Content` kısmında `PayWithHPSPM` kütüphanesi görünür olmalıdır. Ekli değilse + butonundan ekleyin.

### <a name="manual">Option 3 - Manual</a>
Repository içerisinde artifacts klasöründe bulunan PayWithHPFramework.xcframework dosyasını projenizin içine sürükleyip bırakın. Embed kısmında `Embed & Sign` seçin.

`Project -> Target Settings -> General -> Frameworks, Libraries, and Embedded Content` kısmında `PayWithHPFramework.xcframework` dosyası görünür olmalıdır, ekli değilse soldaki proje ağacından sürükleyip ekleyin.


# <a name="initialization">Initialization</a>

Framework import edilir.
```swift
import PayWithHPFramework
```

`PayWithHPManager.getPayWithHPView` metodu aşağıdaki gibi çağırılır. Bu metod, akışın sağlanacağı bir UIView döner.
```swift
let payWithHPView = PayWithHPManager.getPayWithHPView(
    model: .init(
        environment: .qa,
        token: "MERCHANT_TOKEN",
        uniqueDeviceId: "UNIQUE_DEVICE_ID",
        style: .init(
            edgeInsets: .init(top: 16, left: 16, bottom: 16, right: 16),
            overrideFontFamily: "Quicksand",
            hideHeader: false
        ),
        paymentAvailableHandler: { isPaymentAvailable in
            self.payButton.isUserInteractionEnabled = isPaymentAvailable
            self.payButton.setBackgroundColor(isPaymentAvailable ? .accent : .gray)
        },
        paymentCompleteHandler: { paymentResult in
            let successVC = PaymentSuccessViewController()
            // Payment Result Contains: paymentOption, orderNumber, token, merchantCallbackURL
            successVC.paymentResult = paymentResult
            self.navigationController?.pushViewController(successVC, animated: true)
        }
    )
)
```

**Model Parametreleri**
- **`environment: EnvironmentType`**: Ortam bilgisi (qa: Alt ortam, prod: Canlı ortam)
- **`token: String`**: Hepsipay token
- **`uniqueDeviceId: String?`**: Merchant tarafından verilen unique device id. (Optional)
- **`style: Style?`**: Stile ait özellikler bu struct içerisine verilir.
  -  **`edgeInsets: UIEdgeInsets`**: Dönülecek view için inset değeri (`Default: top: 16, left: 16, bottom: 16, right: 16`)
  -  **`overrideFontFamily: String?`**: Varsayılan olarak kullanılan `Inter` font family'i override eder (`Default: nil`). X font family için şu ttf dosyaları projeniz içerisinde bulunmalıdır: **`X-Bold.ttf, X-Medium.ttf, X-Regular.ttf, X-SemiBold.ttf`**
  -  **`hideHeader: Bool`**: Varsayılan olarak false değeri kullanılır. Eğer başlık alanının görünmesi istenmiyor ise true gönderilmelidir.
- **`paymentAvailableHandler`**: Ödeme yapabilme durumunun değiştiği durumlarda buraya düşer.
  - **`isPaymentAvailable: Bool`**: true ise ödeme yapabilir, false ise yapmamalıdır. Merchant tarafında bulunan **Ödeme Yap** butonu bu değere göre aktif/pasif edilir.
- **`paymentCompleteHandler`**: Ödeme başarılı olarak tamamlandıktan sonra buraya düşer.
  -  **`paymentResult`**: Ödeme sonucuna ait aşağıdaki bilgileri içerir.
      -  **`paymentOption: OptionType`**: Ödemenin hangi seçenekle yapıldığı bilgisi. (Wallet, Card, Loan)
      -  **`orderNumber: String`**: Sipariş numarası
      -  **`token: String`**: Kullanılan token bilgisi
      -  **`merchantCallbackURL: String`**: WebView içerisine bu URL verilerek success ekranı gösterilebilir.


**Alışveriş kredisi ile öde** seçeneği ile ilerlendiğinde Hepsiburada uygulaması deeplink ile başlatılacaktır. Bunun için uygulamanızın `Info.plist` dosyası içerisine `Queried URL Schemes` dizisine **`hbapp`** değeri eklenmelidir.
```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>hbapp</string>
</array>
```

# <a name="usage">Usage</a>

PayWithHPManager.getPayWithHPView metodundan dönen UIView nesnesi, istenen yere eklenebilir. Merchant tarafında bulunan **Ödeme Yap** butonuna basıldığında aşağıdaki metod çağırılır.

```swift
PayWithHPManager.makePayment()
```
