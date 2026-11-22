# Mango Panel

Docker tabanlı, egg (şablon) sistemine sahip açık kaynak sunucu yönetim paneli.
Oyun sunucuları, web siteleri ve Linux VPS'leri tek bir arayüzden yönetirsiniz.

Lisans: AGPL-3.0-or-later (bkz. `LICENSE`).

## Özellikler (1.41.6)

- **Sunucular:** egg'den oluşturma, başlat / yeniden başlat / durdur / zorla kapat, ad, RAM, CPU ve ortam değişkenlerini düzenleme
- **Süre yönetimi:** 3 gün, 1 hafta, 1 ay, 3 ay, 1 yıl, süresiz veya özel süre; kalan gün gösterimi ve süre dolmadan hatırlatma
- **Canlı konsol:** log akışı ve konsola komut gönderme (geçmiş için ok tuşları)
- **İstatistikler:** sunucu başına anlık CPU ve bellek kullanımı
- **Dosya yöneticisi:** klasörlerde gezinme, metin dosyalarını düzenleme, klasör oluşturma, silme
- **Yedekler:** elle yedek alma, geri yükleme, silme; günlük veya haftalık otomatik yedek (saat seçilebilir, eski yedekler saklama sayısını aşınca silinir)
- **VPS (Linux):** Ubuntu 24.04 tabanlı izole ortam, tarayıcıdan sekmeli web terminali (kopyala/yapıştır destekli). Gerekli paketler ilk açılışta otomatik kurulur. Port veya IP gerektirmez.
- **Hata yönetimi:** hata oluştuğunda log ile birlikte açılan popup ve tek tıkla destek talebi
- **Kullanıcılar:** yönetici/kullanıcı rolleri, sunucu bazlı yetkiler, şifre değiştirme, kullanıcı silme
- **API anahtarları:** kullanıcıların kendi betiklerinden panel API'sine erişmesi için (`mgp_` öneki)
- **Destek ve duyurular:** destek talepleri, üstte duyuru şeridi, süre ve çökme bildirimleri
- **Discord (isteğe bağlı):** hesap bağlama önerilir, zorunlu değildir. Bağlıysa bildirimler Discord DM olarak gider.
- **Siteler:** barındırılan sitelere alt alan adı (Cloudflare üzerinden)
- **Arayüz:** açık ve koyu tema
- **Kurulum:** `setup.py` ile tek komutla kurulum, Cloudflare Tunnel desteği, teşhis (`--doctor`) ve kaldırma (`--uninstall`)

## Mimari

```
İstemci ──> nginx (tek giriş noktası)
              ├── /          PHP (php/index.php)   arayüz sayfası, php-fpm
              ├── /static/   web/                  JS/CSS
              └── /api/      Python FastAPI        yalnızca 127.0.0.1
Barındırılan sunucular: Docker konteynerleri (oyun, web, Node.js, PHP, Python, bot ve VPS eggleri)
```

Erişim iki şekilde yapılır:

- **Doğrudan:** nginx sunucu portunda dinler.
- **Cloudflare Tunnel:** nginx yalnızca 127.0.0.1:8080'de dinler, tünel HTTPS trafiğini buraya yönlendirir.
  Bu modda oyun sunucuları engellenir; web, PHP, Node.js, Python, bot ve VPS eggleri çalışır.

## Kurulum (Debian/Ubuntu, root)

```bash
cd ~/mango-panel
sudo python3 setup.py --cloudflare --domain panel.alanadi.com   # Cloudflare ile
sudo python3 setup.py --yes                                       # doğrudan erişim
```

Cloudflare modunda kurulum bir giriş bağlantısı gösterir. Bağlantıyı tarayıcıda açıp onaylamanız gerekir.
Bunun dışındaki tüm adımlar otomatiktir. Yönetici şifresi belirtilmezse güçlü bir şifre üretilir ve kurulum sonunda gösterilir.

Teşhis ve onarım:

```bash
sudo python3 setup.py --doctor
```

Kaldırma:

```bash
sudo python3 setup.py --uninstall
```

## Güncelleme

Mevcut kurulumun üzerine yeni sürümün kod dosyalarını kopyalayın. `mango.json`, `data/`, `.venv/` ve yönetici bilgi dosyasına dokunmayın.

```bash
# 1) Yeni zip'i sunucuya yükleyin ve açın (ör. /root/mango-update)
cd /root && unzip -o mango-panel-X.Y.Z.zip -d mango-update

# 2) Kod dosyalarını mevcut kuruluma kopyalayın
cp -r /root/mango-update/mango-panel/mango /root/mango-panel/
cp -r /root/mango-update/mango-panel/web /root/mango-panel/
cp -r /root/mango-update/mango-panel/php /root/mango-panel/
cp -r /root/mango-update/mango-panel/eggs /root/mango-panel/
cp /root/mango-update/mango-panel/setup.py /root/mango-panel/

# 3) Arayüz dosyalarını nginx'in okuduğu klasöre yayınlayın
cp /root/mango-panel/php/*.php /var/lib/mango/www/
cp -r /root/mango-panel/web/. /var/lib/mango/www/static/
chmod -R a+rX /var/lib/mango/www

# 4) Paneli yeniden başlatın
systemctl restart mango-panel
```

Zip'in iç klasör adı farklıysa 2. adımdaki yolları buna göre değiştirin.
Veritabanındaki yeni alanlar (ör. yedek ayarları, süre bilgisi) panel açılırken otomatik eklenir.

Panel günlüğü:

```bash
journalctl -u mango-panel -n 50 --no-pager
```

VPS'lerde bir sorun olursa, sunucuda şu komutla tüm VPS'leri tarayıp sorunluları onarabilirsiniz (panel bunu 10 dakikada bir kendiliğinden de yapar):

```bash
cd /root/mango-panel && .venv/bin/python -m mango.vpsdebug
```

## Dosya yapısı

| Yol | Görev |
|---|---|
| `mango/` | Python API, veritabanı, Docker yönetimi, yedekler, dosya yöneticisi, egg yükleyici, VPS onarımı |
| `php/` | Arayüz sayfası (index.php), sağlık kontrolü (health.php) |
| `web/` | Arayüz dosyaları (index.html, app.js, style.css, platform.js) |
| `eggs/` | Uygulama şablonları (YAML). `eggs/store/` altında ek şablonlar bulunur. |
| `setup.py` | Kurulum, teşhis, onarım ve kaldırma aracı |
| `tests/` | Birim ve API testleri (`pytest`) |

Kurulum sırasında `php/` ve `web/` içeriği `/var/lib/mango/www` altına kopyalanır.
Bu klasör www-data tarafından okunabilir olmalıdır. `/root` altındaki dosyalar buradan okunamaz.

## Egg yazmak

| Alan | Açıklama |
|---|---|
| `name`, `description` | Arayüzde görünen ad |
| `category` | `web`, `php`, `nodejs`, `python`, `bot`, `vps` veya `game`. Tünel modunda `game` engellenir. |
| `docker_image` | Kullanılacak Docker imajı (ör. `ubuntu:24.04`) |
| `install_script` | Sunucu ilk oluşturulduğunda (ve "Yeniden kur" ile) çalışır |
| `startup` | Konteynerin ana komutu; `{{DEGISKEN}}` yer tutucuları kullanılabilir |
| `ports` | İlk port konteyner içindeki birincil porttur |
| `variables` | Kullanıcının dolduracağı değerler |
| `publish_ports` | `false` ise portlar dışarı açılmaz (VPS eggleri bunu kullanır) |

Değişken değerlerinde yalnızca `A-Z a-z 0-9 . _ - / : = @ ,` karakterlerine izin verilir (shell enjeksiyonunu önlemek için).

## Güvenlik

- Docker socket erişimi root yetkisine eşdeğerdir. Docker grubuna yalnızca güvendiğiniz kullanıcıları ekleyin.
- Konteynerler sınırlı yetkilerle (`cap_add` listesiyle), `no-new-privileges` ile çalışır ve süreç sayısı sınırlıdır.
- Dosya yöneticisi yalnızca sunucunun kendi klasörüne erişir. Sembolik bağlantılarla klasör dışına çıkma engellenir.
- VPS terminali konteyner dışına çıkamaz; her oturum sunucunun kendi klasöründe açılır.
- Konsol komutları tek satır ve en fazla 500 karakterdir.
- Python API yalnızca 127.0.0.1'de dinler; dışarıya nginx üzerinden açılır.
- `mango.json` (gizli anahtar, veritabanı bilgisi) yalnızca root tarafından okunabilir. Web klasörüne kopyalanmaz.
- Parolalar PBKDF2-SHA256 (260.000 iterasyon) ile saklanır, oturum JWT ile yönetilir.
- Arayüz inline script kullanmaz (CSP `script-src 'self'`).
- Mevcut bir DNS kaydı, hostname'e ait değilse kurulum tarafından değiştirilmez (`--force-dns` gerekir).

## Testler

```bash
pip install -r requirements.txt
python -m pytest -q tests
```

Testler Docker gerektirmez. Konteyner işlemleri (başlat, konsol, terminal, istatistik) gerçek bir Docker ortamında ayrıca doğrulanmalıdır.

## Bilinen sınırlamalar

- Barındırılan sitelerin Cloudflare Tunnel üzerinden otomatik yayını sınırlı; ayrıntılar için kurulum belgelerine bakın.
- Dosya yükleme yok; metin düzenleyici 1 MB sınırlıdır.
- Yedekler yerel diskte tutulur; uzak yedekleme yok.
- Kurulum yalnızca Debian/Ubuntu (apt) ve systemd için tasarlandı.
- VPS'lerde süreç ve thread sayısı, sunucunun toplam sınırına bağlıdır. Çok sayıda çalışan sunucuda sınıra yaklaşılabilir.

## Katkı ve lisans

Projeye katkı memnuniyetle karşılanır. Değişiklikleri gönderirken testlerin geçtiğinden emin olun.

Bu proje AGPL-3.0-or-later lisansı altındadır. Tam metin için `LICENSE` dosyasına bakın veya `https://www.gnu.org/licenses/agpl-3.0.txt` adresini ziyaret edin.
## Powered By Kali
Discord: kalihastasi.dev2
Destek Sunucusu: https://discord.gg/Kq6tgYnxrR
