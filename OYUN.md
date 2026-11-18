# MappyBlast

Queens tarzı mantık bulmacası: her satır, sütun ve renkli bölgede 1 polis fare var, fareler birbirine (çapraz dahil) değmez.

## Oynanış
- Tek dokunuş: pati koyar ya da kaldırır (kapalı ↔ pati, "burada yok" işareti). Hemen olur, kontrol edilmez, bedava; 8 pop döngüsü sesi.
- Çift dokunuş (aynı kareye 300 ms içinde 2. dokunuş): açma denemesi. 1. dokunuşun değişikliği geri alınır, sonra fare varsa çıkış + idle animasyonu ve fare sesi; yoksa −1 can, titreme, kısa siyah-beyaz, titreşim ve kare pati olarak kalır. Açık fareye dokunmak bir şey yapmaz. `dblclick` yok sayılır (yakınlaştırma olmaz).
- Sol üstteki ilk tuş (`ik-yeniden`, "Bölümü baştan başla"): aynı bulmacayı baştan başlatır.
- 3 can (kalp paneli). Can bitince ödüllü reklam = canlar dolar (`Ads.rewarded('can')`).
- Ampul (5 hak): fare olamayacak en fazla 5 kareyi patiler. Büyüteç (3 hak): bir fareyi açar. Hak bitince reklam +2.

## Ekran yerleşimi (yukarıdan aşağı, 360×740'ta kaydırmasız)
- Ölçek: tüm boyutlar `--u` biriminin katı (`calc(N*var(--u))`). `fitLayout()` (yüklenince, resize, orientationchange, tam ekran, yazı tipi yüklenince, her tablo kuruluşunda): u = max(min(genişlik/390, yükseklik/844), min(1, genişlik/360)) px; CSS'te `min(100vw/390, 100dvh/844)` yedeği. 360–390 px telefonda u = 1, 1072×1918'de ≈ 2.27; yatayda yükseklik belirler. Sahne genişliği sınırsız; kartlar (`--cw` = min(88vw, 360u)) da u ile büyür.
- `--bw`: tablo çerçevesinin genişliği (en az üst çubuğun sığdığı 336u). Üst çubuk, gösterge sırası, kural şeridi, benzetici banner bu genişlikte ve çerçeveyle hizalı; tam ekran tuşu çerçevenin sağ kenarına hizalı.
- Üst çubuk: sol üstte `ui-ayar` + sağında `ui-pause`, ayarların altında `ik-yeniden` (44u). Ortada bölüm plakası + yıldızlar, sağ üstte para paneli.
- Gösterge sırası: can paneli, süre, bulunan/N sayacı. Öğreticide gizli; öğretici yazı kutusu (ve bölüm 1 kural balonu) bu sıranın yerinde durur.
- Kural şeridi: beyaz yuvarlak şerit, 3 küçük kart (12 px kare/pati/fare ile 3×3 mini resim + yazı): "Her renkte 1 fare", "Her satır ve sütunda 1 fare", "Fareler birbirine değmez". Öğreticide de görünür.
- Bölge sekmesi: çerçevenin üst kenarına yapışık, çerçeve renginde sekme (`#regTab`, `role="img"`, "Bulunan fareler: x / N"). Her bölge için sırayla mini kare (düz `kare-N` + `fare-cikis` 0. kare = boş delik), boyu tablo karesinin yarısı. O bölgenin faresi bulununca aynı sprite oynatıcıyla (`startPet`) çıkış → idle. Her bölüm başında / baştan başlatınca yeniden kurulur.
- Tablo: olabildiğince büyük, kalan yüksekliğe sığar (`.play` boy kabı; genişlik = min(sahne içi, (yükseklik − 92u) / (1.018 + 0.454/N))). Üst blok yukarıda, tablo + yardım çubuğu kalan alanda dikey ortalı.
- Yardım çubuğu: ampul + büyüteç (72u, arası ~2 tuş = 144u; dar çerçevede tam ekran tuşuna yer kalsın diye `--bw − 248u`'ya, en az 20u'ya iner), tablonun 14u altında, ortalı.
- En altta 60u reklam bandı (`#adBand`). Tam ekran tuşu (44u) sağ altta bandın hemen üstünde, çerçevenin sağ kenarına hizalı.
- Banner: `Ads.banner()` bölüm başında; yalnızca bölüm > 5, `noAds` değil, öğretici değil. `Ads.provider.showBanner()` / `hideBanner()`; benzetici bantta 50u yüksek, çerçeve genişliğinde ortalı açık gri "Reklam" şeridi çizer. Bölüm 6'dan önce bant boş durur.

## Bölüm sistemi
- Bölümler oyun içinde üretiliyor (tohumlu RNG), hep tek çözümlü ve tahminsiz çözülebilir.
- Tablo: 1–3 → 4×4, 4–7 → 5×5, 8–13 → 6×6, 14–22 → 7×7, sonrası 8×8 (8 renk kare olduğu için şimdilik tavan).
- Zeka puanı (ZP) süre/hata/yardım kullanımına göre güncellenir; 10. bölümden sonra testere dişi zorluk, 5'te bir zor bölüm + nefes bölümü.
- Bölüm 0: rehberli 4×4 eğitim, "Geç" tuşu. Gösterilen kareler çift dokunuşla açılır; ilk fareden sonra otomatik patiler gösterilir ve "Tek dokunuş pati koyar ya da kaldırır." yazısı çıkar.
- Kayıt: localStorage `fareBulmaca.v1`.

## Ayarlar & sesler
- Sol üstte `ui-ayar` tuşu (bkz. Ekran yerleşimi).
- "Ayarlar" kartı: dikey satır listesi (200 px sütun, simge 56 px solda, yazı 20 px sağda). Müzik / Ses / Titreşim, her biri `aria-pressed` satır tuşu (açık/kapalı simge + yazı + "Açık"/"Kapalı"), son satır `ik-kapat` "Kapat". Kayıtta `music`, `sfx`, `vibe` (varsayılan hepsi açık). Kart açıkken süre durur.
- `playSound` `sfx` kapalıysa çalmaz; `vibrate()` `vibe` kapalıysa titremez.
- Müzik: `Music` nesnesi, tek `Audio('muzik.mp3')`, döngü, ses .35. İlk dokunuşta başlar (tarayıcı kuralı), sayfa gizlenince durur, görünür olunca devam eder, reklam sırasında susar. Açma/kapama anında.
- Her tuşa basışta `pop-1.mp3` (.5); karelerde 8 pop döngüsü; fare açılınca `ses-fare.mp3`.
- Duraklat kartı (aynı satır düzeni): `ik-oynat` Devam et, `ik-ev` Ana sayfa (şimdilik bölümü baştan başlatır, `// TODO: ana menü`). Kazandın kartında `ik-kupa`. Can bitti kartında `ik-reklam` + `ik-yeniden`.

## Reklam katmanı (`Ads`)
- Gerçek SDK yok; `Ads.provider` = `SimAdProvider` (2 sn "Reklam" kartı + çubuk). SDK gelince yalnızca `provider` değişir: `showRewarded(cb, failCb)`, `showInterstitial(cb)`, `showBanner()`, `hideBanner()` (`// TODO: AdMob (Android) / AdSense H5 (web)`).
- `Ads.rewarded(placement, onReward)`: yerler `can`, `ampul`, `buyutec`. Teklif tuşu ekrana gelince `rewardedOffered`, ödül verilince `rewardedWatched` artar. Reklam gelmezse "Reklam şu an yok" kartı.
- `Ads.interstitialAfterLevel(level, then)`: "Sonraki bölüm"e basınca. Öğreticide, `noAds` oyuncuda ve bölüm 4'ten önce yok. O bölümde can bittiyse sonraki 1 (iki kez bittiyse 2) geçiş reklamsız (`adCalm`). Son reklamdan beri en az 3 dk ve en az N bölüm: C 3, B 4, A 5. Sayaç `interstitialShown`.
- Profil (`profile`): D = `noAds`; 5 tekliften önce B; izleme oranı ≥%60 A, %20–60 B, <%20 C.
- A oyuncusu: bir yardım hakkı 0'a inince bir kez "+2 hak" kartı kendiliğinden açılır (`adProactiveDone`).

## Dosyalar
- `kare-N` / `pati-N`: pastel bölge kareleri, 9 renk (0 kırmızı, 1 sarı, 2 mavi, 3 mor, 4 pembe, 5 turkuaz, 6 turuncu, 7 yeşil, 8 kahve). `fare-cikis` / `fare-idle`: fare sprite şeritleri (12 kare, 170 ms).
- `ui-*`, `rakam-*`, `kalp-dolu` / `kalp-bos` (pastel): arayüz görselleri.
- `ik-*` yuvarlak pastel tuşlar: `ik-ampul` (ipucu), `ik-buyutec` (fare göster), `ik-reklam` (reklam izle), `ik-muzik(-kapali)`, `ik-ses(-kapali)`, `ik-titresim(-kapali)`, `ik-ev`, `ik-yeniden`, `ik-oynat`, `ik-kapat`, `ik-kupa`.
- Yardım tuşlarındaki sayı rozeti: 28×28 `ui-rozet` dairesi (sağ üst), içinde 18 px rakam; hak bitince 28×28 `ui-arti`.
- Henüz kullanılmayan (bölüm haritası / günlük hediye için): `ik-kilit`, `ik-tik`, `ik-bolum-kare`, `ik-hediye`.
- Sesler: `muzik.mp3` (arka plan döngüsü), `pop-1..8.mp3` (pati sesleri, sırayla döner), `ses-fare.mp3` (fare açılma).
- Tema: pastel (2026-11-18). Çerçeve: farenin halkası renginde açık lila-gri kenar, iç zemin farenin deliğindeki koyu mor-kahve (#251d27).
