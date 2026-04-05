# Proje Analizi - Adım 5: Kaynak Kod ve Akış Analizi (Threat Modeling)

**Analiz Konusu:** Wazuh API ve Manager Mimarisinde Güvenlik Akışı  
**Kritik Bileşenler:** `wazuh-apid` (Python), `ossec-authd` (C)

## 5.1. Başlangıç Noktası (Entrypoint) Tespiti
Wazuh devasa bir yapı olduğu için tek bir giriş noktası yoktur, ancak bir güvenlik uzmanı için en kritik "Entrypoint" **Wazuh API**'dir.
*   **API Girişi:** `framework/wazuh/core/cluster/control.py` ve ilgili API rotaları üzerinden sistem yönetilir.
*   **Agent Girişi:** `src/ossec_auth/main.c` (ossec-authd). Yeni bir bilgisayar (agent) ağa dahil olduğunda ilk bu servis ile konuşur.

## 5.2. Kimlik Doğrulama (Authentication) Mekanizması
Wazuh, modern bir mimari kullanarak **JWT (JSON Web Token)** tabanlı bir kimlik doğrulama sistemi uygular.

1.  **Talep:** Kullanıcı veya Dashboard, `/security/user/authenticate` uç noktasına (endpoint) kullanıcı adı ve şifre gönderir.
2.  **Doğrulama:** API, bu bilgileri kendi dahili veritabanı (RBAC - Role Based Access Control) ile karşılaştırır.
3.  **Token:** Eğer bilgiler doğruysa, kısa süreli (varsayılan 15 dk) bir **JWT** üretilir.
4.  **Yetki:** Bundan sonraki tüm isteklerde bu token `Authorization: Bearer <TOKEN>` başlığıyla gönderilir.

## 5.3. Kritik Soru: Bir Hacker Neyi, Nasıl Çalar?

### **A. Veri Keşfi (What can be stolen?)**
Bir saldırgan Wazuh kaynak koduna baktığında şu "hazinelere" odaklanır:
*   **Agent Keys:** `/var/ossec/etc/client.keys` dosyasında tutulan anahtarlar. Eğer bir saldırgan bu anahtarları ele geçirirse, ağdaki tüm agent'ların trafiğini taklit edebilir veya izleyebilir.
*   **Log Data:** SIEM içinde tutulan tüm sistem logları. Bu loglar içinde unutulmuş parolalar, sistem zafiyetleri ve ağ haritası bulunur.
*   **API Credentials:** Yapılandırma dosyalarındaki zayıf parolalar.

### **B. Saldırı Senaryosu: Auth Mekanizmasına Saldırı**
Bulduğumuz JWT tabanlı sisteme karşı şu saldırılar kurgulanabilir:

1.  **JWT Bypassing / Weak Secret:** Eğer Wazuh kurulumu sırasında JWT imzalamak için kullanılan "secret key" zayıfsa veya varsayılan bırakılmışsa, saldırgan kendi admin token'ını imzalayabilir.
2.  **Brute Force (API):** `/security/user/authenticate` uç noktasında "Rate Limiting" (istek sınırlama) yoksa, saniyede binlerce şifre denenebilir.
3.  **Insecure Direct Object Reference (IDOR):** Bir kullanıcı kendi yetkisi olmayan bir agent'ın verisini çekmek için API parametrelerini (örneğin `agent_id`) değiştirebilir.

## 5.4. Tehdit Modellemesi (STRIDE Metodu)
Wazuh özelinde hazırladığım mini tehdit tablosu:

| Tehdit Türü | Risk | Açıklama |
| :--- | :--- | :--- |
| **Spoofing** | Yüksek | Sahte bir agent gibi davranarak SIEM'e yanlış veri göndermek. |
| **Tampering** | Orta | `/var/ossec/etc/ossec.conf` dosyasını değiştirip log kaydını durdurmak. |
| **Information Disclosure** | Çok Yüksek | API üzerinden tüm ağın zafiyet tarama sonuçlarını sızdırmak. |
| **Denial of Service** | Orta | Manager servisine aşırı yüklenerek (flood) tehdit algılamayı devre dışı bırakmak. |

---
