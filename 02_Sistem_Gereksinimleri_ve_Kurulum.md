# Bölüm 2: Sistem Gereksinimleri ve Kurulum (2026 Standartları)

BigBlueButton v3.x serisini (BBB) 2026 güncel BT standartlarında verimli koşturabilmek için güçlü bir donanıma veya sanal sunucuya (VPS/Cloud) ihtiyaç vardır. Video şifreleme/çözme, eş zamanlı veri aktarımı (ses/video/ekran/sunum) yoğun CPU ve RAM gerektirir.

## 2.1 Sunucu Gereksinimleri (Minimum ve Önerilen)

> [!WARNING]
> BBB, Swap alanı kapalı (Disabled Swap) olacak şekilde tasarlanmamıştır ancak performans için yeterli RAM'e sahip olunmalıdır ve veritabanı yedeği, temp dosyaları vb. için hızlı bir disk (NVMe/SSD) zorunludur.

| Bileşen | Minimum (Test Formatı - < 50 Kullanıcı) | Önerilen (Prod. / > 150 Kullanıcı için) |
| --- | --- | --- |
| CPU | 8 Core | 16 Core (Mümkünse High-Frequency) |
| RAM | 16 GB | 32 GB |
| Disk | 50 GB SSD | 500 GB NVMe / SSD (Kayıtlar için ek alan önerilir) |
| İşletim Sistemi | Ubuntu 22.04 LTS (BBB 3.0+) | Ubuntu 22.04 LTS (64-bit) |
| Ağ Bağlantısı | 500 Mbps Simetrik | 1 Gbps veya 10 Gbps Simetrik |

## 2.2 Ön Hazırlıklar

### 1. FQDN (Domain) Tanımlaması
BigBlueButton IP adresi ile güvenli çalışmaz (Mikrofon/Kamera onayı için WebRTC HTTPS gerektirir).
- DNS sunucunuz üzerinde `A` (ya da varsa `AAAA`) kaydını sunucunuzun *External (Dış)* IP adresine yönlendirin. Ek olarak TURN sunucunuz olacaksa onun için de bir kayıt lazımdır (örneğin: `turn.sirket.com`).

**Örnek:** `bbb.sirket.com` -> `203.0.113.1`

### 2. Network / Port Gereksinimleri
Kurulum yapılacak sunucunun Internet'e aşağıdaki portlardan erişilebilir (açık) olması şarttır:
- **TCP/80** (HTTP - Let's Encrypt botu ve yönlendirme için)
- **TCP/443** (HTTPS - Nginx Reverse Proxy)
- **TCP/1935** (HTML5 ekran paylaşımı yedeklemesi)
- **TC/UDP 3478** (TURN/STUN istekleri)
- **UDP 16384-32768** (Media / WebRTC, Mediasoup RTCP/SRTP portları)
- *(Opsiyonel)* **TCP 7443** (Güvenli WebSocket yönlendirmesi)

## 2.3 Kurulum (bbb-install.sh Kullanımı)

BBB kurulumu için en güvenli, en kolay ve hata payı en düşük yöntem resmi olarak yayınlanan `bbb-install.sh` bash betiğini kullanmaktır. Bu script arka planda GPG anahtarlarını ekler, PPA/repo tanımlamalarını yapar ve tüm apt paketlerini kurar.

> [!IMPORTANT]
> Script'in Let's Encrypt sertifikasını (SSL/TLS) otomatik alabilmesi için komutu çalıştırmadan önce FQDN'nin IP'nize yönlendiğinden (DNS Propagation) tam emin olmalısınız. (Ping veya `dig bbb.sirket.com` ile kontrol edin).

Hedefimiz: BBB son sürüm kurmak, Let's Encrypt SSL almak ve API entegrasyonu yapmak. (İsteğe bağlı Greenlight UI).

```bash
wget -qO- https://raw.githubusercontent.com/bigbluebutton/bbb-install/refs/heads/v3.0.x-release/bbb-install.sh | bash -s -- \
  -v jammy-300 \  # BBB 3.0 için (Ubuntu 22.04).
  -s bbb.sirket.com \  # Alan adınız.
  -e admin@sirket.com \  # Let's Encrypt uyarı e-postası için.
  -g \ # (Opsiyonel) Greenlight v3 kurmak için -g yazılır.
  -w   # (Opsiyonel) Sunucu üzerindeki ufw (firewall)'u otomatik yapılandırır.
```

### Örnek (Ubuntu 22.04 Üzerine, BBB 3.0 ve Greenlight ile):

```bash
# Sunucuyu güncelleyelim
apt update && apt upgrade -y

# Kurulum scriptini indirelim ve koşturalım
# Not: Kurulum indirme ve çıkarma dahil 15-20 dakika sürebilir.
wget -qO- https://raw.githubusercontent.com/bigbluebutton/bbb-install/refs/heads/v3.0.x-release/bbb-install.sh | bash -s -- -v jammy-300 -s bbb.sirket.com -e admin@sirket.com -w -g
```

## 2.4 Kurulum Sonrası (Post-Installation) Doğrulama

Kurulum tamamlandığında ekranda "Potential Problems" vs. olup olmadığını kontrol edin.

### 1. Servis Durumu Kontrolü
Aşağıdaki komut BBB servislerinin genel sağlığını ve kurulu BBB sürümünü size gösterir:
```bash
bbb-conf --status
```
`active`, `running` olarak Nginx, FreeSWITCH, Redis, mongodb, bbb-html5, bbb-web vb. görmelisiniz.

### 2. Genel Check
```bash
bbb-conf --check
```
Bu komut sistemin IP adresleri, portları (TURN eşleşmeleri, FreeSWITCH şifreleri) uyumsuzluğunu kontrol eder, ekranda uyarılar (Warnings) verirse dikkate alınmalıdır.

### 3. API Key (Secret) Öğrenme
Eğer sistemi Moodle, Canvas, WordPress veya harici bir Greenlight sunucusuna bağlayacaksanız BBB'nin API Endpoint'i ve Secret Key'i lazımdır:
```bash
bbb-conf --secret
```

**Örnek Çıktı:**
```text
URL: https://bbb.sirket.com/bigbluebutton/
Secret: xyz1234567890abcdefghijklmnopqrstuvwxyz
```

### 4. Greenlight Yönetici Hesabı Oluşturma
Greenlight'ı kurmak yetmez, ilk giriş yapabilmeniz için veritabanında (PostgreSQL) bir admin user açmanız gerekir:

**(Greenlight v3 için):**
```bash
cd ~/greenlight-v3
docker exec greenlight-v3 bundle exec rake admin:create
```
*Script size rastgele bir şifre ve admin@... adresi verecektir, bununla `bbb.sirket.com` üzerinden login olup şifrenizi değiştirebilirsiniz.*
