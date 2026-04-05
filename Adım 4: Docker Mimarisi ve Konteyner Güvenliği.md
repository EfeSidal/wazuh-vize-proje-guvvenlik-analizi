# Proje Analizi - Adım 4: Docker Mimarisi ve Konteyner Güvenliği

**Analiz Konusu:** Wazuh Docker-Compose Yapısı ve İmaj Güvenliği  
**Repo:** `wazuh-docker` (Wazuh'un resmi Docker dağıtımı)

## 4.1. Docker İmajı ve Katman (Layer) Analizi
**Docker İmajı Nedir?**  
Docker imajı, uygulamanın çalışması için gereken kodun, kütüphanelerin, ortam değişkenlerinin ve yapılandırma dosyalarının bir araya getirilmiş, **salt okunur (read-only)** bir paketidir. 

**Katmanlı Yapı:**  
Wazuh imajları (özellikle `wazuh-manager`) çok katmanlı bir yapıdan oluşur. Her `RUN`, `COPY` ve `ADD` komutu yeni bir katman oluşturur:
1.  **Base Layer:** Genellikle `CentOS` veya `Amazon Linux 2` gibi hafifletilmiş bir OS katmanı.
2.  **Dependency Layer:** Wazuh'un ihtiyaç duyduğu `openssl`, `libc` gibi kütüphanelerin yüklendiği katman.
3.  **Application Layer:** Wazuh binary dosyalarının kopyalandığı katman.
4.  **Config Layer:** Varsayılan konfigürasyonların (ossec.conf) yerleştiği katman.

> **Mimari Avantaj:** Katmanlı yapı sayesinde, eğer sadece konfigürasyon değişirse Docker tüm imajı tekrar indirmez; sadece değişen üst katmanı çeker. Bu, bant genişliği ve hız kazandırır.

## 4.2. Wazuh Docker-Compose Mimarisi
Wazuh tek bir konteyner değil, bir **mikroservis kümesi** olarak çalışır. `docker-compose.yml` dosyası incelendiğinde şu servislerin birbirine bağımlı olduğu görülür:
*   **wazuh-indexer:** Logları saklayan ve indeksleyen motor.
*   **wazuh-manager:** Agent'lardan gelen veriyi işleyen beyin.
*   **wazuh-dashboard:** Görselleştirme arayüzü.

**Konteyner Erişimi:**  
Varsayılan yapılandırmada konteynerler, ana sistemin (host) sadece belirli portlarına (`1514`, `55000`, `443` vb.) erişebilir. Ancak **Volume Mounts** (`-v /var/lib/wazuh:/var/ossec/data`) kullanılarak, konteyner içindeki verilerin ana sistemde kalıcı olması sağlanır. Bu nokta, güvenlik açısından en kritik sızıntı alanıdır.



## 4.3. Modern Mimari Karşılaştırması

| Özellik | Sanal Makine (VM) | Docker (Konteyner) | Kubernetes (K8s) |
| :--- | :--- | :--- | :--- |
| **İzolasyon** | Donanım seviyesi (Yüksek) | İşletim sistemi seviyesi (Orta) | Orkestrasyon bazlı (Yüksek/Karmaşık) |
| **Hız** | Yavaş (Dakikalar) | Çok Hızlı (Saniyeler) | Hızlı (Ölçeklenebilir) |
| **Kaynak Kullanımı** | Çok yüksek (Her VM'de tam OS) | Çok düşük (Kernel paylaşılır) | Verimli (Cluster yönetimi) |
| **Kullanım Amacı** | Kompleks, eski monolitik yapılar. | Uygulama paketleme ve taşıma. | Binlerce konteyneri yönetme/ölçekleme. |

## 4.4. Güvenlik Sertleştirme (Hardening) Önerileri
Bir güvenlik uzmanı olarak Wazuh Docker ortamını şu adımlarla kurşun geçirmez hale getirebiliriz:

1.  **Rootless Execution:** Konteyner içindeki işlemleri `root` kullanıcısı yerine düşük yetkili bir kullanıcıyla (`USER wazuh`) çalıştırarak "Privilege Escalation" riskini azaltmak.
2.  **Read-Only Filesystem:** Konteynerin dosya sistemini salt-okunur modda başlatıp, sadece gerekli dizinlere (`/var/ossec/data`) yazma yetkisi vermek.
3.  **Resource Limits:** Bir konteynerin tüm host kaynağını tüketip sistemi kilitlemesini (DoS) engellemek için CPU ve RAM limitleri koymak.
4.  **İmaj Tarama:** `Trivy` veya `Anchore` gibi araçlarla imaj içindeki kütüphanelerin bilinen zafiyetlerini (CVE) düzenli olarak taramak.

---
