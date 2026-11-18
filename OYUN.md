# MappyBlast

## Yapı (2026-11-18)
- `index.html` = açılış (launcher): yükleme ekranı + ana menü. `<title>Mappy</title>`.
- `puzzle.html` = Mappy Puzzle (eski `index.html`; varlıkları kök klasörde, yollar aynı). "Ana sayfa" (`ik-ev`) → `index.html`. Kayıt `fareBulmaca.v1`.
- `jump/index.html` = Mappy Jump (kendi varlıkları `jump/` içinde; ortak arayüz resimlerini `../` ile kullanır). Kayıt `mappyJump.v1`. Puzzle ile ortak JS dosyası yok, kod ayrı.
- Launcher ayarı: `mappyMenu.v1` (`music`).
- Dil ayarı: `mappyLang` (üç sayfanın ortak localStorage anahtarı). Bkz. "Dil sistemi".

## Dil sistemi (2026-11-18)
- 11 dil, bu sırayla (dil tuşu da bu sırayla döner): `tr` Türkçe, `en` English, `zh` 中文 (sadeleştirilmiş), `hi` हिन्दी, `es` Español, `fr` Français, `ar` العربية, `bn` বাংলা, `pt` Português, `ru` Русский, `ur` اردو.
- Varsayılan: kayıt yoksa tarayıcı dili (`navigator.languages`, ilk eşleşen) listede ise o, değilse İngilizce. Seçim yalnızca launcher'daki dil tuşuyla kaydedilir.
- Her sayfanın `<script>` başında aynı üretilmiş blok var: `/* @@I18N-BEGIN */ … /* @@I18N-END */`. İçinde `LANGS`, o sayfanın kullandığı anahtarlarla `I18N` sözlüğü (anahtar → 11 dillik dizi, sıra yukarıdaki gibi) ve ortak kod: `readLang()`, `setLang()`, `saveLang()`, `t(key, a, b…)` (`{0}`, `{1}` yer tutucuları; eksik çeviri → İngilizce), `textDir(el)`, `fitText(el)` / `fitAll(root, sel)`, `LANG_NAMES`, `LANG_CODES`.
- Çeviri eklemek / düzeltmek: üç sayfadaki ilgili satır (aynı anahtar birden çok sayfada olabilir: `settings`, `pause`, `ad`, `home`, `music`, `on/off` …) elle düzeltilir. HTML'deki Türkçe yazılar yalnızca yedek; açılışta `applyTexts()` seçili dille değiştirir.
- Arapça ve Urduca: yalnızca yazı kaplarına `dir="rtl"` (kartlar, öğretici kutusu, kural şeridi, jump başlama yazısı, launcher bildirimi); oyun yerleşimi değişmez.
- Yazı tipi: Baloo 2 (Latin + Devanagari). Dil seçilince eksik yazı ailesi Google Fonts'tan yüklenir (`LANG_FONTS`): zh → Noto Sans SC, ar/ur → Baloo Bhaijaan 2, bn → Baloo Da 2, ru → Nunito. `--font` yığınında ayrıca sistem yedekleri (Noto Sans, Noto Sans Devanagari/Bengali, Noto Naskh Arabic, PingFang SC, Microsoft YaHei, system-ui) var; kutu görünmez. ar/ur/hi/bn/zh'de harf aralığı 0.
- Uzun çeviriler: kart başlığı, satır tuşu yazıları ve tuş yazıları sığmazsa `fitText` yazıyı en çok %40 küçültür (kart açılınca, yeniden boyutlanınca, yazı tipi gelince). Puzzle plakasındaki kelime (`BÖLÜM` / çevirisi) rakamlarla birlikte sığacak kadar küçülür (`fitPlaque`, en az 5cqw).
- Geri/ileri ile önbellekten açılan sayfa başka dilde kaldıysa yenilenir (`pageshow`).

## Açılış (`index.html`)
- `menu.webp` (941×1672, dik): üst yarı "Mappy Puzzle" afişi, alt yarı "Mappy Jump". Ayırıcı mavi/altın çubuk y ≈ 845–890, bölme çizgisi y = 867.
- Yükleme: yumuşak zemin, menü resminin soluk gri kopyası; renkli resim soldan sağa açılır, en altta ince ilerleme çizgisi (5u). İlerleme = gerçek yükleme: önce `menu.webp`, sonra iki oyunun ana dosyaları (puzzle kare/pati/ui/ik/rakam resimleri, fare sprite'ları, `muzik.mp3`, `pop-1`, `ses-fare`; jump `fare.webp`, `ip.webp`, `baslangic.mp3`, `muzik-jump.mp3`, `ziplama-1..3.mp3`). Resimler `new Image()`, mp3'ler `fetch` ile; hata / 8 sn zaman aşımı da "bitti" sayılır. En az 1.2 sn sürer.
- Menü: resim ekranı doldurur. Dik ekranda (en/boy ≤ 0.75) cover, ama yanlardan en çok 22 px (kaynak) kesilir ki logolar kesilmesin; kalan üst/alt şerit aynı resmin bulanık açık kopyasıyla dolar. Geniş ekranda contain + aynı bulanık zemin. Hiç germe yok.
- Ayırıcının üstü tam genişlik `<a href="puzzle.html" aria-label="Mappy Puzzle'ı oyna">`, altı `<a href="jump/index.html" aria-label="Mappy Jump'ı oyna">`; her biri resmin kendi yarısını gösterir. Basınca o yarı %97.5'e küçülür + hafif parlar, 140 ms sonra sayfa açılır.
- Sol altta müzik tuşu (`ik-muzik` / `ik-muzik-kapali`, `aria-pressed`), `muzik.mp3` ilk dokunuştan sonra, ayara göre. Sağ altta tam ekran (`ui-fullscreen`, dikey kilit; Android uygulamasında yok).
- Dil tuşu (`#lang`): `ik-bolum-kare.webp` (krem kare, altın kenar), ayırıcı çubuktaki yıldız rozetinin tam üstünde: resimde merkez (471, 865), genişlik 130 px (rozet kalkanı ~412–532 × 807–925); konum menünün aynı cover/contain ölçeğiyle hesaplanır, her ekranda rozette kalır. İçinde kalın koyu yazıyla dil kodu (TR, EN, 中文, HI, ES, FR, AR, BN, PT, RU, UR; yazı = tuş genişliği × 0.3). Her dokunuş sıradaki dile geçer, `mappyLang`'a yazar, aria-label'ları günceller ve üstte 1.4 sn küçük beyaz yuvarlak kartta dilin kendi adını gösterir (Türkçe, English, 中文, हिन्दी, Español, Français, العربية, বাংলা, Português, Русский, اردو). Tuş z-index 4, `pointerdown`/`click` yayılımını durdurur; alttaki oyun bağlantılarını tetiklemez.

## Mappy Puzzle

Queens tarzı mantık bulmacası: her satır, sütun ve renkli bölgede 1 polis fare var, fareler birbirine (çapraz dahil) değmez.

## Oynanış
- Tek dokunuş: pati koyar ya da kaldırır (kapalı ↔ pati, "burada yok" işareti). Hemen olur, kontrol edilmez, bedava; 8 pop döngüsü sesi.
- Çift dokunuş (aynı kareye 300 ms içinde 2. dokunuş): açma denemesi. 1. dokunuşun değişikliği geri alınır, sonra fare varsa çıkış + idle animasyonu ve fare sesi; yoksa −1 can, titreme, kısa siyah-beyaz, titreşim ve kare pati olarak kalır. Açık fareye dokunmak bir şey yapmaz. `dblclick` yok sayılır (yakınlaştırma olmaz).
- Sol üstteki ilk tuş (`ik-yeniden`, "Bölümü baştan başla"): aynı bulmacayı baştan başlatır.
- 3 can (kalp paneli). Can bitince ödüllü reklam = canlar dolar (`Ads.rewarded('can')`).
- Ampul (5 hak): fare olamayacak en fazla 5 kareyi patiler. Büyüteç (3 hak): bir fareyi açar. Hak bitince reklam +2.

## Ekran yerleşimi (yukarıdan aşağı, 360×740'ta kaydırmasız)
- Ölçek: tüm boyutlar `--u` biriminin katı (`calc(N*var(--u))`). `fitLayout()` (yüklenince, resize, orientationchange, tam ekran, yazı tipi yüklenince, her tablo kuruluşunda): u = max(min(genişlik/390, yükseklik/844), min(1, genişlik/360)) px; CSS'te `min(100vw/390, 100dvh/844)` yedeği. 360–390 px telefonda u = 1, 1072×1918'de ≈ 2.27; yatayda yükseklik belirler. Sahne genişliği sınırsız; kartlar (`--cw` = min(88vw, 360u)) da u ile büyür.
- `--uc` (üst/alt blok birimi, ≤ u): sahnenin içinde `--u` = `--uc` (üst çubuk, kural şeridi, yardım tuşları, tam ekran tuşu, reklam bandı). `fitLayout()` blok yüksekliğini ölçer (uc ile doğrusal) ve uc = (yükseklik − tam genişlik tablo boyu) / blok, en az 0.72u, en çok u. Yine sığmazsa tablo küçülür (`.board-wrap` formülü). Kartlar sahne dışında, `--u` ile kalır. Gösterge sırası kalktığından (2026-11-18) uc çoğu telefonda u'ya döndü. Ör.: 360×740 uc = 1 (kare 57 px), 390×844 uc = 1 (kare 62 px), 1072×1918 uc ≈ 2.26 (u 2.27), 768×1024 ve 1366×768'de uc tabanda (0.72u), tablo yüksekliğe göre ortalı.
- Tablo tam genişlik: sahnenin yan boşluğu yok (yalnızca `env(safe-area-inset-*)`), tablo çerçevesinin dış kenarı ekranın sol/sağ kenarına değer.
- `--bw`: tablo çerçevesinin genişliği (en az üst çubuğun sığdığı 352uc). Üst çubuk ve kural şeridi bu genişlikten 16uc dar (her yanda 8uc iç boşluk), benzetici banner tam `--bw`; tam ekran tuşu çerçevenin sağ kenarından 8uc içeride.
- Üst çubuk tek sıra (grid `auto 1fr auto`): solda `ui-ayar`, `ui-pause`, `ik-yeniden` yan yana (42u, en az 40 px, arası 3u); ortada bölüm plakası (en çok 180u, orta sütuna sığar; yazı/rakam plakayla orantılı `cqw`) ve altında 3 kalp (`#lives.hearts`, çerçevesiz, 22u, `kalp-dolu` / `kalp-bos`, `role="img"` "Kalan can: x / 3", can bitince `.out` gri/soluk); sağ üstte para paneli (en az 72u).
- Gösterge sırası yok (2026-11-18 kaldırıldı): süre ekranda gösterilmez ama arka planda işler (yıldız, ZP, kazandın kartındaki "Süre"); bulunan/N sayacı yerine bölge sekmesi ilerlemeyi gösterir. Plakadaki yıldızlar da kalktı (`lastStars` kayıtta duruyor, kazandın kartı yıldız gösterir).
- Kural şeridi (`.rules-slot` içinde): beyaz yuvarlak şerit, 3 küçük kart (12 px kare/pati/fare ile 3×3 mini resim + yazı): "Her renkte 1 fare", "Her satır ve sütunda 1 fare", "Fareler birbirine değmez". Öğretici yazı kutusu (ve bölüm 1 kural balonu) açıkken şeridin yerini alır (şerit gizlenir, kutu en az şerit boyunda, uzunsa aşağı uzar); kutu kapanınca şerit geri gelir.
- Bölge sekmesi: çerçevenin üst kenarına yapışık, çerçeve renginde sekme (`#regTab`, `role="img"`, "Bulunan fareler: x / N"). Her bölge için sırayla mini kare (düz `kare-N` + `fare-cikis` 0. kare = boş delik), boyu tablo karesinin yarısı (`47.7em / n`). O bölgenin faresi bulununca aynı sprite oynatıcıyla (`startPet`) çıkış → idle. Her bölüm başında / baştan başlatınca yeniden kurulur.
- Tablo: olabildiğince büyük, kalan yüksekliğe sığar (`.play` boy kabı; genişlik = min(sahne içi, (yükseklik − 92u) / (1.018 + 0.477/N))). Boşluklar yarıya indi: `.tray` kenar dolgusu 1.3em, `.well` dolgusu 1em, karenin hücre içindeki payı her yanda %1.5 (resim %97); fare `.pet` %85.2 (resim karenin %87.8'i), ortalı. (1em = tablo genişliğinin %1'i.) Üst blok yukarıda, tablo + yardım çubuğu kalan alanda dikey ortalı.
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
- Duraklat kartı (aynı satır düzeni): `ik-oynat` Devam et, `ik-ev` Ana sayfa (→ `index.html`, açılış menüsü). Kazandın kartında `ik-kupa`. Can bitti kartında `ik-reklam` + `ik-yeniden`.

## Reklam katmanı (`Ads`)
- `Ads.provider`: Android uygulamasında `AdMobProvider` (gerçek AdMob, bkz. "Android / AdMob"), tarayıcıda `SimAdProvider` (2 sn "Reklam" kartı + çubuk). Sözleşme: `showRewarded(cb, failCb)`, `showInterstitial(cb)`, `showBanner()`, `hideBanner()`, (AdMob'da) `ready()`.
- `Ads.rewarded(placement, onReward)`: yerler `can`, `ampul`, `buyutec`. Teklif tuşu ekrana gelince `rewardedOffered`, ödül verilince `rewardedWatched` artar. Reklam gelmezse "Reklam şu an yok" kartı.
- `Ads.interstitialAfterLevel(level, then)`: "Sonraki bölüm"e basınca. Öğreticide, `noAds` oyuncuda ve bölüm 4'ten önce yok. O bölümde can bittiyse sonraki 1 (iki kez bittiyse 2) geçiş reklamsız (`adCalm`). Son reklamdan beri en az 3 dk ve en az N bölüm: C 3, B 4, A 5. Sayaç `interstitialShown`.
- Profil (`profile`): D = `noAds`; 5 tekliften önce B; izleme oranı ≥%60 A, %20–60 B, <%20 C.
- A oyuncusu: bir yardım hakkı 0'a inince bir kez "+2 hak" kartı kendiliğinden açılır (`adProactiveDone`).

## Dosyalar
- `kare-N` / `pati-N`: pastel bölge kareleri, 9 renk (0 kırmızı, 1 sarı, 2 mavi, 3 mor, 4 pembe, 5 turkuaz, 6 turuncu, 7 yeşil, 8 kahve). `fare-cikis` / `fare-idle`: fare sprite şeritleri (12 kare, 170 ms).
- `ui-*`, `rakam-*`, `kalp-dolu` / `kalp-bos` (pastel): arayüz görselleri.
- `ik-*` yuvarlak pastel tuşlar: `ik-ampul` (ipucu), `ik-buyutec` (fare göster), `ik-reklam` (reklam izle), `ik-muzik(-kapali)`, `ik-ses(-kapali)`, `ik-titresim(-kapali)`, `ik-ev`, `ik-yeniden`, `ik-oynat`, `ik-kapat`, `ik-kupa`.
- Yardım tuşlarındaki sayı rozeti: 28×28 `ui-rozet` dairesi (sağ üst), içinde 18 px rakam; hak bitince 28×28 `ui-arti`.
- Henüz kullanılmayan (bölüm haritası / günlük hediye için): `ik-kilit`, `ik-tik`, `ik-hediye`. `ik-bolum-kare` launcher'da dil tuşu.
- Sesler: `muzik.mp3` (arka plan döngüsü), `pop-1..8.mp3` (pati sesleri, sırayla döner), `ses-fare.mp3` (fare açılma).
- Tema: pastel (2026-11-18). Çerçeve: farenin halkası renginde açık lila-gri kenar, iç zemin (`.well`) farenin deliğinde arkasındaki açık tondan (~#3f3640) biraz aydınlık: yumuşak radyal #4a4152 (orta) → #3b3341 (kenar), üstte hafif iç gölge.

## Mappy Jump (`jump/index.html`, dikey, sonsuz)
Fare kendi kendine sonsuza dek zıplar; oyuncu yalnızca sola / sağa yön verir. Düşüp ekranın altından çıkarsa oyun biter.

### Kontroller
- Sol / sağ alt köşede büyük yuvarlak pastel tuşlar (SVG ok; PC'de de görünür). Ekranın sol / sağ yarısına basılı tutmak da yön verir (çoklu dokunuş). Klavye: ← → ve A / D. Esc / P duraklatır.
- Yatay: ivme 32 b/sn², en çok 5.5 b/sn, ters yöne dönüşte ivme ×1.8, tuş yokken sürtünme 20 b/sn². İpte dururken hız birikir, fırlayınca taşınır. Ekran kenarında durur (karşıya geçme yok).

### Ölçek ve döngü
- 1 birim = ekran yüksekliği / 10 (her ekranda aynı oynanış). Skor metre = birim.
- Oyun sütunu: en çok yükseklik × 0.62 geniş, geniş ekranda ortada (dışı hafif koyu). Telefonda tam ekran genişliği.
- Tuval DPR'ye göre (en çok 2.5), requestAnimationFrame + sabit fizik adımı 1/120 sn. Sayfa gizlenince Duraklat kartı açılır.

### Fare (`fare.webp`, 6×2, kare 202×256, 130 ms)
- Çizim yüksekliği 1.3 birim, ayak yarı genişliği (yakalama) 0.2.
- Kare seçimi: girişte ve ipte beklerken 0; ipe değince 0–2 (çömelme, 130 ms); yükselirken 3–6 (hıza göre); düşerken 7–11. Sola giderken yatay çevrilir.

### İp (`ip.webp`, 6×2, kare 360×130, 110 ms)
- Boy (2026-11-18): tüm ip resmi uçlar dahil aynı oranla ölçeklenir. Başlangıç eskinin %90'ı: uzunluk 1.8, kalınlık 0.648 birim (ilk ip 2.16 uzun). Yükseldikçe her 19 m'de %1 küçülür (`sizeAt(y) = max(0.5, 1 − 0.01 × floor(y / 19))`): 60 m %97, 150 m %93, 270 m %86, 420 m %78, 600 m %69, 760 m %60, 950 m'de taban %50 (uzunluk 0.9). Kalınlık en az 0.38 birim (≈ %59'da, ~780 m'den sonra sabit) — görünür ve basılabilir kalsın. Her ip kendi `w` / `h` değerini taşır.
- 3 parçalı çizim: altın uçlar kalınlıkla orantılı (kaynak 24 px), orta kısım ipin uzunluğuna uzar/kısalır. İpin üst yüzeyi karede y = 12 px (kalınlıkla ölçeklenir). Çarpışma genişliği çizilen genişliği izler: |dx| ≤ 0.46 × w + 0.2.
- Çökme (ortada, kaynak px, ipin kalınlığıyla ölçeklenir): 0, 16, 45, 73, 85, 64, 45, 30, 16, 7, 1, 0. Fare ipe inince animasyon bir kez oynar, fare çökmeyle aşağı iner (kenara yakın indiyse daha az: 1 − (2dx/w)², en az 0.25), 0.66 sn sonra (6. kare, ip geri yaylanırken) fırlar. Boşta ip düz (kare 0).
- Tek yönlü çarpışma: yalnızca düşerken ve ayaklar ipin çizgisini yukarıdan geçerken yakalar; alttan geçerken çarpmaz.
- Zıplama tepesi: ipin çizgisinin 3.15 birim üstü (yerçekimi 22 b/sn²; fırlama hızı buna göre hesaplanır). İlk aralık 2.2 → 0.95 birim pay.

### Kamera
- Yalnızca yukarı: son inilen ip ekranın altından 2.2 birim yukarıda kalacak şekilde yumuşak kayar. Farenin tepesi ekranın altına inerse 0.35 sn sonra oyun biter.

### İplerin yıpranması
- İlk ip (başlangıç, 2.4 birim) hiç kopmaz, hep mavi.
- Diğer iplerin hak sayısı (budget) var. Her inişten sonra renk basamağı = round(5 × iniş / budget): 0 mavi, 1 turkuaz, 2 yeşil, 3 sarı, 4 turuncu, 5 kırmızı (son izinli iniş kırmızı yapar). Renk değişince kısa beyaz parlama + küçük parçacık.
  - budget 5: turkuaz, yeşil, sarı, turuncu, kırmızı · 4: turkuaz, sarı, turuncu, kırmızı · 3: yeşil, sarı, kırmızı · 2: sarı, kırmızı.
- Kırmızıyken iniş: ip ortadan kopar (iki yarı dönerek düşer ve solar, parçacık patlaması, `pop-1` sesi, titreşim 80 ms), fare içinden düşer.
- Çürük ip (budget 1, yalnızca 600 m üstü, %50): ilk inişte zıplatır, kırmızı olur ve fare fırlarken kopar (tekrar konulamaz).
- Renkler (2026-11-18 yeni palet): mavi (orijinal) → turkuaz/camgöbeği → taze yeşil → güneş sarısı → kehribar turuncu → kırmızı (eski kırmızı aynen). Yüklemede sprite sayfasının 6 boyalı kopyası hazırlanır: piksel piksel HSL, yalnızca mavi ip pikselleri (ton 165–262, doygunluk ≥ 0.22) boyanır; altın uçlar kalır, burgu gölgeleri ton/açıklık farkıyla korunur. `ROPE_TINT` = [ton, ton yayılımı, doygunluk ×, açıklık ×, açıklık +, açıklık tavanı]: turkuaz [186, .30, 1, 1, .05, .90], yeşil [118, .25, .92, 1, .05, .90], sarı [50, .12, 1, 1.15, .12, .93], turuncu [32, .12, 1, 1.05, .07, .90] (kırmızıya kaymasın diye dar yayılım), kırmızı [0, .40, 1.15, .95, 0, .90]. Parçacık renkleri `ROPE_CSS`: #2f8fe8, #25c6e0, #3cc54a, #ffd83a, #ff9a1f, #e8333a. Piksel okunamazsa (file://) `hue-rotate` yedeği (uçlar da boyanır).

### Zorluk (ipin yüksekliğine göre)
| Bölge | Yükseklik | Hak | Aralık | Yan kayma (merkezden) | Kayan ip | Kayma hızı |
|---|---|---|---|---|---|---|
| 1 sabah | 0–60 m | 5 | 2.20 | 0.6–1.4 | – | – |
| 2 ikindi | 60–150 | 4 | 2.35 | 0.9–1.8 | %15 | 0.5 b/sn |
| 3 gün batımı | 150–270 | 3 | 2.50 | 1.2–2.2 | %30 | 0.7 |
| 4 alacakaranlık | 270–420 | 2 | 2.60 | 1.4–2.4 | %45 | 0.9 |
| 5 gece | 420–600 | 2 | 2.70 | 1.7–2.8 | %60 | 1.1 |
| 6 uzay | 600+ | 2 (%50'si 1 = çürük) | 2.75 | 1.9–3.1 → 950 m'de 2.2–3.5 | %70 | 1.3 |
- İp boyu artık bölge tablosunda değil, `sizeAt(y)` eğrisinde (bkz. İp).
- Aralık, yan kayma ve kayma hızı bir sonraki bölgeye doğru yumuşakça artar (son bölgede yan kayma 950 m'ye kadar artar); hak bölge sınırında düşer.
- Kayan ipler: genlik 0.45–0.85 birim (ekrana sığdırılır), sinüs.
- Adalet (2026-11-18): havada gidilebilen yol `reachAt(aralık) = 0.85 × 5.5 × havada kalma`, havada kalma = √(2·3.15/22) + √(2·(3.15 − aralık)/22) (fırlayıştan bir sonraki ipin yüksekliğine düşene kadar; aralık 2.2 → 0.83 sn → 3.88 birim, 2.75 → 0.73 sn → 3.39 birim). En kötü yol = yan kayma + önceki ipin yarısı (uzak ucundan kalkış) + iki ipin kayma genliği; bu ≤ reach olacak şekilde önce yeni ipin genliği kısılır (kayan ipte en az 0.45 kalır), sonra yan kayma. İpte dururken yan hız birikebildiği için fırlayışta tam hız alınabilir.
- Kusursuz yön botu (yerel /home/claude/tools/jump_bot.js, repoda değil; sahte DOM ile gerçek sayfa kodu): 40 deneme (390×844, 360×740, 1366×768, 1072×1918; 10'ar tohum), her biri 900 sn oyun = 1716 m, hiç düşmedi. 390×844'te ortalama / en büyük yan kayma: 0–150 m 1.34 / 1.93, 450 m 2.24 / 2.77, 750 m+ ~2.3–2.5 / 2.94 (eskiden en çok 2.7).

### Gökyüzü
- Dikey renk geçişi + birkaç yumuşak tuval bulutu (kameranın 0.3 katı hızla parallaks, yavaş yatay kayma) + yıldızlar (gece / uzayda kıpırdar).
- Bölgeler zorluk sınırlarında (60/150/270/420/600 m, ±15 m'de yumuşak geçiş): sabah mavisi → ikindi → gün batımı pembe/turuncu → alacakaranlık moru → yıldızlı gece → uzay laciverti. Bulutlar gece soluklaşır.

### Giriş
- Önce "Başlamak için dokun" kaplaması (ses kilidi açılır). Yazı tuş gibi değil: "EN İYİ" hapıyla aynı yarı saydam beyaz hap, mor yazı, ekranın ortasında gökyüzünde (alttan 50vh), hafif nabız; tüm ekran dokunulabilir. Sonra fare ilk ipte 0. karede durur, `baslangic.mp3` (~4.2 sn) çalar; bitince (ya da 4.5 sn sonra, ses kapalı / engelliyse hemen) zıplama başlar ve müzik girer.
- "Tekrar oyna" / "Yeniden başla"da giriş sesi yok: 0.7 sn bekleyip başlar.

### Sesler
- `ziplama-1/2/3.mp3` her fırlayışta sırayla (1, 2, 3, 1 …), Ses ayarına uyar. Müzik `jump/muzik-jump.mp3` (13.75 sn, daha hızlı ve neşeli döngü; puzzle/launcher `muzik.mp3` kullanmaya devam eder) döngü (.35) yalnızca oyun akarken; kartlarda, reklamda ve sayfa gizliyken susar. Tuşlarda `../pop-1.mp3`.

### HUD
- Üst orta: skor (`../rakam-*`, 46u) + "m"; altında "EN İYİ" hapı (20u rakam). Sol üst: Ayarlar (Müzik / Ses / Titreşim, kendi kaydı) ve Duraklat (Devam et / Yeniden başla / Ana sayfa → `../index.html`). Sağ üst: tam ekran (dikey kilit; Android uygulamasında yok).
- Skor = inilen en yüksek ipin yüksekliği (yuvarlanmış metre). En iyi kaydedilir.

### Oyun bitti kartı
- "Düştün!", yeni rekorsa "Yeni rekor!" + parıltı, SKOR ve EN İYİ (rakam resimleri).
- `ik-reklam` "Reklam izle, devam et" (ödüllü, oyun başına 1 kez; kullanıldıysa gizlenmez: gri/soluk, basılamaz, altında "Bu oyunda kullanıldı" notu): ekranın alt kısmında (görünen bir sonraki ipin bir aralık altında) taze bir ip (en az 1.8 birim, hak en az 2), fare üstünde, 0.8 sn sonra zıplar; yakındaki eski ipler silinir.
- "Ana sayfa" (mor) ve "Tekrar oyna" (altın).

### Reklam (`Ads`, jump'a ait)
- Android'de `AdMobProvider`, tarayıcıda `SimAdProvider` (2 sn "Reklam" kartı). `Ads.rewarded('devam', …)`.
- Geçiş reklamı "Tekrar oyna"ya basınca: yalnızca toplam oyun bitişi > 2 (ilk 2 oyunda hiç), son reklamdan beri ≥ 3 oyun bitişi ve ≥ 3 dk. Sayaçlar kayıtta: `runs`, `goSince`, `adLastAt`, `counters.interstitialShown / rewardedOffered / rewardedWatched`.

### Diğer
- `prefers-reduced-motion`: daha az parçacık, yıldız kıpırtısı ve bulut kayması yok, kart animasyonları yok.
- Kayıt `mappyJump.v1`: `music`, `sfx`, `vibe`, `best`, sayaçlar.

### Jump dosyaları
- `jump/fare.webp`, `jump/ip.webp`, `jump/baslangic.mp3`, `jump/ziplama-1..3.mp3`, `jump/muzik-jump.mp3`. Kökten: `ui-pause`, `ui-ayar`, `ui-fullscreen`, `ui-kart`, `ui-btn-altin`, `ui-btn-mor`, `ui-parilti`, `ik-*`, `rakam-*`, `pop-1.mp3`.
- Tuvale yazı çizilmiyor; tüm jump yazıları HTML'de ve `t()` ile çevriliyor.

### Mappy Jump — 2026-11-18 güncellemesi
- Zorlaşma 2 kat hızlı: zorluk bölgeleri 0 / 30 / 75 / 135 / 210 / 300 m (eskisi 0/60/150/270/420/600). Son bölgede yan mesafe 475 m'ye kadar artar.
- İp boyu her 9,5 m'de %1 küçülür, %50'ye ~475 m'de iner. Gökyüzü geçişleri aynı sınırlarda (±8 m).
- En üstte banner reklam alanı (56u + çentik) her zaman ayrılı; HUD onun altında. Simülatör banner'ı 2. oyundan itibaren gösterir (`Ads.banner()`), reklamsız oyuncuya hiç.

## Kazanma / kaybetme ekranı (2026-11-18, puzzle + jump)
- Kartın üstünde animasyonlu fare durur (ayakları kartın üst kenarına ~26u basar), arkasında yumuşak ışık halesi (radyal parıltı + `backdrop-filter: blur`, maskeli kenar, çerçeve yok). Boy 168u (`--celh`); kart `has-celeb` sınıfıyla o kadar aşağı iner, içerik 12u aşağıda başlar (puzzle'da alt boşluk da +8u). 360×740'ta en kalabalık kart (puzzle can bitti) dahil kaydırmasız sığar.
- Sprite'lar 6×2, 12 kare, döngü (`Celeb` nesnesi, her sayfada kendi kopyası; CSS background-position + setTimeout). `prefers-reduced-motion`: 0. kare sabit.
  - Kazanma (puzzle kazandın kartı + öğretici bitişi): 3 kutlamadan biri rastgele, art arda aynısı yok: `kutlama-alkis` (160×240, 140 ms, `ses-alkis.mp3`), `kutlama-duduk` (175×240, 140 ms, `ses-duduk.mp3`), `kutlama-trampet` (162×240, 170 ms, `ses-trampet.mp3`). Ses kart açılınca bir kez.
  - Kaybetme (puzzle can bitti, jump oyun bitti): `uzgun-aglama` (191×240, 170 ms). `ses-aglama.mp3` bir kez, bitince `ses-kayip-loop.mp3` kart kapanana dek döner. "Reklam şu an yok"tan kart'a dönüşte ağlama sesi tekrar çalmaz, doğrudan döngü.
- Sesler Ses ayarına uyar. Ses / döngü çalarken müzik susar (`Music.duck`), kazanma sesi bitince ya da kart kapanınca geri gelir. Her `openOverlay` / `closeOverlay` ve her reklam başlangıcı (ödüllü + geçiş) `Celeb.stop()` çağırır: zamanlayıcılar ve sesler hemen durur. Sayfa gizlenince ses durur; kaybetme kartı hâlâ açıksa görününce döngü sürer.
- Önyükleme: puzzle açıldıktan 1.2 sn sonra, jump "Başlamak için dokun"dan 0.6 sn sonra (`Celeb.preload()`); launcher da 4 sprite + 5 sesi yükler. Dosyalar kökte (jump `../` ile).

## Android / AdMob (2026-11-18)
- Uygulama: Capacitor 8 + `@capacitor-community/admob` (^8.3.0), paket `com.air.Mappyjump`. Android derleme dosyaları (package.json, capacitor.config.json, .github/workflows, android-ayar/, assets/) ayrı yürütülüyor; AdMob uygulama kimliği manifestte.
- Kimlikler (gerçek): uygulama `ca-app-pub-7983529419004158~5565061172` · banner `ca-app-pub-7983529419004158/1882667153` · geçiş `ca-app-pub-7983529419004158/9017221021` · ödüllü `ca-app-pub-7983529419004158/1625816161` (sayfalarda `ADMOB_IDS`).
- Algılama: üç sayfanın `<head>`'indeki küçük betik `window.Capacitor.isNativePlatform()` doğruysa `<html class="native">` ekler; JS'te `IS_NATIVE`. Eklenti `window.Capacitor.Plugins.AdMob` (yoksa `Capacitor.registerPlugin('AdMob')`).
- `html.native`: üç sayfada tam ekran tuşu gizli (`#fs`), tıklama kodu tam ekran istemez; uygulama yerelde immersive. Benzetici gri şeritler gizli. Tarayıcıda her şey eskisi gibi (tam ekran tuşu + `SimAdProvider`).
- Launcher: reklam yok; açılınca `removeBanner()` (oyundan dönünce kalan banner kalkar).
- Her oyun kendi `makeAdMobProvider(cfg)` kopyasını taşır (ortak JS yok). Açılış: önce `removeBanner()` (öbür oyunun banner'ı), `initialize({})` → `requestConsentInfo()`; `status === 'REQUIRED'` ve `isConsentFormAvailable` ise `showConsentForm()` → 1 geçiş (`prepareInterstitial({adId})`) + 1 ödüllü (`prepareRewardVideoAd({adId})`) ön yükleme. Gösterildikten / kapandıktan / hata verdikten sonra yeniden yüklenir; yükleme hatasında 15 → 30 → 60 → 120 sn arayla yeniden dener.
- Ödüllü: `showRewardVideoAd()`. Ödül yalnızca `onRewardedVideoAdReward` olayı (ya da çözülen ödül) gelirse; `cb` reklam kapanınca (`onRewardedVideoAdDismissed`). Yüklü değilse, gösterilemezse ya da ödülsüz kapanırsa `failCb` → oyunun "Reklam şu an yok" kartı + yeniden yükleme.
- Geçiş: `showInterstitial()`; kapanınca (`interstitialAdDismissed`) ya da hata / `interstitialAdFailedToShow` olunca oyun devam eder (`cb`). Yüklü reklam yoksa (`provider.ready()` yanlış) hiç bekletmez ve sayaçlara (`adLastAt`, `interstitialShown`, `goSince` / `adLevelsSince`) dokunmaz.
- Yedek: kapanma olayı gelmezse sayfa yeniden görünür olunca 1.5 sn sonra oyun devam eder.
- Banner: `showBanner({adId, adSize:'ADAPTIVE_BANNER', position, margin:0})` ilk kez, sonra `hideBanner()` / `resumeBanner()`; yüklenemezse (`bannerAdFailedToLoad`) bir sonraki istekte ya da bekleme sonrası yeniden. Puzzle `BOTTOM_CENTER` alt bant (`#adBand`, bölüm > 5), Jump `TOP_CENTER` üst bant (`#adTop`, 2. oyundan itibaren). `bannerAdSizeChanged` yüksekliği (dp = CSS px) `--adh` olarak yazılır: puzzle bandı = `--adh` (+ alt güvenli alan; `fitLayout` bu sabit px'i ölçekli bloktan ayrı hesaplar), jump bandı ve HUD (`--band`) = `--adh` + üst güvenli alan. Banner yokken bant 0.
- Müzik tam ekran reklam boyunca susar (`Music.hold`), sonra devam eder. Tüm çağrılar try/catch + `.catch`; reklam hatası oyunu bozmaz.
- Sıklık kuralları aynı (puzzle geçiş kuralları, profil, ödüllü +2 hak / can; jump ilk 2 oyunda geçiş yok, ≥3 oyun bitişi + 3 dk, oyun başına 1 ödüllü devam).
