# Proje Analizi - Adım 1: Kurulum ve `wazuh-install.sh` Tersine Mühendislik

**Hazırlayan:** [Efe Sidal]  
**Rol:** Güvenlik Uzmanı / Sistem Mimarı  
**Analiz Edilen Repo:** [Wazuh](https://github.com/wazuh/wazuh)  
**Hedef Dosya:** `wazuh-install.sh` (Assisted Installation Script)

## 1.1. Script Genel İşleyişi (Logic Flow)
`wazuh-install.sh` scripti, karmaşık bir yapıyı (Indexer, Manager, Dashboard) tek bir komutla ayağa kaldırmayı amaçlayan "orchestrator" niteliğinde bir Bash scriptidir. Çalıştırıldığında şu ana aşamaları takip eder:

1.  **Environment Check:** İşletim sistemi (Ubuntu, CentOS vb.) ve CPU mimarisi (x86_64, AARCH64) uyumluluğunu kontrol eder.
2.  **Privilege Escalation:** Scriptin `root` yetkisiyle çalışıp çalışmadığını denetler (SIEM bileşenleri düşük yetkiyle kurulamaz).
3.  **Dependency Management:** Gerekli olan `gnupg`, `apt-transport-https`, `curl` gibi temel paketlerin varlığını kontrol eder, eksikse yükler.
4.  **Repository Setup:** Wazuh'un resmi paket depolarını sisteme ekler.
5.  **Certificate Generation:** Bileşenler arası (Indexer <-> Manager) iletişim için gerekli olan SSL/TLS sertifikalarını (varsayılan olarak self-signed) üretir.
6.  **Installation & Orchestration:** Seçilen moda göre (`-a` all-in-one gibi) bileşenleri indirir, kurar ve `systemd` servislerini yapılandırır.

## 1.2. Oluşturulan Dizinler ve Yetki Analizi
Script, sistem üzerinde derinlemesine bir dizin yapısı oluşturur. Kritik olanlar şunlardır:

| Dizin | Amaç | Varsayılan Yetki |
| :--- | :--- | :--- |
| `/var/ossec` | Wazuh Manager'ın kalbi; kurallar, loglar ve binary dosyalar burada tutulur. | `root:ossec` (750) |
| `/etc/wazuh-indexer` | Log verilerinin depolandığı arama motoru yapılandırması. | `wazuh-indexer:wazuh-indexer` |
| `/etc/wazuh-dashboard` | Kullanıcı arayüzü yapılandırma dosyaları. | `wazuh-dashboard:wazuh-dashboard` |
| `/usr/share/wazuh-indexer/certs` | SSL sertifikalarının tutulduğu en kritik dizin. | `500` (r-x------) |

> **Güvenlik Notu:** Script, sertifika dizinleri için `500` veya `700` gibi kısıtlayıcı yetkiler kullanarak "Least Privilege" (En Az Yetki) prensibine sadık kalmaya çalışır.

## 1.3. Kritik Soru: Kaynak Güvenliği ve İmza Kontrolü
Wazuh kurulum scripti, modern güvenlik standartlarını ne kadar karşılıyor?

### **A. Dış Paket Güvenliği (Hash & GPG)**
Script, paketleri çekerken körü körüne bir `curl | bash` mantığıyla çalışmaz. Analizim sonucunda şu mekanizmalar tespit edilmiştir:
*   **GPG Anahtar Doğrulaması:** Script, paketleri indirmeden önce Wazuh'un resmi GPG anahtarını (`[https://packages.wazuh.com/key/GPG-KEY-WAZUH](https://packages.wazuh.com/key/GPG-KEY-WAZUH)`) sisteme ithal eder. Bu sayede indirilen her `.deb` veya `.rpm` paketi, işletim sisteminin paket yöneticisi tarafından imzası kontrol edilerek doğrulanır.
*   **Repository Pinning:** Sadece güvenilir kaynaktan veri çekilmesini garanti altına alır.

### **B. Scriptin Kendisi (Zayıf Nokta)**
Kritik bir açık nokta ise şudur: Kullanıcılara genellikle `curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh) && sudo bash wazuh-install.sh` komutu önerilir. 
*   **Risk:** Scriptin kendisi indirilmeden önce bir **SHA256 Hash kontrolüne** tabi tutulmaz. Eğer Wazuh'un indirme sunucusu (CDN) ele geçirilirse (Supply Chain Attack), saldırgan scriptin içine zararlı kod enjekte edebilir ve kullanıcı bunu fark etmeden `sudo` yetkisiyle çalıştırabilir.

### **C. Sertifika Güvenliği**
Script, kurulum sırasında otomatik sertifikalar üretir. Bu, "Out-of-the-box" kullanım için kolaylık sağlasa da, üretim (Production) ortamlarında **CA (Certificate Authority)** tarafından imzalanmamış sertifikaların kullanılması "Man-in-the-Middle" (Ortadaki Adam) saldırılarına karşı bir risk yüzeyi oluşturur.

---

**Sonuç (Uzman Görüşü):**  
Wazuh'un kurulum mimarisi, paket bazında oldukça güvenli (GPG imzalı) ancak scriptin dağıtım yöntemi açısından klasik "güven temelli" (trust-based) bir model izlemektedir. Gerçek dünya standartlarında bir mimar, bu scripti çalıştırmadan önce içeriğini manuel olarak denetlemeli ve mümkünse sertifika üretim aşamasını kurumsal bir CA ile değiştirmelidir.
