# Ubuntu 24.04 Sunucu Güvenlik Sıkılaştırma Kılavuzu

Bu kılavuz, yeni kurulan bir Ubuntu 24.04 sunucusunda uygulanması gereken temel ve ileri düzey güvenlik önlemlerini adım adım anlatmaktadır.

---

## İçindekiler

1. [Sistem Güncellemeleri](#1-sistem-güncellemeleri)
2. [Yeni Kullanıcı Oluşturma ve Root Erişimini Kısıtlama](#2-yeni-kullanıcı-oluşturma-ve-root-erişimini-kısıtlama)
3. [SSH Yapılandırması ve Güvenliği](#3-ssh-yapılandırması-ve-güvenliği)
4. [Firewall (UFW) Yapılandırması](#4-firewall-ufw-yapılandırması)
5. [Fail2ban Kurulumu](#5-fail2ban-kurulumu)
6. [Zaman Senkronizasyonu (NTP)](#6-zaman-senkronizasyonu-ntp)
7. [Gereksiz Servislerin Kapatılması](#7-gereksiz-servislerin-kapatılması)
8. [AppArmor Güvenlik Modülü](#8-apparmor-güvenlik-modülü)
9. [Otomatik Güvenlik Güncellemeleri](#9-otomatik-güvenlik-güncellemeleri)
10. [Sysctl Kernel Parametreleri ile Güvenlik](#10-sysctl-kernel-parametreleri-ile-güvenlik)
11. [Log Yönetimi ve İzleme](#11-log-yönetimi-ve-izleme)
12. [Dosya İzinleri ve Özel Dosyaların Korunması](#12-dosya-izinleri-ve-özel-dosyaların-korunması)
13. [Ek Güvenlik Önerileri](#13-ek-güvenlik-önerileri)

---

## 1. Sistem Güncellemeleri

İlk yapılacak işlem, sistemi en son güvenlik yamalarıyla güncellemektir.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

---

## 2. Yeni Kullanıcı Oluşturma ve Root Erişimini Kısıtlama

Root hesabıyla doğrudan oturum açmak güvenlik riski oluşturur. Yeni bir kullanıcı oluşturun ve sudo yetkisi verin.

### 2.1. Yeni Kullanıcı Oluşturma

```bash
sudo adduser guvenli_kullanici
```

Şifre belirleyin ve gerekli bilgileri girin.

### 2.2. Kullanıcıya Sudo Yetkisi Verme

```bash
sudo usermod -aG sudo guvenli_kullanici
```

### 2.3. Root SSH Erişimini Test Etme

Yeni kullanıcı ile SSH bağlantısını test etmeden root erişimini kapatmayın.

```bash
ssh guvenli_kullanici@sunucu_ip_adresi
```

Bağlantı başarılı olduktan sonra bir sonraki adıma geçin.

---

## 3. SSH Yapılandırması ve Güvenliği

SSH, sunucuya uzaktan erişim için kullanılan en kritik servistir.

### 3.1. SSH Yapılandırma Dosyasını Düzenleme

```bash
sudo nano /etc/ssh/sshd_config
```

Aşağıdaki ayarları yapın veya değiştirin:

```conf
# Root girişini kapat
PermitRootLogin no

# Şifre yerine SSH anahtarı kullanımı zorunlu olsun
PasswordAuthentication no
PubkeyAuthentication yes

# Varsayılan portu değiştirin (örnek: 2222)
Port 2222

# Boş şifrelere izin verme
PermitEmptyPasswords no

# X11 yönlendirmeyi kapat
X11Forwarding no

# Belirli kullanıcıların girişine izin ver (isteğe bağlı)
AllowUsers guvenli_kullanici

# Maksimum deneme sayısı
MaxAuthTries 3

# Oturum zaman aşımı (saniye)
ClientAliveInterval 300
ClientAliveCountMax 2
```

### 3.2. SSH Servisini Yeniden Başlatma

```bash
sudo systemctl restart sshd
```

**ÖNEMLİ:** Port değiştirdiyseniz, firewall'da yeni portu açmayı unutmayın ve yeni bağlantıyı test edin.

```bash
ssh -p 2222 guvenli_kullanici@sunucu_ip_adresi
```

---

## 4. Firewall (UFW) Yapılandırması

Ubuntu'nun varsayılan firewall aracı UFW'dir.

### 4.1. UFW Kurulumu ve Temel Ayarlar

```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 4.2. Gerekli Portları Açma

```bash
# SSH (varsayılan veya değiştirilmiş port)
sudo ufw allow 2222/tcp

# HTTP ve HTTPS (web sunucusu varsa)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Diğer gerekli servisler (örnek: MySQL)
# sudo ufw allow 3306/tcp
```

### 4.3. UFW'yi Aktif Etme

```bash
sudo ufw enable
sudo ufw status verbose
```

---

## 5. Fail2ban Kurulumu

Fail2ban, tekrarlayan başarısız giriş denemelerini tespit eder ve IP adreslerini geçici olarak engeller.

### 5.1. Fail2ban Kurulumu

```bash
sudo apt install fail2ban -y
```

### 5.2. Yapılandırma Dosyası Oluşturma

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Aşağıdaki ayarları `[sshd]` bölümünde yapın:

```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

### 5.3. Fail2ban'ı Yeniden Başlatma

```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
sudo fail2ban-client status
```

---

## 6. Zaman Senkronizasyonu (NTP)

Doğru saat, log analizi ve güvenlik protokolleri için kritiktir.

### 6.1. systemd-timesyncd Kontrolü

Ubuntu 24.04'te varsayılan olarak gelir.

```bash
sudo timedatectl status
```

### 6.2. NTP Sunucusu Ayarlama

```bash
sudo timedatectl set-ntp true
sudo timedatectl set-timezone Europe/Istanbul
```

Durumu tekrar kontrol edin:

```bash
timedatectl timesync-status
```

---

## 7. Gereksiz Servislerin Kapatılması

Çalışan gereksiz servisler saldırı yüzeyini genişletir.

### 7.1. Aktif Servisleri Listeleme

```bash
systemctl list-units --type=service --state=running
```

### 7.2. Gereksiz Servisleri Durdurma ve Devre Dışı Bırakma

Örnek olarak cups (yazıcı servisi):

```bash
sudo systemctl stop cups
sudo systemctl disable cups
```

Diğer yaygın gereksiz servisler:
- `bluetooth`
- `avahi-daemon`
- `ModemManager`

**Dikkat:** Hangi servislerin gerekli olduğunu analiz ederek hareket edin.

---

## 8. AppArmor Güvenlik Modülü

AppArmor, uygulamaların sistem kaynaklarına erişimini kısıtlayan bir güvenlik modülüdür.

### 8.1. AppArmor Durumunu Kontrol Etme

```bash
sudo aa-status
```

### 8.2. AppArmor'u Aktif Etme

```bash
sudo systemctl enable apparmor
sudo systemctl start apparmor
```

### 8.3. Profilleri Güncelleme

```bash
sudo apt install apparmor-profiles -y
sudo aa-enforce /etc/apparmor.d/*
```

---

## 9. Otomatik Güvenlik Güncellemeleri

Güvenlik yamalarının otomatik olarak yüklenmesini sağlayın.

### 9.1. Unattended-Upgrades Kurulumu

```bash
sudo apt install unattended-upgrades -y
```

### 9.2. Yapılandırma

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

"Evet" seçeneğini işaretleyin.

### 9.3. Yapılandırmayı Doğrulama

```bash
sudo cat /etc/apt/apt.conf.d/50unattended-upgrades
```

Güvenlik güncellemelerinin aktif olduğundan emin olun:

```conf
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
};
```

---

## 10. Sysctl Kernel Parametreleri ile Güvenlik

Kernel seviyesinde güvenlik önlemleri alın.

### 10.1. Sysctl Yapılandırma Dosyası

```bash
sudo nano /etc/sysctl.d/99-security.conf
```

Aşağıdaki parametreleri ekleyin:

```conf
# IP spoofing koruması
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# ICMP redirect kabul etme
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0

# ICMP gönderme
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# SYN flood koruması
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2

# Kaynak yönlendirme kapalı
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# IPv6 devre dışı (kullanılmıyorsa)
# net.ipv6.conf.all.disable_ipv6 = 1
# net.ipv6.conf.default.disable_ipv6 = 1

# Çekirdek hata mesajlarının loglanması
kernel.printk = 3 3 3 3

# Core dump dosyalarının oluşturulmasını engelle
fs.suid_dumpable = 0

# Randomize virtual address space
kernel.randomize_va_space = 2
```

### 10.2. Ayarları Uygulama

```bash
sudo sysctl --system
```

---

## 11. Log Yönetimi ve İzleme

Loglar, güvenlik olaylarının tespiti için hayati önem taşır.

### 11.1. Log Dosyalarını Kontrol Etme

```bash
ls -la /var/log/
```

Önemli log dosyaları:
- `/var/log/auth.log` - Kimlik doğrulama logları
- `/var/log/syslog` - Sistem logları
- `/var/log/kern.log` - Kernel logları
- `/var/log/ufw.log` - Firewall logları

### 11.2. Logrotate Yapılandırması

Log dosyalarının boyutunu kontrol altında tutun.

```bash
sudo nano /etc/logrotate.d/rsyslog
```

### 11.3. Merkezi Loglama (İsteğe Bağlı)

Büyük ölçekli ortamlarda merkezi log sunucusu düşünün (örn: ELK Stack, Graylog).

---

## 12. Dosya İzinleri ve Özel Dosyaların Korunması

### 12.1. Kritik Dosyaların İzinlerini Kontrol Etme

```bash
# /etc/passwd ve /etc/shadow
ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow

# Beklenen izinler:
# -rw-r--r-- 1 root root /etc/passwd
# -rw-r----- 1 root shadow /etc/shadow
```

### 12.2. World-Writeable Dosyaları Bulma

```bash
sudo find / -type f -perm -002 -exec ls -l {} \;
```

### 12.3. SUID/SGID Bit'i Olan Dosyaları Kontrol Etme

```bash
sudo find / -type f \( -perm -4000 -o -perm -2000 \) -exec ls -l {} \;
```

### 12.4. Önemli Dosyaları Değişmez Yapma (İsteğe Bağlı)

```bash
# Örnek: /etc/passwd dosyasını değiştirilemez yap
sudo chattr +i /etc/passwd
sudo chattr +i /etc/shadow

# Değiştirmek için önce i bitini kaldırın
# sudo chattr -i /etc/passwd
```

**Dikkat:** Bu işlem sistem güncellemelerini etkileyebilir, dikkatli kullanın.

---

## 13. Ek Güvenlik Önerileri

### 13.1. SSH Anahtar Tabanlı Kimlik Doğrulama

Şifre yerine SSH anahtarı kullanın:

```bash
# Yerel makinede anahtar oluşturun
ssh-keygen -t ed25519 -C "your_email@example.com"

# Anahtarı sunucuya kopyalayın
ssh-copy-id -p 2222 guvenli_kullanici@sunucu_ip_adresi
```

### 13.2. İki Faktörlü Kimlik Doğrulama (2FA)

Google Authenticator ile 2FA ekleyin:

```bash
sudo apt install libpam-google-authenticator -y
google-authenticator
```

SSH yapılandırmasında şu satırı ekleyin:

```conf
AuthenticationMethods publickey,keyboard-interactive
```

### 13.3. Düzenli Güvenlik Taramaları

#### Lynis Güvenlik Denetimi

```bash
sudo apt install lynis -y
sudo lynis audit system
```

#### Rkhunter (Rootkit Hunter)

```bash
sudo apt install rkhunter -y
sudo rkhunter --check
```

### 13.4. Ağ Dinleme Portlarını Kontrol Etme

```bash
sudo ss -tulpn
# veya
sudo netstat -tulpn
```

### 13.5. Düzenli Yedekleme Stratejisi

- Sistem yapılandırma dosyalarını yedekleyin
- Otomatik yedekleme scriptleri oluşturun
- Yedekleri farklı bir lokasyonda saklayın

### 13.6. Güvenlik Duyurularını Takip Etme

- [Ubuntu Security Announcements](https://ubuntu.com/security/notices)
- [CVE Database](https://cve.mitre.org/)

---

## Son Kontrol Listesi

- [ ] Sistem güncellemeleri yapıldı
- [ ] Yeni kullanıcı oluşturuldu ve sudo yetkisi verildi
- [ ] Root SSH erişimi kapatıldı
- [ ] SSH portu değiştirildi ve anahtar tabanlı kimlik doğrulama aktif
- [ ] UFW firewall yapılandırıldı ve aktif
- [ ] Fail2ban kuruldu ve yapılandırıldı
- [ ] Zaman senkronizasyonu aktif
- [ ] Gereksiz servisler kapatıldı
- [ ] AppArmor aktif
- [ ] Otomatik güvenlik güncellemeleri aktif
- [ ] Sysctl güvenlik parametreleri uygulandı
- [ ] Log yönetimi yapılandırıldı
- [ ] Kritik dosya izinleri kontrol edildi
- [ ] Güvenlik tarama araçları kuruldu (Lynis, Rkhunter)

---

## Faydalı Komutlar

```bash
# Sistem bilgisi
uname -a
lsb_release -a

# Aktif kullanıcılar
who
last

# Sistem kaynakları
top
htop
df -h
free -m

# Firewall durumu
sudo ufw status verbose

# Fail2ban durumu
sudo fail2ban-client status

# AppArmor durumu
sudo aa-status

# Dinleme portları
sudo ss -tulpn

# Başarısız giriş denemeleri
sudo grep "Failed password" /var/log/auth.log | tail -20

# Güvenlik güncellemelerini kontrol
sudo unattended-upgrade --dry-run --debug
```

---

## Sonuç

Bu kılavuzda belirtilen adımlar, Ubuntu 24.04 sunucunuzun temel güvenlik seviyesini önemli ölçüde artıracaktır. Ancak unutmayın ki güvenlik tek seferlik bir işlem değil, sürekli bir süreçtir. Düzenli olarak:

- Sistem güncellemelerini kontrol edin
- Logları inceleyin
- Güvenlik taramaları yapın
- Yapılandırmaları gözden geçirin
- Yeni güvenlik tehditlerini takip edin

**Önemli Not:** Her ortam farklıdır. Bu kılavuzdaki ayarları kendi kullanım senaryonuza göre özelleştirin. Değişiklik yapmadan önce mutlaka yedek alın ve değişiklikleri test ortamında deneyin.

---

*Son Güncelleme: $(date +%Y-%m-%d)*
*Ubuntu Sürümü: 24.04 LTS (Noble Numbat)*
