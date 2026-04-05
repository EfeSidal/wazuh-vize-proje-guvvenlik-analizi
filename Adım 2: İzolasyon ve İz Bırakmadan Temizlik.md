# Proje Analizi - Adım 2: İzolasyon ve İz Bırakmadan Temizlik

**Analiz Konusu:** Wazuh All-in-One Kurulum Sonrası Forensics ve Cleanup  
**Metot:** Post-Installation Audit & Manual Purge

## 2.1. Standart Kaldırma İşlemi Neden Yetersiz?
Wazuh kurulum scripti paket bazlı çalıştığı için `apt remove` komutu sadece binary dosyaları siler. Ancak bir SIEM mimarisi olduğu için sistemde şu kritik kalıntıları bırakır:
*   **Konfigürasyonlar:** `/etc/wazuh-indexer`, `/etc/wazuh-manager` vb.
*   **Log Verileri:** `/var/ossec/logs` (En büyük veri buradadır).
*   **Sistem Kullanıcıları:** `wazuh`, `wazuh-indexer`, `wazuh-dashboard` kullanıcıları ve grupları `/etc/passwd` içerisinde kalmaya devam eder.

## 2.2. Adım Adım Tam Temizlik (The Purge)

Sistemde hiçbir iz kalmaması için şu komut dizisi uygulanmalıdır:

```bash
# 1. Servisleri durdur ve devre dışı bırak
systemctl stop wazuh-manager wazuh-indexer wazuh-dashboard
systemctl disable wazuh-manager wazuh-indexer wazuh-dashboard

# 2. Paketleri konfigürasyon dosyalarıyla birlikte sil
apt-get purge -y wazuh-manager wazuh-indexer wazuh-dashboard

# 3. Manuel dizin temizliği (En kritik adım)
rm -rf /var/ossec
rm -rf /etc/wazuh-*
rm -rf /usr/share/wazuh-*

# 4. Kullanıcı ve Grupların silinmesi
userdel wazuh && groupdel wazuh
userdel wazuh-indexer && groupdel wazuh-indexer
```

## 2.3. Kritik Soru: Temizliği Nasıl İspatlarım? (Forensics)

Bir sistem mimarı olarak temizliği "ispatlamak" için 3 farklı katmanda kanıt sunman gerekir:

### **A. Ağ Katmanı (Port Analizi)**
Wazuh kuruluyken `1514 (Agent communication)`, `1515 (Enrollment)`, `55000 (API)` ve `9200 (Indexer)` portlarını dinler. 
*   **İspat Komutu:** `ss -tulpn | grep -E '1514|1515|55000|9200'`
*   **Beklenen Sonuç:** Boş çıktı. Eğer çıktı geliyorsa, arkada bir servis hala zombi olarak çalışıyordur.

### **B. Dosya Sistemi Katmanı (Kalıntı Taraması)**
Kurulumdan önce aldığınız dosya listesi ile temizlik sonrası listeyi karşılaştırmalısınız.
*   **İspat Komutu:** `find / -name "*wazuh*" -o -name "*ossec*"`
*   **Analiz:** Bu komut hala bir şeyler döndürüyorsa temizlik başarısızdır. Özellikle `/var/lib/` ve `/var/cache/` dizinlerine dikkat edilmelidir.

### **C. İşlem ve Bellek Katmanı (Process Check)**
*   **İspat Komutu:** `ps aux | grep -iE "wazuh|ossec"`
*   **Analiz:** Çalışan bir daemon kalıp kalmadığını gösterir.

## 2.4. En Güçlü Kanıt: VM Snapshot Diff
Gerçek dünya senaryosunda en "kurşun geçirmez" ispat yöntemi şudur:
1.  **Snapshot A (Temiz):** Kurulumdan hemen önce alınan yedek.
2.  **Snapshot B (Kirli):** Kurulum yapılmış hal.
3.  **Temizlik:** Yukarıdaki adımların uygulanması.
4.  **Kıyas:** `diff` aracıyla dosya sistemindeki farkların (metadata bazlı) kontrol edilmesi.

---
