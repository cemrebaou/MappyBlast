# MappyBlast

Queens tarzı mantık bulmacası: her satır, sütun ve renkli bölgede 1 polis fare var, fareler birbirine (çapraz dahil) değmez.

## Oynanış
- Kareye 1 dokunuş: patili işaret (burada fare yok). Kontrol edilmez.
- Patili kareye tekrar dokunuş: açar. Fare varsa çıkış + idle animasyonu ve fare sesi; yoksa −1 can, titreme, kısa siyah-beyaz.
- 3 can (kalp paneli). Can bitince reklam = canlar dolar (şimdilik reklam SDK yok, `// TODO: gerçek ödüllü reklam`).
- Ampul (5 hak): fare olamayacak en fazla 5 kareyi patiler. Büyüteç (3 hak): bir fareyi açar. Hak bitince reklam +2.

## Bölüm sistemi
- Bölümler oyun içinde üretiliyor (tohumlu RNG), hep tek çözümlü ve tahminsiz çözülebilir.
- Tablo: 1–3 → 4×4, 4–7 → 5×5, 8–13 → 6×6, 14–22 → 7×7, sonrası 8×8 (8 renk kare olduğu için şimdilik tavan).
- Zeka puanı (ZP) süre/hata/yardım kullanımına göre güncellenir; 10. bölümden sonra testere dişi zorluk, 5'te bir zor bölüm + nefes bölümü.
- Bölüm 0: rehberli 4×4 eğitim, "Geç" tuşu.
- Kayıt: localStorage `fareBulmaca.v1`.

## Dosyalar
- `kare-N` / `pati-N`: bölge kareleri (8 kullanılmaz). `fare-cikis` / `fare-idle`: fare sprite şeritleri (12 kare, 170 ms).
- `ui-*`, `rakam-*`, `kalp-*`, `btn-*`: arayüz görselleri. `ses-pati.mp3`, `ses-fare.mp3`: sesler.
