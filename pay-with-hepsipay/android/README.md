# **Pay With HepsiPay Android Documentation**

# Contents
- [Integration](#integration)
- [Initialization](#initialization)
    - [PWHPConfig](#pwhpconfig)
    - [Container View](#containerview)
    - [Initialize](#initialize)
- [Usage](#usage)

# <a name="integration"> Integration </a>
Root **`build.gradle`** dosyasına belirtilen Maven repository tanımlamalarını ekleyin:

```kotlin
allprojects {
    repositories {
        google()
        mavenCentral()
        maven(url = "https://images.hepsiburada.net/payment/assets/pwhp-native") {
            content {
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui")
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
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui")
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
    implementation("com.hepsiburada.hepsipay:paywithhp-ui:0.0.3")
}
```

# <a name="initialization">Initialization</a>
### <a name="pwhpconfig">1. PWHPConfig</a>
Konfigürasyon objesini oluşturun.
```kotlin
val config = PWHPConfig(
    token = "MERCHANT_TOKEN",
    uniqueDeviceId = "UNIQUE_DEVICE_ID"
)

```
**`token: String`** değerini belirleyin. **(Required)**

**`uniqueDeviceId: String?`** benzersiz bir cihaz kimliği belirleyin. **(Optional)**

### <a name="containerview">2. Container View</a>
SDK arayüzünün gösterileceği bir FragmentContainerView oluşturun:
```xml
<androidx.fragment.app.FragmentContainerView
         android:id="@+id/fl_container"
         android:layout_width="match_parent"
         android:layout_height="wrap_content" />
```
### <a name="initialize">3. Initialize</a>
`PWHPPresenterImpl` sınıfından bir obje oluşturun:
```kotlin
private val presenter = PWHPPresenterImpl()
```

`presenter` objesi üzerinden `init()` metodunu belirtildiği gibi çağırın:
```kotlin
presenter.init { builder ->
            builder.fragmentManager = childFragmentManager
            builder.pwhpConfig = config // 1. adımda oluşturuldu
            builder.containerViewId = R.id.fl_container // 2. adımda oluşturuldu
            builder.resultCallback = { result ->
                when (result) {
                    is PWHPResult.CompletePayment -> {

                    }

                    is PWHPResult.PaymentAvailable -> {
                        // Arayüzde bulunan "Ödemeyi Tamamla" butonu
                        binding.btnMakePayment.isEnabled = result.isAvailable
                    }
                }
            }
        }
```

#### PWHPResult.CompletePayment
Ödeme işlemleri tamamlandığında tetiklenir.
#### PWHPResult.PaymentAvailable
Mevcut durumda ödeme yapılıp yapılamayacağının bilgisini verir. Arayüzde **"Ödemeyi Tamamla"** gibi bir buton varsa **enabled/disabled** ayarı bu bilgiye göre yapılmalıdır.

# <a name="usage">Usage</a>
Ödeme işlemlerinin tamamlanması için `presenter` objesi üzerinden `makePayment()` metodunu belirtildiği gibi çağırın:
```kotlin
// Arayüzde bulunan "Ödemeyi Tamamla" butonu
binding.btnMakePayment.setOnClickListener {
    presenter.makePayment()
}
```
