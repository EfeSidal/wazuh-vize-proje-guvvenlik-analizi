# 🛡️ Wazuh Open Source Analysis: A Security Expert Perspective

Bu proje, dünyanın en kapsamlı açık kaynak SIEM/XDR platformlarından biri olan **Wazuh**'un; kurulumundan kaynak koduna, CI/CD süreçlerinden Docker mimarisine kadar bir **"Güvenlik Uzmanı ve Sistem Mimarı"** gözüyle derinlemesine analizini içermektedir.

> **Not:** Bu çalışma, İstinye Üniversitesi Bilişim Güvenliği Teknolojisi vize projesi kapsamında "Gerçek Dünya Açık Kaynak Repo Analizi" görevlerini yerine getirmek amacıyla hazırlanmıştır.

-----

## 📑 Analiz Kapsamı

Proje 5 temel aşamadan oluşmaktadır. Her aşama, sistemin farklı bir güvenlik katmanını ve mimari yapısını ele alır:

| Aşama | Konu | Odak Noktası |
| :--- | :--- | :--- |
| **Adım 1** | [Kurulum Analizi](https://www.google.com/search?q=./adim1_kurulum.md) | `install.sh` Tersine Mühendisliği ve Tedarik Zinciri Güvenliği |
| **Adım 2** | [İzolasyon ve Temizlik](https://www.google.com/search?q=./adim2_temizlik.md) | Adli Bilişim (Forensics) Bakış Açısıyla İz Bırakmadan Kaldırma |
| **Adım 3** | [CI/CD Pipeline Analizi](https://www.google.com/search?q=./adim3_cicd.md) | GitHub Actions İş Akışları ve Webhook Mekanizmaları |
| **Adım 4** | [Docker ve Konteyner Güvenliği](https://www.google.com/search?q=./adim4_docker.md) | Katmanlı Mimari ve Host İzolasyon Riskleri |
| **Adım 5** | [Tehdit Modellemesi (STRIDE)](https://www.google.com/search?q=./adim5_threat_modeling.md) | Kaynak Kod Analizi, Auth Mekanizması ve Saldırı Vektörleri |

-----

## 🚀 Öne Çıkan Bulgular (Executive Summary)

Analiz süreci boyunca Wazuh ekosisteminde tespit edilen kritik mimari noktalar:

  * **Tedarik Zinciri Riski:** Kurulum scriptlerinin (`curl | bash`) indirme aşamasında SHA256 kontrolü yapmaması, potansiyel bir zafiyet noktası olarak not edilmiştir.
  * **İmza Doğrulama:** Paket yönetiminde GPG anahtarlarının aktif kullanımı, yazılım bütünlüğü açısından güçlü bir savunma hattı oluşturmaktadır.
  * **Auth Mimarisi:** Sistemde modern **JWT (JSON Web Token)** tabanlı kimlik doğrulama kullanılmakta olup, RBAC (Rol Tabanlı Erişim Kontrolü) ile yetkilendirme katmanlandırılmıştır.
  * **Docker Hardening:** Varsayılan Docker yapılandırmasının "Privilege Escalation" risklerine karşı kısıtlanması gerektiği saptanmıştır.

-----

## 🛠️ Kullanılan Araçlar ve Metodoloji

Analiz sırasında aşağıdaki araçlar ve teknikler kullanılmıştır:

  * **Ortam:** Ubuntu Server (VMware/VirtualBox) üzerinde izole snapshot yönetimi.
  * **Analiz Araçları:** `ss`, `ps`, `find`, `diff`, `grep`, `openssl`.
  * **Metodoloji:** STRIDE Tehdit Modellemesi ve Dinamik/Statik Analiz yöntemleri.

-----

## 👨‍💻 Hazırlayan

  * **İsim:** Efe Sidal
  * **Bölüm:** Bilişim Güvenliği Teknolojisi
  * **Üniversite:** İstinye Üniversitesi

-----

### 📜 Lisans ve Uyarı

Bu projedeki analizler eğitim amaçlıdır. Wazuh, Wazuh Inc. şirketine ait tescilli bir ticari markadır. Bu rapordaki bulgular sistemin güncel sürümüne göre değişiklik gösterebilir.

-----
