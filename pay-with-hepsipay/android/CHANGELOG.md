PWHP
Android

### [1.2.1] - 2025-07-24

- **fix**: Push doğrulama ekranında "SMS ile devam et" butonu, artık yalnızca geri sayım süresi dolduğunda görünecek şekilde düzenlendi.

### [1.2.0] - 2025-03-20

- **feat**: Common Payment işlevi eklendi.
- **feat**: Yeni kampanya görünümü eklendi.
- **feat**: Text içerisindeki URL renklendirmeleri değiştirildi.
- **feat**: Linkleme ekranındaki bilgilendirme mesajına hyper link eklendi.
- **feat**: Ödeme seçenekleri ekranına kampanya görünümü eklendi.
- **feat**: Common Payment ekranındaki kart formuna bilgilendirme mesajı eklendi.
- **feat**: Kayıtlı kart numarası ile işlemde kart kaydetme checkbox kaldırıldı.
- **feat(log)**: Metric eventler eklendi.

### [1.1.3] - 2024-12-02

- **feat(log)**: Page Load ve Payment Success işlemleri için metric eventleri eklendi.
- **fix**: Linkleme OTP ekranındaki başarılı işlem doğru şekilde işlenmeye başlandı.

### [1.1.2] - 2024-10-16

- **feat**: PWHPResult içerisine **`PaymentFailureLog`** callback eklendi.
- **fix**: Cüzdan ile ödeme seçeneğinden ön ödemeli kart kaldırıldı.

### [1.1.1] - 2024-10-04

- **feat**: Hepsifinans ödeme yöntemi eklendi.
- **feat**: PWHPView fonksiyonuna Network loglarını görebilmek için **`networkInterceptors`** parametresi eklendi.
- **feat(log)**: Request header içerisine platform, osplatform, sdk version parametreleri eklendi.
- **refactor**: Metinler dil dosyasından okunacak şekilde ayarlandı.