# Rclone Google Drive Komutlari

Bu dosyadaki yeni formatta Google Drive remote adi:

```bash
Drive:
```

`Drive:` rclone config icindeki Google Drive baglantisidir. Google Drive icindeki klasor yolu `:` isaretinden sonra yazilir.

Ornek:

```bash
Drive:media
Drive:media/projects/SelinsCompass
Drive:orion-backup-local
```

## 1. Google Drive Klasorlerini Sorgulama

Google Drive ana dizinindeki klasorleri gormek icin:

```bash
rclone lsd Drive:
```

Dosya ve klasorleri birlikte gormek icin:

```bash
rclone lsf Drive:
```

Bir klasorun icini gormek icin:

```bash
rclone lsd Drive:media
```

Ornek:

```bash
rclone lsd Drive:media/projects
```

## 2. Google Drive'da Klasor Yolunu Bulma

Yeni olusturdugun klasorun yolunu bilmiyorsan ada gore ara:

```bash
rclone lsf Drive: --recursive --dirs-only | grep -i "KLASOR_ADI"
```

Ornek:

```bash
rclone lsf Drive: --recursive --dirs-only | grep -i "SelinsCompass"
```

Cikti boyleyse:

```text
media/projects/SelinsCompass/
```

Rclone hedef yolu boyle olur:

```bash
Drive:media/projects/SelinsCompass
```

Klasor adinda bosluk varsa yolu tirnak icine al:

```bash
rclone lsd "Drive:Colab Notebooks"
```

## 3. Localden Google Drive'a Guvenli Kopyalama

Localden Google Drive'a yedek/arsiv icin `copy` kullan.

`copy`, localden silinen dosyalari Google Drive'dan silmez. Bu yuzden localde yer acmak istiyorsan dogru komut budur.

Genel format:

```bash
rclone copy /LOCAL/KLASOR/YOLU Drive:GOOGLE_DRIVE_HEDEF_KLASOR --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Once her zaman test et:

```bash
rclone copy /LOCAL/KLASOR/YOLU Drive:GOOGLE_DRIVE_HEDEF_KLASOR --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

`--dry-run` hicbir sey yuklemez ve silmez. Sadece rclone'un ne yapacagini gosterir.

## 4. SelinsCompass Archive Kopyalama

Local kaynak:

```bash
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive
```

Google Drive hedef:

```bash
Drive:media/projects/SelinsCompass
```

Once test:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Bu komutun anlami:

```text
Archive klasorunun icindeki uygun dosyalari Google Drive'daki SelinsCompass klasorune kopyala.
Filtre dosyasindaki gereksiz/hassas seyleri alma.
.ignore-backup olan klasorleri komple atla.
Localden silinenleri Google Drive'dan silme.
Dry-run oldugu icin simdilik sadece test et, gercek islem yapma.
```

Komutun parcalari:

```bash
rclone copy
```

Localden Google Drive'a kopyalama yapar.

Backup/arsiv icin en onemli nokta budur:

- Localde yeni dosya varsa Google Drive'a yukler.
- Localde degisen dosya varsa Google Drive'daki kopyayi gunceller.
- Localde silinen dosyayi Google Drive'dan silmez.

Bu yuzden localde yer acmak istiyorsan `copy` guvenlidir.

```bash
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive
```

Local kaynak klasordur.

Bu klasorun kendisi degil, bu klasorun icindeki dosya ve klasorler hedefe kopyalanir.

Yani kaynak:

```text
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive
```

Hedef:

```text
Drive:media/projects/SelinsCompass
```

oldugu icin `archive` klasorunun icindekiler direkt `SelinsCompass` icine gider.

```bash
Drive:media/projects/SelinsCompass
```

Google Drive hedef klasordur.

Burada:

- `Drive:` rclone config icindeki Google Drive remote adidir.
- `media/projects/SelinsCompass` Google Drive icindeki klasor yoludur.

Bu hedef yoksa rclone normalde hedef klasoru olusturabilir.

```bash
--filter-from /home/murat/rclone-filter.txt
```

Filtre kurallarini bu dosyadan okur.

Bu filtre su tarz seyleri backup disinda birakir:

- `node_modules/`
- `.venv/`, `venv/`, `__pycache__/`
- `target/`, `build/`, `dist/`, `bin/`, `obj/`
- cache klasorleri
- log ve gecici dosyalar
- disk imajlari
- sertifika/private key dosyalari

Yani bu komut "her seyi oldugu gibi kopyala" degil, filtreye gore temizlenmis backup yapar.

```bash
--exclude-if-present .ignore-backup
```

Bir klasorun icinde `.ignore-backup` dosyasi varsa, o klasoru ve altindaki her seyi backup disinda birakir.

Ornek:

```text
archive/buyuk-gecici-klasor/.ignore-backup
```

varsa su klasor kopyalanmaz:

```text
archive/buyuk-gecici-klasor/
```

```bash
--local-no-check-updated
```

Local dosyalar okunurken "dosya okuma sirasinda degisti mi" kontrolunu azaltir.

Bu local diskten Google Drive'a backup yaparken performans icin kullanilir. Normal arsiv kopyalama isinde sorun cikarmasi beklenmez.

```bash
--progress
```

Aktarim durumunu ekranda gosterir.

Ne kadar yuklendi, hiz nedir, kalan sure ne olabilir gibi bilgileri gormek icindir.

```bash
-P
```

`--progress` kisa yoludur.

Bu komutta hem `--progress` hem `-P` yazilmis. Ikisi birlikte zarar vermez ama gereksiz tekrardir.

Daha temiz yazim icin sadece `-P` yeterlidir.

```bash
--dry-run
```

Test modudur.

Hicbir dosya yuklemez, silmez veya degistirmez. Sadece rclone'un ne yapacagini gosterir.

Ilk denemede mutlaka kullan.

Not: Dogru flag `--dry-run` seklindedir. `--dry-ru` eksik yazimdir ve kullanilmaz.

Daha temiz dry-run komutu:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated -P --dry-run
```

Dry-run dogru gorunurse gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

Daha temiz gercek kopyalama komutu:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated -P
```

Bu iki gercek kopyalama komutu arasindaki fark:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

ve:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated -P
```

Aralarindaki tek fark sudur:

```bash
--progress -P
```

yerine sadece:

```bash
-P
```

kullaniliyor.

`-P`, `--progress` flag'inin kisa halidir. Yani ikisi ayni isi yapar: aktarim ilerlemesini ekranda gosterir.

Bu yuzden:

```bash
--progress -P
```

yazmak gereksiz tekrardir.

Ikisi de calisir, ama daha temiz ve kisa olan komut sudur:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated -P
```

Sonuc olarak davranis aynidir:

- Ayni kaynak klasoru kullanir.
- Ayni Google Drive hedef klasorune kopyalar.
- Ayni filtre dosyasini kullanir.
- `.ignore-backup` olan klasorleri atlar.
- Localden silinenleri Google Drive'dan silmez.
- Sadece ilerleme gosterme flag'i daha temiz yazilmistir.

Bu komut `archive` klasorunun kendisini hedefe koymaz. `archive` klasorunun icindeki dosya ve klasorleri direkt hedefe kopyalar.

Ornek local dosya:

```text
/mnt/samsung/orion-backup-local/areas/SelinsCompass/archive/foto1.jpg
```

Google Drive'da boyle olur:

```text
Drive:media/projects/SelinsCompass/foto1.jpg
```

Boyle olmaz:

```text
Drive:media/projects/SelinsCompass/archive/foto1.jpg
```

Eger `archive` klasor adi da Google Drive'da olsun istersen hedefe `archive` ekle:

```bash
rclone copy /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass/archive --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

## 5. Kopyalama Sonrasi Kontrol

Kopyalama bittikten sonra local kaynak ile Google Drive hedefini karsilastir:

```bash
rclone check /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive Drive:media/projects/SelinsCompass --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup
```

`check` sorunsuz gecerse localdeki dosyalari silebilirsin.

Bu islemde `copy` kullanildigi icin, localden sildigin dosyalar Google Drive'dan silinmez.

## 6. Google Drive'dan Locale Geri Kopyalama

Google Drive'daki yedegi locale geri almak icin:

```bash
rclone copy Drive:media/projects/SelinsCompass /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive-restore --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --progress -P --dry-run
```

Dry-run dogruysa gercek indirme:

```bash
rclone copy Drive:media/projects/SelinsCompass /mnt/samsung/orion-backup-local/areas/SelinsCompass/archive-restore --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --progress -P
```

## 7. Orion Backup Local Klasorunu Drive'a Kopyalama

Local kaynak:

```bash
/mnt/samsung/orion-backup-local
```

Google Drive hedef:

```bash
Drive:orion-backup-local
```

Once test:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P --dry-run
```

Gercek kopyalama:

```bash
rclone copy /mnt/samsung/orion-backup-local Drive:orion-backup-local --filter-from /home/murat/rclone-filter.txt --exclude-if-present .ignore-backup --local-no-check-updated --progress -P
```

## 8. Git Auto Commit

Yedekleme oncesi otomatik git commit:

```bash
/home/murat/orion-backup-local/dev/git-auto-commit.sh
```

## 9. Sync Uyarisi

Bu arsiv/yedek akisi icin `sync` kullanma.

`sync`, hedefi kaynakla ayni yapar. Localde dosya silersen sonraki `sync` Google Drive'dan da silebilir.

Localden silip Google Drive'da kalsin istiyorsan:

```bash
rclone copy
```

kullan.
