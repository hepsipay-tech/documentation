# **Pay With HepsiPay Android Documentation**

# Contents
- [Integration](#integration)
- [Initialization](#initialization)
    - [ComposeView](#composeview)
    - [Initialize](#initialize)
- [Usage](#usage)
- [UI Customization](#customization)
    - [Font](#font)
    - [Color](#color)

# <a name="integration"> Integration </a>
Projenizde Compose desteği bulunmuyorsa ilgili bağlantıdan kurulumları tamamlayın: [Set up Compose for an existing app](https://developer.android.com/develop/ui/compose/setup#setup-compose)

Root **`build.gradle`** dosyasına belirtilen Maven repository tanımlamalarını ekleyin:

```kotlin
allprojects {
    repositories {
        google()
        mavenCentral()
        maven(url = "https://images.hepsiburada.net/payment/assets/pwhp-native") {
            content {
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui-compose")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-data")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-domain")
            }
        }
    }
}
```

Eğer projenizde **`settings.gradle`** kullanılıyorsa Maven repository tanımlamalarını belirtilen şekilde yapabilirsiniz:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven(url = "https://images.hepsiburada.net/payment/assets/pwhp-native") {
            content {
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui-compose")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-data")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-domain")
            }
        }
    }
}
```

İlgili modülün **`build.gradle`** dosyasına belirtilen implementasyonu ekleyin:

```kotlin
dependencies {
    implementation("com.hepsiburada.hepsipay:paywithhp-ui-compose:0.0.3")
}
```

# <a name="initialization">Initialization</a>
### <a name="composeview">1. Compose View (XML)</a>
Eğer XML yerine Composable bir ekranda UI gösterimi yapılacaksa bir sonraki adıma geçilebilir.

SDK arayüzünün gösteriminin sağlanacağı kısımda bir Compose View oluşturun:
```xml
<androidx.compose.ui.platform.ComposeView
        android:id="@+id/compose_view_pwhp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
```

### <a name="initialize">2. Initialize</a>
PaymentActionState için bir Flow tanımlayın:

```kotlin
class CheckoutViewModel : ViewModel() {

    private val _paymentActionState = MutableStateFlow<PaymentActionState>(PaymentActionState.None)
    val paymentActionState get() = _paymentActionState.asStateFlow()

    fun updatePaymentActionState(state: PaymentActionState) {
        _paymentActionState.update {
            state
        }
    }

}
```

Compose View tanımlamalarını gösterildiği şekilde yapın:

```kotlin
binding.composeViewPwhp.apply {
  setViewCompositionStrategy(
    ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed,
  )
  
  setContent {
      
    // View Model içerisindeki PaymentActionState Flow değeri
    // Örn: MutableStateFlow<PaymentActionState>(PaymentActionState.None)
    val paymentActionState = viewModel.paymentActionState.collectAsStateWithLifecycle()

    PWHPTheme {
        PWHPView(
            modifier = Modifier.padding(16.dp),
            token = "MERCHANT_TOKEN",
            hideHeader = false,
            uniqueDeviceId = "UNIQUE_DEVICE_ID",
            environment = Environment.TEST,
            paymentActionState = paymentActionState.value,
            onPaymentActionStateChanged = viewModel::updatePaymentActionState,
            onResult = { result ->
                when (result) {
                    is PWHPResult.PaymentAvailable -> {
                        // Arayüzde bulunan "Ödemeyi Tamamla" butonu
                        binding.btnMakePayment.isEnabled = result.isAvailable
                    }

                    is PWHPResult.CompletePayment -> {
                        // Payment flow completed
                        // Result: Callback URL, Order Number, Token
                    }
                }
            }
        )
    }
  }
}
```

**`token: String`** değerini belirleyin. **(Required)**

**`hideHeader: Boolean`** değerini belirleyin. Hepsipay logolu header gizlenebilir. **(Optional)** *(Default: false)*

**`uniqueDeviceId: String?`** benzersiz bir cihaz kimliği belirleyin. **(Optional)**

**`environment: Environment`** test veya production ortamını tanımlayın. **(Optional)** *(Default: Environment.TEST)*

**`paymentActionState: PaymentActionState`** değerini belirleyin. ödeme başlatma durumunu iletir. **(Required)** *(PaymentActionState.None || PaymentActionState.MakePayment)*

**`onPaymentActionStateChanged:`** değerini belirleyin. paymentActionState buradan gelen result durumuna göre güncellenmelidir. **(Required)**

#### PWHPResult.CompletePayment
Ödeme işlemleri tamamlandığında tetiklenir. Result içerisinde `calllbackUrl`, `token` ve `orderNumber` döndürülür. Burada success ekranı gösterilebilir.
#### PWHPResult.PaymentAvailable
Mevcut durumda ödeme yapılıp yapılamayacağının bilgisini verir. Arayüzde **"Ödemeyi Tamamla"** gibi bir buton varsa **enabled/disabled** ayarı bu bilgiye göre yapılmalıdır.

# <a name="usage">Usage</a>
Ödeme işlemlerinin tamamlanması için bir Flow üzerinde tutulan PaymentActionState değeri MakePayment olarak güncellenmelidir.
```kotlin
// Arayüzde bulunan "Ödemeyi Tamamla" butonu
binding.btnMakePayment.setOnClickListener {
  viewModel.updatePaymentActionState(PaymentActionState.MakePayment)
}
```

# <a name="customization">UI Customization</a>
UI customization işlemleri resources üzerinden yapılmaktadır. Projenizin `values` resource klasörünün içerisine yeni bir resource file oluşturun. Örneğin; `pwhp_resources.xml` 

[<img src="sample-pwhp-resources.png" width="250"/>](sample-pwhp-resources.png)

### <a name="font">1. Font</a>
Varsayılan olarak **Inter** font kullanılmaktadır. Custom font tanımlaması yapabilmek için `pwhp_resources.xml` içerisinde key-value tanımlamalarını yapın:

```xml
<resources>
    <!-- Fonts -->
    <font name="font_regular_pwhp">@font/quicksand_regular</font>
    <font name="font_medium_pwhp">@font/quicksand_medium</font>
    <font name="font_semi_bold_pwhp">@font/quicksand_semi_bold</font>
</resources>
```

### <a name="color">2. Color</a>

[<img src="pwhp-color-scheme.png" width="800"/>](pwhp-color-scheme.png)

Custom renk tanımlaması yapabilmek için `pwhp_resources.xml` içerisinde Color Scheme üzerinden karşılık gelen key'lerin değerlerinin değiştirilmesi gerekmektedir:

```xml
<resources>
    <!-- Colors -->
    <color name="color_primary_pwhp">@android:color/holo_red_dark</color>
</resources>
```
