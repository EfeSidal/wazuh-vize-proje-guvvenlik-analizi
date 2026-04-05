# Proje Analizi - Adım 3: İş Akışları ve CI/CD Pipeline Analizi

**Analiz Konusu:** Wazuh GitHub Actions İş Akışı (`.github/workflows`)  
**İnceleme Nesnesi:** `test_manager.yml` (veya benzeri bir Unit Test/Build workflow'u)

## 3.1. Seçilen İş Akışının Adım Adım Analizi
Wazuh Manager deposunda yer alan temel bir CI (Continuous Integration) dosyasını ele aldığımızda, sürecin şu mimari adımlarla ilerlediğini görüyoruz:

1.  **Trigger (Tetikleyici):** 
    *   `on: [push, pull_request]`: Kod her gönderildiğinde veya bir birleştirme isteği (PR) açıldığında fabrika çalışmaya başlar.
2.  **Environment Setup (Ortam Kurulumu):**
    *   `runs-on: ubuntu-latest`: İşlemler izole bir Ubuntu konteynerinde başlatılır.
    *   **Checkout:** `actions/checkout@v3` ile repo kodu sanal makineye çekilir.
3.  **Dependency Installation (Bağımlılıklar):**
    *   Wazuh, C ve Python ağırlıklı olduğu için `gcc`, `make`, `python3-dev` gibi paketler `apt-get` ile kurulur.
4.  **Compilation & Build (Derleme):**
    *   `make` komutuyla kaynak kod derlenir. Burada amaç, kodun sözdizimi (syntax) hatası içerip içermediğini ve derlenebilir olduğunu doğrulamaktır.
5.  **Unit Tests (Birim Testleri):**
    *   **Nasıl Çalışır?** `ctest` (C kodları için) veya `pytest` (Python bileşenleri için) araçları kullanılır. Bu adımda, her bir fonksiyonun beklenen çıktıyı verip vermediği küçük parçalar halinde test edilir.
    *   **Güvenlik Kontrolü:** Eğer testlerden biri kalırsa (fail), pipeline durur ve hatalı kodun ana dallara (main/master) karışması engellenir.

## 3.2. Kritik Soru: Webhook Nedir ve Ne İşe Yarar?

**Sistem Mimarı Tanımı:** 
Webhook, bir sistemde bir olay (event) gerçekleştiğinde, başka bir sisteme gerçek zamanlı veri gönderen bir **HTTP POST** isteğidir. Basitçe; "Benim tarafımda bir şey oldu, işte detayları, sen de kendi tarafında ne gerekiyorsa yap" diyen bir mesajcıdır.

### **Wazuh / CI/CD Özelindeki Görevleri:**
1.  **GitHub -> Jenkins/Buildkite:** Wazuh bazen çok ağır testler için GitHub Actions yerine kendi özel sunucularını (Jenkins) kullanır. GitHub, bir "Push" olduğunda Webhook aracılığıyla Jenkins sunucusuna "Yeni kod geldi, derlemeye başla" talimatı gönderir.
2.  **Bildirimler (Slack/Discord):** Testler bittiğinde veya bir hata oluştuğunda, pipeline bir Webhook tetikleyerek ekibin iletişim kanalına (örneğin Slack) "Build #452 başarısız oldu!" mesajı atar.
3.  **Deployment (Dağıtım):** Kod tüm testlerden geçerse, Webhook aracılığıyla Docker Hub'a veya bir bulut sunucusuna "Yeni imajı oluştur ve yayına al" sinyali gönderilir.

## 3.3. CI/CD Pipeline Güvenliği (Security Insights)
Bir güvenlik uzmanı olarak bu pipeline'da şu iki noktaya dikkat edilmelidir:
*   **Secrets Management:** Pipeline içinde kullanılan API anahtarları veya şifreler asla kodun içine (hardcoded) yazılmaz. GitHub "Actions Secrets" içinde şifrelenmiş olarak tutulur ve sadece çalışma anında (runtime) çağrılır.
*   **Artifact Integrity:** Testler bittikten sonra üretilen binary dosyaların (artifact) hash değerleri alınarak, dağıtım aşamasında dosyanın değiştirilmediği garanti altına alınmalıdır.


**Sıradaki Adım (Adım 4):** Docker Mimarisi ve Konteyner Güvenliği. Wazuh'un Docker katmanlarını ve "Konteyner içinde root olmak ya da olmamak" meselesini inceleyeceğiz. Devam edelim mi?
