# Adım 1: Kurulum ve install.sh Analizi

**Proje:** Wazuh[](https://github.com/wazuh/wazuh)  
**Analiz edilen dosya:** `install.sh` (root klasörde)  
**Raw link:** https://raw.githubusercontent.com/wazuh/wazuh/main/install.sh  
**Son commit (install.sh):** 1 Nisan 2026 - "refactor: streamline installer logging"

## Görev
Bu dosya tek tek ne yapıyor? Hangi dizinleri oluşturuyor, hangi yetkileri istiyor? Belgelenmesi gerekmektedir.

## Analiz

`install.sh` **kaynak kodundan derleme** (source install) yapan interaktif bir kurulum wizard’ıdır. Manager veya Agent kurulumunu yönetir.

### Script’in Adım Adım Yaptıkları:
1. Kendi dizinine geçer ve argümanları okur (dil, debug, binary vs.).
2. OS tespiti yapar.
3. Kurulum tipi seçimi (Manager / Agent).
4. Mevcut kurulum kontrolü (`/var/ossec` veya `/var/wazuh-manager`).
5. Kurulum dizini belirlenir (kullanıcı değiştirebilir).
6. Özellikler toggle’lanır (Syscheck, Rootcheck, Active Response vb.).
7. `make deps` ile external kütüphaneler indirilir.
8. `make` ile kaynak kod derlenir.
9. Dosyalar `make install` ile yerleştirilir.
10. Konfigürasyon yapılır (Agent ise manager IP’si istenir).
11. Systemd / init script’leri yerleştirilir ve servis otomatik başlangıca eklenir.

### Oluşturduğu / Değiştirdiği Dizinler:
- Ana dizin: `/var/ossec` (Agent) veya `/var/wazuh-manager` (Manager)
- Alt dizinler: `bin/`, `etc/`, `logs/`, `queue/`, `active-response/`, `var/` vb.
- Servis dosyaları: `/etc/systemd/system/` veya `/etc/init.d/`

### İstediği Yetkiler:
- **Mutlaka root** yetkisi gerekir (script başında kontrol eder, root değilse çıkar).
- `chmod` ve `chown` işlemleri `make install` aşamasında yapılır.

## Kritik Soru Cevabı
**Yazılımın indirdiği kaynaklar ne kadar güvenli? Depardan paket çekerken hash kontrolü yapıyor mu, yoksa körü körüne ‘curl | bash’ mantığıyla mı çalışıyor?**

**Cevap:**  
Bu `install.sh` **curl | bash** pattern’i kullanmıyor. Tamamen yerel kaynak kodundan derliyor.  
Tek external indirme `make deps` komutu ile external/ klasörüne yapılır.  
**Hash veya GPG imza doğrulaması bu script’te yok.**  
Bağımlılıklar `make` tarafından indiriliyor, ancak doğrulama mekanizması belirtilmemiş.

**Güvenlik Bulgusu (Orta Risk):**  
Root yetkisi + hash kontrolü olmadan external dependency indirme → **supply-chain attack** riski taşıyor.  
En iyi pratik: Repoyu clone ettikten sonra `git tag` ve GPG imzalarını manuel kontrol etmek.

## Ekran Görüntüleri (Sen Ekle)
- `install.sh` dosyasının GitHub sayfası
- Script’in başında root kontrolü satırı
- `make deps` ve `make` komutlarının geçtiği bölüm

---
**Hazırlayan:** Efe Sidal  
**Tarih:** 5 Nisan 2026
