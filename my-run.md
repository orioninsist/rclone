```bash
rclone sync /mnt/samsung/orion-backup-local orion-backup-local: \
  --filter-from /home/murat/rclone-filter.txt \
  --exclude-if-present .ignore-backup \
  --delete-excluded \
  --progress -P
```


Ne işe yarar:

rclone sync: Kaynak klasörü hedefle aynı hale getirir.
/mnt/samsung/orion-backup-local: Kaynak, yani yerel klasör.
orion-backup-local:: Hedef rclone remote’u.
--filter-from /home/murat/rclone-filter.txt: Hangi dosyaların dahil/haric tutulacağını bu filtre dosyasından okur.
--exclude-if-present .ignore-backup: İçinde .ignore-backup dosyası olan klasörleri yedeklemez.
--delete-excluded: Filtreyle hariç tutulan dosyalar hedefte varsa onları da siler.
--progress -P: Aktarım ilerlemesini terminalde gösterir.
Dikkat: sync hedefi kaynakla birebir yapmaya çalışır. Kaynakta olmayan dosyalar hedefte varsa silinebilir. Özellikle --delete-excluded da açık olduğu için önce şu güvenli test komutunu çalıştırmanı öneririm:



```bash
rclone sync /mnt/samsung/orion-backup-local orion-backup-local: \
  --filter-from /home/murat/rclone-filter.txt \
  --exclude-if-present .ignore-backup \
  --delete-excluded \
  --progress -P \
  --dry-run
```

--dry-run gerçek silme/kopyalama yapmaz, sadece ne yapacağını gösterir.