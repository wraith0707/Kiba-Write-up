# Kiba - Write-up

* **Platform:** TryHackMe
* **Zorluk Seviyesi:** Easy
* **Makale Amacı:** Bu makinede Nmap ile servis keşfi, Elasticsearch/Kibana üzerinden RCE (CVE-2019-7609) zafiyeti ile sisteme sızma ve Linux Capabilities (`cap_setuid`) kullanarak root yetkisi yükseltme adımları gerçekleştirilmiştir.

---

## 1. Keşif (Reconnaissance)

Hedef IP adresine yönelik gerçekleştirilen ilk Nmap taraması ile açık portlar ve servis sürümleri tespit edilmiştir.

* **Komut:** `nmap -p- -v -T4 10.113.134.222`
* **Açık Portlar:**
  * **Port 22 (SSH):** OpenSSH
  * **Port 80 (HTTP):** Apache httpd
  * **Port 5044 (lxi-evntsvc):** Aktif servis
  * **Port 5601 (esmagent):** Elasticsearch Kibana

| Nmap Taraması | Açıklama |
| :--- | :--- |
|<img width="1479" height="943" alt="1_3" src="https://github.com/user-attachments/assets/b92afa16-6e1b-489f-b2e2-f0d206e5a216" /> | Genel port taraması sonuçları |
|<img width="1479" height="943" alt="2_2" src="https://github.com/user-attachments/assets/b8319496-21da-4156-bfe9-6b5260bd456e" /> | Servis versiyon tespiti (`-sC -sV`) |

---

## 2. Servis Analizi & Bilgi Toplama (Enumeration)

* Port 5601 üzerinde çalışan Kibana servisine tarayıcı üzerinden erişilerek versiyon bilgisi `6.5.4` olarak tespit edilmiştir.
* Yapılan araştırmalar neticesinde bu versiyonun `CVE-2019-7609` kodlu RCE zafiyetine sahip olduğu görülmüştür.

| Kibana Arayüzü ve Zafiyet Tespiti | Açıklama |
| :--- | :--- |
|<img width="1479" height="943" alt="3_3" src="https://github.com/user-attachments/assets/f194963e-7adf-4138-a61b-12f113931463" /> | Kibana Management paneli ve sürüm bilgisi (6.5.4) |
|<img width="1479" height="943" alt="4_3" src="https://github.com/user-attachments/assets/3e962862-51dc-4ca1-9f4d-d5bd021e8f5a" /> | Zafiyet kodu tespiti (CVE-2019-7609) |

---

## 3. Sömürü (Exploitation / Foothold)

* `CVE-2019-7609` python exploit betiği kullanılarak hedef Kibana servisine RCE saldırısı başlatılmıştır.
* Saldırgan makinede Netcat ile dinleme (`nc`) açılmış ve başarılı bir şekilde reverse shell alınmıştır.
* Sistemde kullanıcı bayrağı (`user.txt`) okunmuştur.

| Sömürü Adımları | Açıklama |
| :--- | :--- |
|<img width="1479" height="943" alt="5_3" src="https://github.com/user-attachments/assets/abb18938-8079-49f5-a97c-2cf54b886ae2" /> | Python exploit ile RCE tetiklenmesi |
|<img width="1479" height="943" alt="6_3" src="https://github.com/user-attachments/assets/6d8c4406-1dc6-4a7d-81cf-5a52fe26b25b" /> | Netcat üzerinden bağlantının yakalanması |
|<img width="1479" height="943" alt="8_3" src="https://github.com/user-attachments/assets/bccef494-d916-4565-ae38-22b3e8fb2229" /> | Kullanıcı bayrağının (`user.txt`) okunması |

---

## 4. Yetki Yükseltme (Privilege Escalation)

* Sistemdeki yetki zafiyetlerini bulmak için `getcap -r / 2>/dev/null` komutu çalıştırılmıştır.
* `/home/kiba/.hackmeplease/python3` dosyasında `cap_setuid+ep` yetkisinin bulunduğu tespit edilmiştir.
* Bu yetki kullanılarak python üzerinden UID 0 (root) set edilmiş ve `/bin/bash` çağrılarak root erişimi sağlanmıştır.
* `root.txt` dosyasına ulaşılarak bayrak elde edilmiştir.

| Yetki Yükseltme Adımları | Açıklama |
| :--- | :--- |
|<img width="1479" height="943" alt="7_3" src="https://github.com/user-attachments/assets/905f5356-4538-4e84-b8a9-58b72793f1dc" /> | `getcap` ile Capabilities taraması |
|<img width="1479" height="943" alt="9_3" src="https://github.com/user-attachments/assets/0feca5b8-c91b-428e-914f-c0a0f3f168e4" /> | Python ile root yetkisine geçiş ve `root.txt` okuma |
