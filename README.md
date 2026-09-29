# Kiba - Write-up

* **Platform:** TryHackMe
* **Zorluk Seviyesi:** Easy
* **Makale Amacı:** Bu makinede Nmap ile servis keşfi, Elasticsearch/Kibana üzerinde CVE-2019-7609 (RCE) zafiyetinin sömürülmesi ve Linux Capabilities (`cap_setuid+ep`) kullanılarak root yetkisi yükseltme adımları gerçekleştirilmiştir.

---

## 1. Keşif (Reconnaissance)

Hedef IP adresine yönelik gerçekleştirilen kapsamlı Nmap taraması ile açık portlar ve servis sürümleri tespit edilmiştir.

* **Komut:** `nmap -p- -v -T4 10.113.134.222`[cite: 25]
* **Açık Portlar:**
  * **Port 22 (tcp):** ssh[cite: 25]
  * **Port 80 (tcp):** http[cite: 25]
  * **Port 5044 (tcp):** lxi-evntsvc[cite: 25]
  * **Port 5601 (tcp):** esmagent (Kibana web arayüzü)[cite: 25, 26]

<img width="1250" height="916" alt="1" src="https://github.com/user-attachments/assets/bdd17ce7-59c6-4f32-bda4-6f32cee75b2e" />


* **Detaylı Servis Taraması:** 5044 ve 5601 portlarına yönelik çalıştırılan servis versiyon taramasında Kibana 6.5.4 sürümü tespit edilmiştir[cite: 26].

<img width="1716" height="483" alt="2" src="https://github.com/user-attachments/assets/7b1e01ee-8c5c-4335-bda1-5ef2b21ce8e5" />


---

## 2. Servis Analizi & Bilgi Toplama (Enumeration)

* Tarayıcı üzerinden `http://10.113.134.222:5601` adresine gidildiğinde Kibana yönetim paneli ile karşılaşılmıştır[cite: 27].
* Yönetim panelinde sürüm numarasının **6.5.4** olduğu doğrulanmıştır[cite: 27].

<img width="1912" height="817" alt="3" src="https://github.com/user-attachments/assets/72391baa-374e-44bd-9d17-1824b6133bd0" />


---

## 3. Sömürü (Exploitation / Foothold)

* Tespit edilen Kibana 6.5.4 sürümünün **CVE-2019-7609** (Timelion RCE) zafiyetine karşı savunmasız olduğu belirlenmiştir[cite: 28, 33].
* Saldırgan makine üzerinde netcat dinlemeye alınmış ve exploit betiği çalıştırılarak sisteme ters bağlantı (reverse shell) sağlanmıştır[cite: 28, 29].
* Bağlantı sonrası `kiba` kullanıcısı olarak sistem oturumu açılmış ve `user.txt` bayrağı okunmuştur[cite: 29, 32].

| Sömürü Adımları | Açıklama |
| :--- | :--- |
|<img width="1699" height="265" alt="5" src="https://github.com/user-attachments/assets/765ca6d8-75a7-49f6-a033-fdfe1d671fae" />
| CVE-2019-7609 exploit scriptinin çalıştırılması[cite: 28] |
|<img width="1376" height="380" alt="6" src="https://github.com/user-attachments/assets/f58d98dd-b2ec-4fc5-86fe-27ad521f100a" />
| Netcat ile reverse shell alınması (`kiba@ubuntu`)[cite: 29] |
|<img width="1178" height="312" alt="7" src="https://github.com/user-attachments/assets/94954972-100b-477a-b975-1c273687b7a4" />
| Kullanıcı bayrağının okunması (`THM{1s_easy_pwn3d_k1bana_w1th_rce}`)[cite: 32] |

---

## 4. Yetki Yükseltme (Privilege Escalation)

* `sudo -l` komutu denendiğinde tty kısıtlaması ile karşılaşılmıştır[cite: 32].
* Sistem genelinde yetki yükseltme vektörlerini bulmak için `getcap -r / 2>/dev/null` komutu çalıştırılmıştır[cite: 31].
* Yapılan tarama sonucunda `/home/kiba/.hackmeplease/python3` dosyasında `cap_setuid+ep` yeteneği (capability) bulunduğu tespit edilmiştir[cite: 31].
* Bu yetki sayesinde Python binary'si doğrudan root yetkileriyle komut çalıştırabilecek kapasiteye sahiptir.
* İlgili dizine geçilerek yetki yükseltme komutu tetiklenmiş ve root erişimi sağlanarak `root.txt` dosyasına ulaşılmıştır[cite: 30].

| Yetki Yükseltme Adımları | Açıklama |
| :--- | :--- |
|<img width="921" height="210" alt="8" src="https://github.com/user-attachments/assets/643a16c5-26ed-41d3-ab93-9ced3bfd6ff6" />
| `getcap` ile Linux Capabilities taraması[cite: 31] |
|<img width="1900" height="485" alt="9" src="https://github.com/user-attachments/assets/feee0e28-238e-4bb1-8a56-6c597adb981f" />
| Python üzerinden root yetkisi alınması ve bayrağa erişim (`THM{pr1v1lege_escalat1on_us1ng_capab1l1t1es}`)[cite: 30] |
