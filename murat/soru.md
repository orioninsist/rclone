# Drive:order Duplicate Temizligi - Bundan Sonra Yapilacaklar

## Mevcut Durum

Google Drive tarafinda kontrol edilen ana dizin:

```text
Drive:order/
```

Envanter dosyasi olusturuldu, bozuk progress satirlari temizlendi ve JSON dogrulandi.

Gecerli envanter dosyasi:

```text
/home/murat/rclone-order-cleanup/order-drive-lsjson.json
```

Bozuk orijinal dosya yedegi:

```text
/home/murat/rclone-order-cleanup/order-drive-lsjson.json.broken-progress-backup
```

JSON dogrulama sonucu:

```text
141747 dosya
```

## Cikan Sonuclar

`Size + MD5 + SHA256` kriterine gore:

```text
Toplam dosya                         : 141747
Size + MD5 + SHA256 olan dosya       : 141446
MD5 olmayan dosya                    : 73
SHA256 olmayan dosya                 : 301
Duplicate grup sayisi                : 31961
Fazla duplicate kopya sayisi         : 68554
Potansiyel bosalacak alan            : 278.678 GiB
```

Olusturulan rapor dosyalari:

```text
/home/murat/rclone-order-cleanup/order-metadata-duplicate-report.txt
/home/murat/rclone-order-cleanup/order-metadata-duplicate-delete-list.txt
/home/murat/rclone-order-cleanup/order-no-md5-or-no-sha256-report.txt
```

## Cok Onemli Uyari

`lsjson` icin `-P` kullanma.

Testte goruldu ki `-P` progress satirlari JSON dosyasinin icine karisabiliyor ve JSON bozuluyor.

Dogru envanter komutu:

```bash
mkdir -p /home/murat/rclone-order-cleanup

echo "[$(date '+%F %T')] Drive envanteri basliyor"

rclone lsjson "Drive:order/" \
  --recursive \
  --hash \
  --files-only \
  --exclude ".Trash-*/**" \
  --exclude "**/.Trash-*/**" \
  --log-level INFO \
  --log-file /home/murat/rclone-order-cleanup/order-drive-lsjson.log \
  > /home/murat/rclone-order-cleanup/order-drive-lsjson.json

echo "[$(date '+%F %T')] Drive envanteri bitti"
jq length /home/murat/rclone-order-cleanup/order-drive-lsjson.json
```

## Duplicate Silme Kuralimiz

Bir dosya sadece su sartlarda duplicate kabul edilir:

```text
Size ayni
MD5 ayni
SHA256 ayni
```

Bu uc deger ayniysa dosyalar pratikte ayni icerik kabul edilir.

Silme yapilmayan durumlar:

```text
MD5 yoksa silme yok
SHA256 yoksa silme yok
Size farkliysa silme yok
MD5 farkliysa silme yok
SHA256 farkliysa silme yok
```

Klasorler silinmeyecek.

Dosya adi, klasor adi, tarih, `MimeType`, Google Drive `ID` tek basina duplicate kaniti degildir.

## Ayni Path Duplicate Uyarisi

Google Drive'da ayni path ile gorunen duplicate nesneler de var.

Tespit:

```text
Ayni path duplicate path sayisi      : 177
Ekstra ayni-path obje sayisi         : 199
```

Bu dosyalar `--files-from` ile temizlenmemeli.

Sebep:

```text
Path ayni oldugu icin rclone delete --files-from hangi objeyi silecegini guvenli sekilde ayirt edemeyebilir.
```

Bu grup icin daha sonra ayri olarak `rclone dedupe` veya ID bazli ozel kontrol gerekir.

Bu asamada ayni-path duplicate konusuna dokunma.

## Simdi Yapilacak Guvenli Islem

Simdi sadece farkli pathlerde duran ve `Size + MD5 + SHA256` ile ayni oldugu tespit edilen duplicate dosyalar icin ilerle.

Hazir metadata silme listesi:

```text
/home/murat/rclone-order-cleanup/order-metadata-duplicate-delete-list.txt
```

Bu liste su an yaklasik olarak:

```text
68506 satir
```

Bu liste dogrudan gercek silme icin kullanilmadan once dry-run ile kontrol edilecek.

## 1. Raporu Incele

Once raporun ilk kismini oku:

```bash
sed -n '1,160p' /home/murat/rclone-order-cleanup/order-metadata-duplicate-report.txt
```

En buyuk duplicate gruplari gormek icin:

```bash
grep -nE '^(GROUP|COUNT|SIZE|KEEP|DELETE_METADATA_MATCH)' \
  /home/murat/rclone-order-cleanup/order-metadata-duplicate-report.txt \
  | sed -n '1,220p'
```

Silme listesinden ilk 100 dosyayi gormek icin:

```bash
sed -n '1,100p' /home/murat/rclone-order-cleanup/order-metadata-duplicate-delete-list.txt
```

Silme listesi kac satir:

```bash
wc -l /home/murat/rclone-order-cleanup/order-metadata-duplicate-delete-list.txt
```

## 2. Final Silme Listesini Hazirla

Simdilik filter temizligini karistirma.

Final listeyi sadece duplicate listesinden olustur:

```bash
cp /home/murat/rclone-order-cleanup/order-metadata-duplicate-delete-list.txt \
  /home/murat/rclone-order-cleanup/order-final-delete-list.txt
```

Final listeyi kontrol et:

```bash
wc -l /home/murat/rclone-order-cleanup/order-final-delete-list.txt
sed -n '1,100p' /home/murat/rclone-order-cleanup/order-final-delete-list.txt
```

## 3. Dry-Run Yap

Bu komut gercek silme yapmaz.

```bash
echo "[$(date '+%F %T')] Dry-run basliyor"

rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-final-delete-list.txt \
  --dry-run \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-final-delete-dry-run.log \
  --log-level INFO

echo "[$(date '+%F %T')] Dry-run bitti"
```

Dry-run logunu kontrol et:

```bash
tail -n 80 /home/murat/rclone-order-cleanup/order-final-delete-dry-run.log
```

Dry-run kac dosya silecek gibi gorunuyor kontrol et:

```bash
grep -Ei 'delete|would remove|would delete|deleted' \
  /home/murat/rclone-order-cleanup/order-final-delete-dry-run.log \
  | wc -l
```

## 4. Dry-Run Sonucu Dogruysa Gercek Silme

Dry-run mantikli gorunuyorsa gercek silme:

```bash
echo "[$(date '+%F %T')] Gercek silme basliyor"

rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-final-delete-list.txt \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-final-delete.log \
  --log-level INFO

echo "[$(date '+%F %T')] Gercek silme bitti"
```

Gercek silme logunu kontrol et:

```bash
tail -n 120 /home/murat/rclone-order-cleanup/order-final-delete.log
```

## 5. Silme Sonrasi Yeni Envanter Al

```bash
echo "[$(date '+%F %T')] Temizlik sonrasi envanter basliyor"

rclone lsjson "Drive:order/" \
  --recursive \
  --hash \
  --files-only \
  --exclude ".Trash-*/**" \
  --exclude "**/.Trash-*/**" \
  --log-level INFO \
  --log-file /home/murat/rclone-order-cleanup/order-drive-lsjson-after-clean.log \
  > /home/murat/rclone-order-cleanup/order-drive-lsjson-after-clean.json

echo "[$(date '+%F %T')] Temizlik sonrasi envanter bitti"
jq length /home/murat/rclone-order-cleanup/order-drive-lsjson-after-clean.json
```

## 6. Son Duplicate Kontrolu

Yeni envanteri ana dosya olarak kullan:

```bash
cp /home/murat/rclone-order-cleanup/order-drive-lsjson-after-clean.json \
  /home/murat/rclone-order-cleanup/order-drive-lsjson.json
```

Sonra duplicate sayimini tekrar yap:

```bash
python3 - <<'PY'
import json
from collections import defaultdict
from pathlib import Path

p = Path('/home/murat/rclone-order-cleanup/order-drive-lsjson.json')
data = json.loads(p.read_text())

groups = defaultdict(list)
no_md5 = 0
no_sha256 = 0

for item in data:
    if item.get('IsDir'):
        continue
    hashes = item.get('Hashes') or {}
    md5 = hashes.get('md5') or hashes.get('MD5')
    sha256 = hashes.get('sha256') or hashes.get('SHA256') or hashes.get('SHA-256')
    size = item.get('Size')
    if not md5:
        no_md5 += 1
    if not sha256:
        no_sha256 += 1
    if md5 and sha256 and isinstance(size, int) and size >= 0:
        groups[(size, md5, sha256)].append(item)

dups = [g for g in groups.values() if len(g) > 1]
extra = sum(len(g) - 1 for g in dups)
bytes_extra = sum((len(g) - 1) * (g[0].get('Size') or 0) for g in dups)

print('Duplicate grup:', len(dups))
print('Fazla kopya:', extra)
print('Potansiyel GiB:', round(bytes_extra / 1024**3, 3))
print('MD5 olmayan:', no_md5)
print('SHA256 olmayan:', no_sha256)
PY
```

Beklenen sonuc:

```text
Duplicate grup sayisi ciddi sekilde azalacak.
Fazla kopya sayisi ciddi sekilde azalacak.
Ayni-path duplicate olanlar kalabilir.
MD5/SHA256 olmayan dosyalar silinmeyecek.
```

## Filter Temizligi Sonraya Kalsin

Bu tur gereksiz dosyalar da bulundu:

```text
.git altinda        : 5001 dosya, 17.757 GiB
.cache altinda      : 2004 dosya, 12.158 GiB
__pycache__ / .pyc  : 784 dosya
.log                : 45 dosya
.tmp                : 2 dosya
```

Ama bunlari duplicate temizligi ile karistirma.

Once duplicate temizligi bitsin.

Sonra filter temizligi icin ayri liste ve ayri dry-run yapilacak.

## Filter Temizligi - Ikinci Faz

Duplicate temizligi bittikten sonra final envanter uzerinden gereksiz dosya raporu olusturuldu.

Kullanilan envanter:

```text
/home/murat/rclone-order-cleanup/order-drive-lsjson-after-final-clean.json
```

Olusturulan rapor ve listeler:

```text
/home/murat/rclone-order-cleanup/order-filter-candidates-report.txt
/home/murat/rclone-order-cleanup/order-filter-delete-list.txt
/home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt
/home/murat/rclone-order-cleanup/order-filter-review-disk-images.txt
```

Genel aday sonucu:

```text
Toplam filter adayi dosya : 12358
Toplam aday alan          : 16.193 GiB
```

Yuksek guvenli silme listesi:

```text
Dosya sayisi : 12352
Alan         : 12.648 GiB
Liste        : /home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt
```

Ayrica dikkatli incelenecek disk imajlari:

```text
Dosya sayisi : 6
Alan         : 3.545 GiB
Liste        : /home/murat/rclone-order-cleanup/order-filter-review-disk-images.txt
```

Disk imajlari `.iso`, `.img`, `.qcow2`, `.vdi`, `.vmdk` gibi dosyalardir. Bunlar bazen gercekten gerekli olabilir. Bu yuzden otomatik silme listesine karistirilmadi.

### 1. Filter Raporunu Incele

```bash
sed -n '1,120p' /home/murat/rclone-order-cleanup/order-filter-candidates-report.txt
```

Yuksek guvenli silme listesinin ilk 100 satiri:

```bash
sed -n '1,100p' /home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt
```

Liste kac dosya:

```bash
wc -l /home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt
```

Disk imajlarini ayri incele:

```bash
cat /home/murat/rclone-order-cleanup/order-filter-review-disk-images.txt
```

### 2. Yuksek Guvenli Filter Dry-Run

Bu komut gercek silme yapmaz.

```bash
rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt \
  --dry-run \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-filter-delete-high-confidence-dry-run.log \
  --log-level INFO
```

Dry-run sonucu beklenen:

```text
Yaklasik 12352 dosya
Yaklasik 12.648 GiB
```

Dry-run logunu kontrol et:

```bash
tail -n 100 /home/murat/rclone-order-cleanup/order-filter-delete-high-confidence-dry-run.log
```

### 3. Dry-Run Dogruysa Gercek Filter Silme

```bash
rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-filter-delete-list-high-confidence.txt \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-filter-delete-high-confidence.log \
  --log-level INFO
```

Silme sonrasi log kontrolu:

```bash
tail -n 120 /home/murat/rclone-order-cleanup/order-filter-delete-high-confidence.log
```

### 4. Disk Imajlari Icin Ayri Karar

Disk imaji listesi:

```bash
cat /home/murat/rclone-order-cleanup/order-filter-review-disk-images.txt
```

Bu dosyalar otomatik silinmedi.

Eger bunlarin gereksiz olduguna karar verilirse path-only silme listesi olustur:

```bash
cut -f2- /home/murat/rclone-order-cleanup/order-filter-review-disk-images.txt \
  > /home/murat/rclone-order-cleanup/order-filter-delete-list-disk-images.txt
```

Once dry-run:

```bash
rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-filter-delete-list-disk-images.txt \
  --dry-run \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-filter-delete-disk-images-dry-run.log \
  --log-level INFO
```

Dogruysa gercek silme:

```bash
rclone delete "Drive:order/" \
  --files-from /home/murat/rclone-order-cleanup/order-filter-delete-list-disk-images.txt \
  -P \
  --log-file /home/murat/rclone-order-cleanup/order-filter-delete-disk-images.log \
  --log-level INFO
```

### 5. Filter Temizligi Sonrasi Son Envanter

```bash
rclone lsjson "Drive:order/" \
  --recursive \
  --hash \
  --files-only \
  --exclude ".Trash-*/**" \
  --exclude "**/.Trash-*/**" \
  --log-level INFO \
  --log-file /home/murat/rclone-order-cleanup/order-drive-lsjson-after-filter-clean.log \
  > /home/murat/rclone-order-cleanup/order-drive-lsjson-after-filter-clean.json

jq length /home/murat/rclone-order-cleanup/order-drive-lsjson-after-filter-clean.json
```

## En Guvenli Sira

```text
1. Raporu oku
2. Final duplicate silme listesini hazirla
3. Dry-run yap
4. Dry-run logunu kontrol et
5. Dogruysa gercek silme yap
6. Yeni envanter al
7. Duplicate sayimini tekrar yap
8. Ayni-path duplicate ve filter temizligini sonraya birak
```

## Su Anda Gercek Silme Yapilmadi

Simdiye kadar yapilanlar:

```text
Envanter alindi.
Bozuk JSON temizlendi.
JSON dogrulandi.
Duplicate raporlari uretildi.
Silme listesi hazirlandi.
```

Henuz gercek silme yapilmadi.
