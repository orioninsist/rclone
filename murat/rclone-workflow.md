# Rclone Google Drive Workflow

```bash
rclone copy "/mnt/local/archives/22MAYISORDER" "Drive:order/22MAYISORDER" --filter-from /home/murat/rclone-filter.txt --exclude ".Trash-*/**" --exclude "**/.Trash-*/**" --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

# Kontrol icin
```bash
rclone check "/mnt/local/archives/22MAYISORDER" "Drive:order/22MAYISORDER" \
  --filter-from /home/murat/rclone-filter.txt \
  --exclude ".Trash-*/**" \
  --exclude "**/.Trash-*/**" \
  --exclude-if-present .ignore-backup \
  -P
```
**Bu sana kaynak ve hedef aynı mı diye kontrol eder. Sorun varsa hangi dosyada olduğunu yazar.**
### Daha detaylı log almak istersen:
```bash
rclone check "/mnt/local/archives/22MAYISORDER" "Drive:order/22MAYISORDER" \
  --filter-from /home/murat/rclone-filter.txt \
  --exclude ".Trash-*/**" \
  --exclude "**/.Trash-*/**" \
  --exclude-if-present .ignore-backup \
  --combined /home/murat/rclone-check-22MAYISORDER.txt \
  -P
```
```bash
rclone copy "/mnt/samsung/orion-backup-local" "Drive:orion-backup-local" --filter-from /home/murat/rclone-filter.txt --exclude ".Trash-*/**" --exclude "**/.Trash-*/**" --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Bu komutta `.Trash-1000` gibi Linux cop kutusu klasorleri Google Drive'a yedeklenmez.

- `--exclude ".Trash-*/**"` kaynak klasorun en ust seviyesindeki `.Trash-1000` benzeri klasorleri atlar.
- `--exclude "**/.Trash-*/**"` alt klasorlerin icindeki `.Trash-1000` benzeri klasorleri de atlar.

Bu notta sadece Google Drive kullanimi anlatiliyor.

Mevcut rclone remote adi:

```bash
Drive:
```

Buradaki `Drive:` Google Drive baglantisinin adidir. Google Drive icindeki klasor yolu `:` isaretinden sonra yazilir.

## En Kisa Cevap: Secilen Klasoru Google Drive'a Yedekle

Senin istedigin is icin dogru komut `rclone copy`.

`copy` kullanirsan:

- Dosyalar localden Google Drive'a kopyalanir.
- Localde sonradan sildigin dosya Google Drive'dan silinmez.
- Ayni komutu tekrar calistirirsan sadece yeni veya degisen dosyalar yuklenir.

Genel komut:

```bash
rclone copy "/LOCAL/KLASOR/YOLU" "Drive:GOOGLE_DRIVE_HEDEF_KLASOR" --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Once deneme yapmak istersen sonuna `--dry-run` ekle:

```bash
rclone copy "/LOCAL/KLASOR/YOLU" "Drive:GOOGLE_DRIVE_HEDEF_KLASOR" --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Ornek: `projects/rclone` klasorunu Google Drive'da `orion-backup-local/projects/rclone` icine yedeklemek:

```bash
rclone copy "/mnt/samsung/orion-backup-local/projects/rclone" "Drive:orion-backup-local/projects/rclone" --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Ornek: herhangi bir local klasoru Drive'daki `media` altina kopyalamak:

```bash
rclone copy "/LOCAL/KLASOR/YOLU" "Drive:media/HEDEF_KLASOR_ADI" --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Onemli mantik:

```text
rclone copy "KAYNAK_LOCAL_KLASOR" "Drive:HEDEF_DRIVE_KLASOR"
```

Kaynak klasorun kendisi degil, icindekiler hedef klasorun icine kopyalanir. Yani:

```bash
rclone copy "/mnt/samsung/fotolar" "Drive:media/fotolar"
```

sonuc:

```text
Drive:media/fotolar/dosya1.jpg
Drive:media/fotolar/alt-klasor/dosya2.jpg
```

Hedefte klasor yoksa rclone genelde kendisi olusturur.

Ornek:

```bash
Drive:media
Drive:orion-backup-local
Drive:projects/selin-projesi
```

## 1. Google Drive Ana Dizinini Sorgulama

Google Drive ana dizinindeki klasorleri gormek icin:

```bash
rclone lsd Drive:
```

Bu komut sadece klasorleri listeler.

Dosya ve klasorleri birlikte gormek icin:

```bash
rclone lsf Drive:
```

Daha detayli bilgi icin:

```bash
rclone lsjson Drive:
```

## 2. Alt Klasorleri Sorgulama

Google Drive icindeki `media` klasorunu gormek icin:

```bash
rclone lsd Drive:media
```

`media` icindeki dosya ve klasorleri birlikte gormek icin:

```bash
rclone lsf Drive:media
```

`orion-backup-local` klasorunu gormek icin:

```bash
rclone lsd Drive:orion-backup-local
```

`orion-backup-local` icindeki dosya ve klasorleri birlikte gormek icin:

```bash
rclone lsf Drive:orion-backup-local
```

## 3. Klasor Yolunu Bulma Mantigi

Komut yapisi sudur:

```bash
rclone lsd Drive:GOOGLE_DRIVE_KLASOR_YOLU
```

Ornekler:

```bash
rclone lsd Drive:media
rclone lsd Drive:orion-backup-local
rclone lsd Drive:orion-backup-local/projects
```

Klasor adinda bosluk varsa tirnak kullan:

```bash
rclone lsd "Drive:Colab Notebooks"
```

## 4. Yeni Olusturulan Klasorun Yolunu Bulma

Google Drive'da yeni bir klasor olusturduysan ve rclone yolunu bilmiyorsan once ana dizini sorgula:

```bash
rclone lsd Drive:
```

Eger yeni klasor ana dizindeyse burada gorunur.

Ornek cikti:

```text
           0 2026-05-18 10:00:00        -1 yeni-yedek
```

Bu klasorun rclone yolu:

```bash
Drive:yeni-yedek
```

Eger klasorun hangi klasorun icinde oldugunu bilmiyorsan ada gore ara:

```bash
rclone lsf Drive: --recursive --dirs-only | grep -i "KLASOR_ADI"
```

Ornek:

```bash
rclone lsf Drive: --recursive --dirs-only | grep -i "yedek"
```

Ornek cikti:

```text
archives/yeni-yedek/
```

Bu durumda rclone yolu:

```bash
Drive:archives/yeni-yedek
```

Eger cikti boyleyse:

```text
media/fotograflar/2026/
```

Rclone yolu:

```bash
Drive:media/fotograflar/2026
```

Klasor adinda bosluk varsa arama yine ayni sekilde yapilabilir:

```bash
rclone lsf Drive: --recursive --dirs-only | grep -i "colab"
```

Ornek cikti:

```text
Colab Notebooks/
```

Bu durumda komutta tirnak kullan:

```bash
"Drive:Colab Notebooks"
```

Yani sorgulama:

```bash
rclone lsd "Drive:Colab Notebooks"
```

Kopyalama:

```bash
rclone copy /LOCAL/KLASOR/YOLU "Drive:Colab Notebooks" --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Kisa mantik:

```text
rclone lsf ciktisi: archives/yeni-yedek/
rclone yolu:        Drive:archives/yeni-yedek
```

## 5. Copy ve Sync Farki

Localden sildigin dosyanin Google Drive'dan silinmesini istemiyorsan `sync` kullanma.

Bu senaryo icin dogru komut:

```bash
rclone copy
```

`copy` sunu yapar:

- Localde olan yeni dosyalari Google Drive'a yukler.
- Degisen dosyalari Google Drive'da gunceller.
- Localde silinen dosyalari Google Drive'dan silmez.

`sync` ise hedefi kaynakla ayni yapmaya calisir. Localde dosya silersen, sonraki `sync` Google Drive'dan da silebilir.

## 6. Orion Backup Klasorunu Google Drive'a Kopyalama

Local kaynak:

```bash
/mnt/samsung/orion-backup-local
```

Google Drive hedef:

```bash
Drive:orion-backup-local
```

Once test etmek icin `--dry-run` kullan:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Bu komut hicbir sey yuklemez veya silmez. Sadece ne yapacagini gosterir.

Her sey dogru gorunurse gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

### Sadece Yeni Eklenen Dosyalari Kopyalamak

Eger Google Drive'da zaten var olan dosyalara hic dokunmak istemiyorsan `--ignore-existing` kullan.

Bu komut hedefte ayni yolda dosya varsa onu atlar. Yani tekrar upload etmez, guncellemez, sadece Google Drive'da olmayan yeni dosyalari yukler.

Once test:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --ignore-existing --progress -P --dry-run
```

Gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --ignore-existing --progress -P
```

Bu yontem en uygunu su durumda:

- Disk formatindan sonra local klasoru tekrar olusturdun.
- Google Drive'daki eski yedeklere dokunmak istemiyorsun.
- Sadece sonradan eklenen yeni dosyalar yuklensin istiyorsun.

Onemli: `--ignore-existing` kullanirsan localde degisen eski dosyalar Google Drive'da guncellenmez. Hedefte dosya varsa rclone onu tamamen atlar.

### Yeni Dosyalari Kopyalayip Degisen Dosyalari Da Guncellemek

Eger sadece yeni dosyalari degil, localde degismis eski dosyalari da Google Drive'da guncellemek istiyorsan `--checksum` kullan.

Bu komut dosyalari checksum ile karsilastirir. Dosya ayniysa tekrar upload etmez. Dosya degismisse hedefteki kopyayi gunceller.

Once test:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --checksum --progress -P --dry-run
```

Gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --checksum --progress -P
```

Bu yontem en uygunu su durumda:

- Google Drive'daki yedek guncel kalsin istiyorsun.
- Ayni dosyalar tekrar upload edilmesin istiyorsun.
- Ama localde icerigi degisen dosyalar Drive'da da yenilensin istiyorsun.

Kisa karar:

```text
Sadece Drive'da olmayan yeni dosyalar yuklensin: --ignore-existing
Yeni dosyalar yuklensin, degisen dosyalar da guncellensin: --checksum
```

## 7. Media Klasorunu Google Drive'a Kopyalama

Google Drive hedef:

```bash
Drive:media
```

Local media klasorunun gercek yolunu bilmen gerekiyor.

Ornek local media yolu:

```bash
/mnt/samsung/media
```

Once test:

```bash
rclone copy /mnt/samsung/media Drive:media --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Her sey dogru gorunurse gercek kopyalama:

```bash
rclone copy /mnt/samsung/media Drive:media --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Eger local media yolu farkliysa sadece ilk yolu degistir:

```bash
rclone copy /LOCAL/MEDIA/YOLU Drive:media --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

## 8. Kopyalama Sonrasi Kontrol

Local klasor ile Google Drive hedefini karsilastirmak icin:

```bash
rclone check /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup
```

Media icin ornek:

```bash
rclone check /mnt/samsung/media Drive:media --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup
```

`check` sorunsuz gecerse, localde yer acmak icin dosyalari silebilirsin. `copy` kullandigin icin localden sildigin dosyalar Google Drive'dan silinmez.

## 9. En Guvenli Akis

Her zaman once klasoru sorgula:

```bash
rclone lsd Drive:
```

Hedef klasoru kontrol et:

```bash
rclone lsd Drive:media
```

Once dry-run yap:

```bash
rclone copy /LOCAL/KLASOR/YOLU Drive:GOOGLE_DRIVE_HEDEF_KLASOR --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Sonra gercek kopyalama yap:

```bash
rclone copy /LOCAL/KLASOR/YOLU Drive:GOOGLE_DRIVE_HEDEF_KLASOR --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Gerekirse kontrol et:

```bash
rclone check /LOCAL/KLASOR/YOLU Drive:GOOGLE_DRIVE_HEDEF_KLASOR --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup
```

## 10. SelinsCompass Archive Icindekileri Google Drive'a Kopyalama

Bu ozel islemde local kaynak klasor:

```bash
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive
```

Google Drive hedef klasor:

```bash
Drive:media/projects/SelinsCompass
```

Bu komut `archive` klasorunun kendisini `SelinsCompass/archive` olarak tasimaz.

Su komut:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass
```

`archive` klasorunun icindeki dosya ve klasorleri direkt olarak su hedefin icine kopyalar:

```bash
Drive:media/projects/SelinsCompass
```

Yani localdeki yapi suysa:

```text
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive/foto1.jpg
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive/notlar/a.md
```

Google Drive'da boyle olur:

```text
Drive:media/projects/SelinsCompass/foto1.jpg
Drive:media/projects/SelinsCompass/notlar/a.md
```

Boyle olmaz:

```text
Drive:media/projects/SelinsCompass/archive/foto1.jpg
```

### Once Test Et

Once `--dry-run` ile test et:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Bu komut hicbir sey yuklemez ve silmez. Sadece ne yapacagini gosterir.

### Gercek Kopyalama

Dry-run ciktisi dogru gorunurse gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

### Kopyalama Sonrasi Kontrol

Kopyalama bittikten sonra local kaynak ile Google Drive hedefini karsilastir:

```bash
rclone check /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup
```

`check` sorunsuz gecerse localdeki dosyalari silebilirsin.

Bu islemde `copy` kullanildigi icin, localden sonra dosya silersen Google Drive'daki kopyalar silinmez.

### Eger Archive Klasor Adi Da Google Drive'da Olsun Istersen

Eger hedefte `archive` klasor adi da gorunsun istersen hedefi boyle yaz:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass/archive --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Bu durumda Google Drive'da yapi boyle olur:

```text
Drive:media/projects/SelinsCompass/archive/foto1.jpg
```
