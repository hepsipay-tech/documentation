PWHP
Android

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