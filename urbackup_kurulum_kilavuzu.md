# UrBackup Uçtan Uca Kurulum Kılavuzu

Bu kılavuz, Ubuntu 24.04 sunucusuna UrBackup yedekleme sunucusunun kurulmasını ve ardından Windows ile Ubuntu istemcilerinin (agent) yapılandırılmasını adım adım anlatır.

---

## İçindekiler

1. [Gereksinimler](#1-gereksinimler)
2. [UrBackup Sunucu Kurulumu (Ubuntu 24.04)](#2-urbackup-sunucu-kurulumu-ubuntu-2404)
3. [UrBackup Sunucu Yapılandırması](#3-urbackup-sunucu-yapılandırması)
4. [Windows İstemci Kurulumu](#4-windows-istemci-kurulumu)
5. [Ubuntu İstemci Kurulumu](#5-ubuntu-istemci-kurulumu)
6. [Yedekleme ve Geri Yükleme İşlemleri](#6-yedekleme-ve-geri-yükleme-işlemleri)
7. [Sorun Giderme](#7-sorun-giderme)

---

## 1. Gereksinimler

### Sunucu Gereksinimleri
- **İşletim Sistemi:** Ubuntu 24.04 LTS
- **RAM:** Minimum 2 GB (önerilen: 4 GB+)
- **Disk Alanı:** Yedeklenecek veri miktarına göre yeterli boş alan
- **Ağ:** Statik IP adresi önerilir

### İstemci Gereksinimleri
- **Windows:** Windows 7/8/10/11 veya Windows Server 2012+
- **Ubuntu:** 18.04 veya üzeri sürümler

---

## 2. UrBackup Sunucu Kurulumu (Ubuntu 24.04)

### Adım 2.1: Sistem Güncellemesi

```bash
sudo apt update
sudo apt upgrade -y
```

### Adım 2.2: Gerekli Paketlerin Kurulumu

```bash
sudo apt install -y software-properties-common wget curl gnupg2
```

### Adım 2.3: UrBackup Deposunun Eklenmesi

UrBackup'ın resmi deposunu ekleyin:

```bash
# GPG anahtarını indir ve ekle
wget -O - https://hpi.obnubil.de/public/urbackup.key | sudo gpg --dearmor -o /usr/share/keyrings/urbackup-archive-keyring.gpg

# Depoyu sources listesine ekle
echo "deb [signed-by=/usr/share/keyrings/urbackup-archive-keyring.gpg] https://hpi.obnubil.de/deb/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/urbackup.list > /dev/null
```

### Adım 2.4: UrBackup Sunucusunun Kurulumu

```bash
sudo apt update
sudo apt install -y urbackup-server
```

Kurulum sırasında size aşağıdaki sorular sorulabilir:
- **Web arayüzü portu:** Varsayılan 55414 (değiştirmeyin)
- **HTTP/HTTPS seçimi:** Başlangıç için HTTP yeterlidir

### Adım 2.5: Servisin Kontrolü

UrBackup servisinin çalıştığından emin olun:

```bash
sudo systemctl status urbackupserver
sudo systemctl enable urbackupserver
```

Servis çalışmıyorsa başlatın:

```bash
sudo systemctl start urbackupserver
```

### Adım 2.6: Güvenlik Duvarı Ayarları

UFW kullanıyorsanız gerekli portları açın:

```bash
sudo ufw allow 55414/tcp    # Web arayüzü
sudo ufw allow 55413/tcp    # İstemci iletişimi
sudo ufw allow 55414/udp    # İstemci keşfi
sudo ufw reload
```

---

## 3. UrBackup Sunucu Yapılandırması

### Adım 3.1: Web Arayüzüne Erişim

Tarayıcınızda şu adresi açın:
```
http://<sunucu-ip-adresi>:55414
```

Örnek: `http://192.168.1.100:55414`

**Not:** İlk girişte kullanıcı adı ve şifre oluşturmanız istenir.

### Adım 3.2: Temel Ayarlar

1. **Settings** menüsüne gidin
2. **General** sekmesinde:
   - **Internet name or IP:** Sunucunuzun IP adresini girin
   - **Max number of backups:** İhtiyaca göre ayarlayın (örn: 10)
   
3. **Backup Storage** sekmesinde:
   - **Backup storage folder:** Yedeklerin saklanacağı dizin (varsayılan: `/var/urbackup`)
   - Disk alanınızı kontrol edin: `df -h`

4. **File Backup** sekmesinde:
   - **Don't do file backups:** İşaretli değilse dosya yedeklemesi aktif olur
   - **Interval:** Yedekleme sıklığını ayarlayın (örn: her 6 saatte bir)

5. **Image Backups** sekmesinde:
   - **Don't do image backups:** Tüm disk imajı almak istemiyorsanız işaretleyin
   - Sadece belirli bölümlerin imajını almak için özelleştirme yapabilirsiniz

### Adım 3.3: Yedekleme Dizini İçin Özel Bölüm (Opsiyonel)

Büyük yedekler için ayrı bir disk bölümü önerilir:

```bash
# Yeni disk ekledikten sonra
sudo mkdir -p /backup/urbackup
sudo chown -R urbackup:urbackup /backup/urbackup
```

Sonra web arayüzünden yedekleme dizinini `/backup/urbackup` olarak değiştirin.

---

## 4. Windows İstemci Kurulumu

### Adım 4.1: İstemci Yazılımının İndirilmesi

1. UrBackup sunucu web arayüzüne gidin
2. Sol menüden **Clients** sekmesine tıklayın
3. **Download client for Windows** bağlantısına tıklayın
4. Alternatif olarak: https://www.urbackup.org/download.html adresinden indirebilirsiniz

### Adım 4.2: Kurulum

1. İndirdiğiniz `UrBackup_Client.exe` dosyasını çalıştırın
2. Kurulum sihirbazını takip edin:
   - Lisans sözleşmesini kabul edin
   - Kurulum dizinini seçin (varsayılan önerilir)
   - Başlat menüsü kısayollarını oluşturun

### Adım 4.3: İstemci Yapılandırması

Kurulum tamamlandıktan sonra:

1. Sistem tepsisindeki UrBackup simgesine sağ tıklayın
2. **Settings** seçeneğine tıklayın

#### Önemli Ayarlar:

- **Server URL:** `http://<sunucu-ip-adresi>:55414`
  - Örnek: `http://192.168.1.100:55414`
  
- **Client name:** İstemci için bir isim girin (örn: WIN-PC01)

- **File backup:** 
  - Yedeklenecek klasörleri ekleyin (örn: C:\Users, C:\ImportantData)
  - Hariç tutulacak klasörleri belirtin (örn: Temp dosyaları)

- **Image backup:**
  - Hangi sürücülerin imajının alınacağını seçin
  - Genellikle C: sürücüsü seçilir

3. **OK** butonuna tıklayarak kaydedin

### Adım 4.4: İlk Yedekleme

1. Sistem tepsisindeki simgeye sağ tıklayın
2. **Start backup now** seçeneğine tıklayın
3. Sunucu web arayüzünden ilerlemeyi takip edin

---

## 5. Ubuntu İstemci Kurulumu

### Adım 5.1: UrBackup İstemci Kurulumu

```bash
# GPG anahtarını indir ve ekle (sunucu ile aynı)
wget -O - https://hpi.obnubil.de/public/urbackup.key | sudo gpg --dearmor -o /usr/share/keyrings/urbackup-archive-keyring.gpg

# Depoyu ekleyin
echo "deb [signed-by=/usr/share/keyrings/urbackup-archive-keyring.gpg] https://hpi.obnubil.de/deb/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/urbackup.list > /dev/null

# Paket listesini güncelleyin ve istemciyi kurun
sudo apt update
sudo apt install -y urbackup-client
```

### Adım 5.2: İstemci Yapılandırması

Yapılandırma dosyasını düzenleyin:

```bash
sudo nano /etc/urbackup/urclient.conf
```

Aşağıdaki ayarları yapın:

```ini
# Sunucu adresi
server_url=http://<sunucu-ip-adresi>:55414

# İstemci adı (opsiyonel, boş bırakılırsa hostname kullanılır)
client_name=UBUNTU-CLIENT01

# Yedeklenecek dizinler (virgülle ayırın)
backuppath=/home,/etc,/var/www

# Log seviyesi
log_level=warning
```

**Not:** `server_url` kısmındaki `<sunucu-ip-adresi>` yerine gerçek sunucu IP'nizi yazın.

### Adım 5.3: Servisi Başlatma

```bash
sudo systemctl enable urbackupclient
sudo systemctl start urbackupclient
sudo systemctl status urbackupclient
```

### Adım 5.4: Güvenlik Duvarı (Gerekirse)

Eğer UFW kullanıyorsanız:

```bash
sudo ufw allow out to any port 55414
sudo ufw allow out to any port 55413
```

### Adım 5.5: Manuel Yedekleme Başlatma

```bash
# Dosya yedeklemesi başlat
sudo /usr/share/urbackup/bin/urbackupclient_backup --backup_type file

# İmaj yedeklemesi başlat (tüm disk)
sudo /usr/share/urbackup/bin/urbackupclient_backup --backup_type image
```

---

## 6. Yedekleme ve Geri Yükleme İşlemleri

### Adım 6.1: Sunucu Üzerinden İzleme

1. Web arayüzünde **Status** sekmesine gidin
2. Bağlı istemcileri ve son yedeklemeleri görüntüleyin
3. **Logs** sekmesinden detaylı logları inceleyin

### Adım 6.2: Dosya Geri Yükleme (Web Arayüzü)

1. **Backups** sekmesine gidin
2. İlgili istemciyi seçin
3. Tarih ve saat seçin
4. Geri yüklemek istediğiniz dosyaları seçin
5. **Restore** butonuna tıklayın
6. Geri yükleme yöntemi seçin:
   - **Download:** Dosyaları tar olarak indir
   - **Restore to client:** Doğrudan istemciye geri yükle

### Adım 6.3: İmaj Geri Yükleme

#### Yöntem 1: UrBackup Kurtarma Ortamı

1. Web arayüzünden **Download rescue system** seçeneği ile ISO indirin
2. ISO'yu USB'ye yazdırın (Rufus veya Etcher kullanın)
3. Bilgisayarı USB'den başlatın
4. UrBackup kurtarma ortamı otomatik olarak sunucuya bağlanır
5. Geri yüklenecek imajı seçin ve işlemi başlatın

#### Yöntem 2: Windows'ta Geri Yükleme

1. UrBackup Windows istemcisini açın
2. **Restore** sekmesine gidin
3. Mevcut imaj yedeklerini görüntüleyin
4. Geri yüklemek istediğiniz yedeği seçin
5. İşlemi onaylayın (sistem yeniden başlatılabilir)

### Adım 6.4: Komut Satırından Geri Yükleme (Ubuntu)

```bash
# Mevcut yedekleri listele
sudo /usr/share/urbackup/bin/urbackupclient_list

# Dosya geri yükleme
sudo /usr/share/urbackup/bin/urbackupclient_restore \
  --backup_id <ID> \
  --filename /path/to/file \
  --restore_to /path/to/destination
```

---

## 7. Sorun Giderme

### Problem 1: İstemci Sunucuya Bağlanamıyor

**Çözüm:**
```bash
# Sunucuda portları kontrol edin
sudo netstat -tlnp | grep 55414
sudo netstat -tlnp | grep 55413

# Güvenlik duvarını kontrol edin
sudo ufw status

# İstemciden sunucuya erişimi test edin
telnet <sunucu-ip> 55414
```

### Problem 2: Yedekleme Çok Yavaş

**Çözüm:**
- Ağ bant genişliğini kontrol edin
- Yedekleme zamanlamasını gece saatlerine ayarlayın
- Hariç tutulan dosya türlerini artırın (video, iso vb.)
- Compression ayarlarını gözden geçirin

### Problem 3: Disk Alanı Yetersiz

**Çözüm:**
```bash
# Kullanılan alanı kontrol edin
du -sh /var/urbackup/*

# Eski yedekleri temizle (web arayüzünden)
# veya manuel olarak:
sudo /usr/share/urbackup/bin/urbackupserver_cleanup
```

### Problem 4: Ubuntu İstemci Başlamıyor

**Çözüm:**
```bash
# Logları kontrol edin
sudo journalctl -u urbackupclient -f

# Yapılandırmayı doğrulayın
sudo cat /etc/urbackup/urclient.conf

# Servisi yeniden başlatın
sudo systemctl restart urbackupclient
```

### Problem 5: Windows İstemci Görünmüyor

**Çözüm:**
1. Windows güvenlik duvarında UrBackup'a izin verin
2. İstemci ayarlarında server_url'i kontrol edin
3. İstemci hizmetini yeniden başlatın:
   ```powershell
   Restart-Service UrBackupClient
   ```

---

## Ek: Otomasyon ve Best Practices

### Yedekleme Zamanlaması

Web arayüzünden her istemci için özel zamanlama yapabilirsiniz:
- **File backup:** Her 6 saatte bir
- **Image backup:** Haftada bir (örn: Pazar gecesi)

### Bildirim Ayararı

**Settings > Mail settings** bölümünden:
- SMTP sunucu bilgilerini girin
- Yedekleme başarısız olduğunda e-posta bildirimi alın

### İzleme ve Loglama

```bash
# Sunucu logları
sudo tail -f /var/log/urbackup/urbackup.log

# İstemci logları (Ubuntu)
sudo tail -f /var/log/urbackup/urclient.log

# İstemci logları (Windows)
# C:\Program Files\UrBackup\urclient.log
```

### Performans İpuçları

1. **SSD Kullanımı:** Yedekleme dizini için SSD kullanın
2. **Ağ:** Gigabit Ethernet veya daha hızlı ağ kullanın
3. **Zamanlama:** Yoğun saatler dışında yedekleme yapın
4. **Incremental Yedekleme:** Varsayılan olarak aktiftir, değiştirmeyin

---

## Sonuç

Bu kılavuz ile Ubuntu 24.04 sunucusuna UrBackup kurulumunu tamamladınız ve Windows/Ubuntu istemcilerini yapılandırdınız. UrBackup, hem dosya hem de tam disk imaj yedeklemesi destekleyen güçlü bir açık kaynak çözümdür.

**Önemli Hatırlatmalar:**
- Düzenli olarak yedekleme testleri yapın
- Yedekleri farklı bir fiziksel konumda saklayın (3-2-1 kuralı)
- Logları periyodik olarak kontrol edin
- UrBackup sunucusunun kendisini de yedekleyin

Daha fazla bilgi için: https://www.urbackup.org/documentation.html
