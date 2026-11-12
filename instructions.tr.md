# OpenSlicer Studio — AI-Assisted Development Instructions (`instructions.tr.md`)

[English Version](instructions.md) | **Türkçe Sürüm**

Bu belge, **OpenSlicer Studio** projesini gelecekteki oturumlarda yapay zeka (AI) asistanlarıyla geliştirirken bağlam kaybını önlemek, kredi/token tüketimini en aza indirmek ve mevcut mimari kuralları korumak amacıyla hazırlanmıştır.

`./Backup` klasörü kod yedekleri içindir; bu klasörü okumana gerek yoktur.

---

## 1. Proje Dosya Yapısı ve Kurallar
* **Ana Uygulama Dosyası:** [index.html](file:///d:/Projeler/Open%20Slicer/index.html) (`6,843` satır)
  * Proje **Single-File Application** (Tek Dosya HTML + CSS + JS) mimarisindedir. Harici derleyici/bundler (Webpack, Vite) kullanılmaz; doğrudan modern tarayıcılarda çalışır ve GitHub Pages ile tam uyumludur.
  * Bağımlılıklar CDN üzerinden yüklenir:
    * `Three.js r128` (`three.min.js`, `OrbitControls.js`, `TransformControls.js`, `STLLoader.js`, `OBJLoader.js`)
    * `JSZip 3.10.1` (`.3mf` okuma ve `.zip` toplu çıktı üretimi için)
    * `ClipperLib 6.4.2` (`clipper.js` — 2B Boolean birleşim, fark ve paralel ofset işlemleri için)
* **Lisans:**
  * Proje resmi olarak **GNU General Public License v3.0 (GPLv3)** lisansıyla lisanslanmıştır ([LICENSE](file:///d:/Projeler/Open%20Slicer/LICENSE)).
* **Sürümleme Kuralı (Strict Versioning Rule):**
  * Üst barda gösterilen sürüm `beta 1.5` olarak ayarlanmıştır (`.app-version-badge`).
  * **Önemli:** Sürüm numarası her geliştirme/sohbette otomatik arttırılmaz. Versiyon arttırımı **yalnızca kullanıcı açıkça 'publish ediyoruz / yeni versiyon yayınlayalım' dediğinde** kullanıcının belirlediği numaraya güncellenir.
* **Destek ve Bağış Butonları:**
  * Üst barda `☀️ Light Mode` ile `? Help` arasında `Buy Me a Coffee` butonu yer alır (`https://www.buymeacoffee.com/iSuat`).
  * İleride GitHub Sponsors onayı geldiğinde yanına GitHub Sponsors butonu da eklenecektir.
* **CAD Entegrasyonu & Eklenti Dosyaları:**
  * [OpenSlicerBridge.py](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/OpenSlicerBridge/OpenSlicerBridge.py): Autodesk Fusion 360 Add-in scripti (Kullanıcının seçtiği klasöre `[Proje]-[Gövde].stl` adlandırmasıyla otomatik STL export).
  * [OpenSlicerBridge.manifest](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/OpenSlicerBridge/OpenSlicerBridge.manifest): Fusion 360 eklenti manifesti.
  * [Install_OpenSlicerBridge.bat](file:///d:/Projeler/Open%20Slicer/Fusion360_Plugin/Install_OpenSlicerBridge.bat): Eklentiyi `%APPDATA%\Autodesk\Autodesk Fusion 360\API\AddIns\` dizinine yükleyen 1-tık kurulum betiği.
* **Yedekleme Standardı (Zorunlu Kural):**
  * Ana dosyada (`index.html`) kapsamlı bir değişiklik yapmadan önce mevcut çalışan sürümü `SlicerStudio-backup-vX.html` (veya `index-backup-vX.html`) olarak `./Backup` klasörüne kopyala (Şu an en son [SlicerStudio-backup-v14.html](file:///d:/Projeler/Open%20Slicer/Backup/SlicerStudio-backup-v14.html) mevcuttur; bir sonraki yedek `Backup/SlicerStudio-backup-v15.html` olmalıdır).
  * **Kural:** Kullanıcı aksini söylemediği sürece her kod düzenlemesinden önce mutlaka bu yedekleme adımı uygulanır.
* **Sözdizimi Doğrulaması (Syntax Check):**
  * Her kod düzenlemesinden sonra tarayıcıda test etmeden önce mutlaka Node.js ile hızlı sözdizimi kontrolü çalıştır:
    ```powershell
    node -e "const fs = require('fs'); const html = fs.readFileSync('index.html', 'utf8'); const scripts = [...html.matchAll(/<script\b[^>]*>([\s\S]*?)<\/script>/gi)]; const mainJs = scripts[scripts.length - 1][1]; new Function(mainJs); console.log('JS Syntax OK! Lines:', html.split('\n').length);"
    ```

---

## 2. Kod Haritası (Section Map — `index.html`)
Dosya içinde arama yaparken tüm dosyayı okumak yerine aşağıdaki bölüm başlıklarını hedefle:

1. **CSS Stilleri (`Lines 6–510`):**
   * Dark / Light tema değişkenleri (`:root`, `body.light-theme`), Sol Akordeon Menü (`#sidebar`), Orta 3B Sahne (`#viewport3d`), Sağ Panel (`#viewport2d`), Tam Ekran Sheet Lightbox (`#sheetLightbox`), Sağ-Alt Assembly Step HUD (`#assemblyStepHud`), `.app-version-badge` ve `.bmc-link`.
2. **HTML İskeleti (`Lines 515–1160`):**
   * Üst Menü (`#topbar`): Marka logosu, `.app-version-badge` (`beta 1.5`), Undo/Redo butonları, Grid Toggle, Light/Dark Toggle, Buy Me a Coffee Linki (`.bmc-link`) ve Help butonu.
   * Sol Menü Kutuları:
     * `Box 1`: Import, Presets & Universal Live Watch (`#presetSelect`, `#fileInput`, `#btnLiveWatchFile`, `#lblLiveWatchStatus`)
     * `Box 2`: Manufacturing Settings (`#unitSelect`, `#sheetW`, `#sheetH`, `#matThick`, `#skipCount`, `#sheetMargin`, `#slotOffset`, `#chkToolDia`, `#toolDia`)
     * `Box 3`: Construction Technique (`#modeSelect` [`stacked` / `waffle` / `radial`], `#waffleBox`, `#radialBox` [`#radialCount`, `#radialRings`, `#radialCoreRadius`, `#radialCenterX`, `#radialCenterY`])
     * `Box 4`: Modify Form / Hollow (`#chkHollow`, `#hollowWall` — *Waffle ve Radial modlarında gizlenir*)
     * `Box 5`: Dowels / Pimler (`#chkDowels`, `#dowelShape`, `#dowelDia`, `#btnAutoDowel` — *Waffle ve Radial modlarında gizlenir*)
     * `Box 6`: Slice Direction (`#sliceAngX`, `#sliceAngY`, `#btnToggleSliceGimbal`)
     * `Box 7`: Assembly Steps & Preview (`#assemblySlider`, `#btnStepPrev`, `#btnStepNext`, `#btnStepPlay`, `#chkShowExtrude`)
     * `Box 8`: Export Plans (`#layoutSelect`, `#exportFormatSelect`, `#chkSvgCut`, `#chkSvgGuide`, `#chkSvgLabel`, `#labelFontSize`, `#btnExportZip`, `#btnExportSingleMaster`)
   * Orta Viewport (`#viewport3d`): Üst Toolbar (`Move/Rotate/Scale/Lay-Flat/Measure`), Kamera/ViewCube Canvas (`#viewCubeCanvas`), Navigasyon Butonları (`#navBtnZoom`, `#navBtnPan`, `#navBtnFit`).
   * Sağ Panel (`#viewport2d`): Model Sorunları (`#issuesCard`) ve Parça Dizilimi (`#cutLayoutContainer`).
3. **JS Bölüm 1–2 (`Lines 1160–1580`): UI Ayarları, Presets & 3B Sahne / Devasa CNC Ölçekleme:**
   * `updateDynamicGridAndFloorDimensions()`: 5m–10m+ CNC projelerinde sahne boyutuna (`sceneSpan`) göre grid hücrelerini (50/100/200/500mm), kamera `near`/`far` sınırlarını (`near = sceneSpan / 2800`, `far = sceneSpan * 14`) ve taban ölçülendirme çizgilerini dinamik ölçekler.
   * Magnifying glass zoom damping (`orbit.zoomSpeed = 0.18`).
4. **JS Bölüm 3–4 (`Lines 1585–2160`): OrcaSlicer Sabit Çentikli İletki Gimbalı & Flicker-Free ViewCube:**
   * `rebuildCustomOrcaSliceGimbal()`: Dış çember sabit durur, fare çember içindeyken 5° snap, dışındayken 0.1° serbest dönüş yapar.
   * `drawViewCube()`: 6 yüz ($[-0.505, 0.505]$), 12 pah kenarı ve 8 köşe ($[0.51, 0.81]$) geometrik olarak ayrıştırılmış ve `polygonOffset` eklenerek Z-fighting / gri titreme tamamen giderilmiştir.
5. **JS Bölüm 5–7 (`Lines 2165–2730`): Dowels, Model Yükleme & OrcaSlicer Lay-on-Face:**
   * `buildLayFlatCandidatePlates()` & `layMeshFlatOnNormal()`: QuickHull 3B dışbükey zarf ile yüzeye oturtma.
   * Göz simgesi (`👁️`): Dilimleri gizleyip orijinal ham modeli opak (`ghostMesh`) gösterir.
6. **JS Bölüm 8 (`Lines 2735–2825`): Evrensel Tarayıcı Dosya İzleme (Universal Live File Watcher):**
   * File System Access API (`window.showOpenFilePicker`) ile seçilen herhangi bir STL/OBJ dosyasının disk üzerindeki değişimini 850ms periyotla izler; dosya değiştiğinde kamera açısını, dönüşümleri ve dilimleme parametrelerini sıfırlamadan modeli anında günceller.
7. **JS Bölüm 9–10 (`Lines 2830–4780`): ClipperLib Boolean, Dilimleme Motoru (`runSlicer`), Waffle & Radial Slices:**
   * `sliceMeshAtZ(geom, val, axis)`: 'z', 'x', 'y' ve polar 'radial' (`{ type: 'radial', cx, cy, cos, sin }`) kesitleri alır.
   * **Radial Slices (`RS`):** Dikey ışınsal kanatlar (`R-1..R-n`) Clipper yarı-düzlem kesimi $[r_{\text{core}}, R_{\text{max}}]$ ile sınırlandırılır; `getAutoRadialCoreRadius` ile merkez fin çakışması trigonometrik olarak otomatik önlenir. Açıdan bağımsız halving joint'ler, T-Bone/Dog-Bone ve Notch Factor giriş pahları uygulanır.
8. **JS Bölüm 11–13 (`Lines 4785–5765`): Dowels, Issues, 2B Nesting, DXF/SVG/EPS Export & Lightbox:**
   * `updateSliceColors()`: Cut Layout üzerinden gizlenen parçanın görünürlüğünü (`grp.visible = !excludedSliceIds.has(id)`) anında geri getiren koşul yapısı.
   * Çok katmanlı AutoCAD R12 (`AC1009`) uyumlu saf DXF (`POLYLINE` + `VERTEX`), orantılı Single-Line CNC Font (`SINGLE_LINE_FONT`).
   * `getSheetLabelFontSizes()` ve parça içi otomatik sığdırma (Auto-Fit Clamping).
9. **JS Bölüm 14–16 (`Lines 5770–6843`): Assembly Steps, 3B Etkileşim & Event Loop:**
   * Kesintisiz slider, macenta çarpışma vurgusu ve Radial dilimler için $(\cos\theta_i, \sin\theta_i, 0)$ doğrultusunda radyal içeri kayma montaj animasyonu.

---

## 3. Koordinat Sistemleri ve Matematiksel Kurallar
1. **3B Sahne Koordinat Sistemi (Z-Up):**
   * `X`: Sağ/Sol, `Y`: İleri/Geri (Derinlik), `Z`: Yukarı (Yükseklik). Baskı tablası `Z = 0` düzlemindedir.
2. **Dilimleme Eksenleri ve 2B Projeksiyon (`sliceMeshAtZ(geom, val, axis)`):**
   * **`Z` Ekseni (Stacked Slices):** Kesit düzlemi $Z = z_0$. 2B koordinatlar: $(x = X,\; y = Y)$.
   * **`X` Ekseni (Waffle X Ribs):** Kesit düzlemi $X = x_0$. 2B koordinatlar: $(x = Y,\; y = Z)$.
   * **`Y` Ekseni (Waffle Y Ribs):** Kesit düzlemi $Y = y_0$. 2B koordinatlar: $(x = X,\; y = Z)$.
   * **`Radial` Ekseni (Radial Fins):** Düşey düzlem normali $(\sin\theta_i, -\cos\theta_i, 0)$. 2B koordinatlar: $(x = \text{radial\_dist},\; y = Z)$.
3. **Geçme ve Yuva Kuralları:**
   * **Waffle:** `Y` dilimleri taban ızgarasıdır (üstten aşağıya yuva); `X` dilimleri üstten indirilir (alttan yukarıya yuva).
   * **Radial Slices:** Dairesel halkalar (`Z-1..Z-m`) dış çevreden merkeze doğru radyal yuvalar alır; radyal kanatlar (`R-1..R-n`) ise göbekten dışa doğru yatay yuvalar alır.

---

## 4. Tamamlanan Son Geliştirmeler (v1.5 Beta)

1. **`P2+` Radial Slices (`RS` - Polar / Işınsal 3B Dilimleme):**
   * Arayüze `Radial Slices` üretim modu eklendi.
   * `R-1..R-n` dikey radyal kanatlar + `Z-1..Z-m` yatay halkalar ile kilitli montaj yapısı oluşturuldu.
   * `getAutoRadialCoreRadius` ile merkez çakışmasını trigonometrik olarak otomatik önleyen dinamik yarıçap motoru eklendi.
2. **Geniş Ölçekli CNC (5m–10m+ / 5000–10000mm+) Sahne & Grid Desteği:**
   * Sahne gridi, kamera near/far sınırları ve zemin ölçü çizgileri dinamik boyutlandırıldı; devasa modellerde Z-fighting önlendi.
3. **ViewCube Titreme (Z-Fighting) Düzeltmesi:**
   * Yüz, kenar ve köşe geometrik sınırları birbirinden tamamen ayrıştırılarak dönüş esnasındaki kırpışmalar giderildi.
4. **Navigasyon Büyüteç Hız Sönümlemesi (Zoom Damping):**
   * Büyüteç aktifken zoom hızı hassas kontrol için `0.18` seviyesine çekildi.
5. **Cut Layout Gizleme/Gösterme Bugfix:**
   * Gizlenen parçaların Cut Layout'tan tekrar açıldığında 3B sahnede anında görünür olması sağlandı.
6. **Evrensel Canlı Dosya İzleyici (Universal Live File Watcher):**
   * Tarayıcı üzerinden doğrudan diskteki STL/OBJ modelini izleyerek harici CAD programlarından aktarımı sıfır kurulumla canlı hale getirdi.
7. **Kişisel Autodesk Fusion 360 Eklentisi (`OpenSlicerBridge`):**
   * Fusion 360'tan seçilen klasöre tek tıkla `[Proje]-[Gövde].stl` adıyla otomatik dışa aktarım sağlayan eklenti hazırlandı.
8. **Yazı Boyutları Düzeltmesi & Box 8 Label Size Kontrolü:**
   * Plakaya orantılı devasa yazı hatası giderildi; gerçek imalat milimetrik yazı boyutuna (4.0mm) ve parça içi otomatik sınırlamaya (Auto-Fit Clamping) geçildi.
   * Dairesel halka yuva yazılarına ışınsal açı desteği eklendi.
9. **Klavye Gezintisi (TAB Tuşu):**
   * Artırma/azaltma ve sıfırlama butonlarına `tabindex="-1"` atanarak form alanları arasında kesintisiz TAB geçişi sağlandı.
10. **GitHub Pages ve index.html Geçişi:**
    * Ana dosya `index.html` olarak adlandırıldı; GitHub Pages ve web sunucularında doğrudan yayına hazır hale getirildi.

---

## 5. Sıradaki Aşama ve Yol Haritası (Roadmap)

### ✅ Faz 1: Hızlı İmalat Çözümleri, Masaüstü Arayüzü & Undo/Redo (Tamamlandı)
1. **Masaüstü Uygulama Görünümü, İkon & Alt Durum Çubuğu:** `slicer-studio.png` uygulama ikonu, 50px üst bar, GitHub linki ve canlı alt durum çubuğu (`#statusBar`).
2. **45 Adımlı Undo / Redo Motoru (`Ctrl+Z` / `Ctrl+Y`):** Model dönüşümleri, parametreler, kavelalar ve parça gizlemeleri için tam geçmiş desteği.
3. **Z-Cut (`Sink Below Bed`):** Tabladan aşağı batırma ve $Z < 0$ geometrisini su geçirmez düz tabanla budama (`Drop to Bed` ve `Lay Flat` ile sıfırlanır).
4. **Radial: Tabla Merkezine Kilitleme (`🔒 Lock Pattern to Bed Center`):** Işınsal dilimleme merkezini `(0,0)` tabla orijinine sabitleme.
5. **Kavela Kör Delik Kontrolü (`Top Cap Layers` / `Bottom Cap Layers`):** Üst ve alt uçta kaç katman kapalı kalacağını seçebilme.

### ✅ Faz 2: Lamba, Aydınlatma, Kare/Dikdörtgen Dowel (`R-Bone`), Çoklu Dil & Bugfix Paketi (Tamamlandı)
1. **Kare / Dikdörtgen Dowel (`Square / Rect Dowel`):**
   * Box 5'te `Shape` seçeneği ölçü kutularının üstüne alındı.
   * `Round` seçiliyken `Diameter` gösterilir; `Square / Rect` seçildiğinde `Diameter` gizlenip yerine **`Width (X)`** ve **`Depth (Y)`** (dikdörtgen ve kare çıta/profil desteği) kutuları açılır.
2. **`R-Bone` (4 Yönlü Döner) Corner Relief & `Relief Type` Taşıması:**
   * `Relief Type` (`#waffleReliefType`) ayarı **2. Manufacturing Settings (`#box2`)** içine taşındı (Advanced açıksa Advanced altında, kapalıysa `Skip Layers` altında görünür; seçenekler: `Square (None)`, `T-Bone`, `Dog Bone`).
   * **Kullanıcı Notu:** Normal geçme yuvalarında R-Bone'un kullanışlı olmadığı tespit edilerek `#waffleReliefType` seçeneklerinden kaldırıldı; normal yuvalar için ileride daha kompleks bir relief stratejisi tasarlanacak.
   * R-Bone modu sadece **kare/dikdörtgen dowel delikleri ve dowel support washer deliklerinde**, eğer Relief Type açıksa (`T-Bone` veya `Dog Bone`) **otomatik** olarak uygulanır. Relief kapalıysa (`Square / None`) kavelalar da düz köşeli kalır.
   * Freze ucu çapı dowel dar kenarını aşıyorsa (`toolDia >= min(dowelW, dowelD)`) arayüzde ve Model Issues sekmesinde belirgin uyarı gösterilir.
3. **Stacked: Dowel Support (Ara Mesafe Pulları / Washers — 3B Sahne + 2B Plaka):**
   * `Skip Layers > 0` ve `Dowels` aktifken katmanlar arasındaki boşluğu korumak için kesim planına ve **3B görüntüleme ekranına** delikli ara pullar (`W-1, W-2...`) eklenir (1 katman atlandıysa boşluk başına 1 pul, 2 atlandıysa 2 pul).
4. **Stacked: Eğimli Dilimlemede Merkezden Uçlara Simetrik Dilimleme (`Tilt X = 90°` veya 1. Parça Tablaya Değmiyorsa):**
   * `6. Slice Direction` içinde eğim verildiğinde (`Tilt X = 90°` vb.) veya 1 numaralı parça yere düz basmıyorsa, dilimleme eksen merkezinden (`zMid`) iki uca doğru simetrik adımlarla başlar; böylece koltuk/kanepe gibi simetrik modeller iki uçta birebir aynı parça ile başlar.
5. **Radial: Toplu İçe Ofset (`Horiz. Ring Offset`) & Sağ Panelde Parça Bazlı Merkez Delik (`Center Hole Ø`):**
   * Dikey dilimli lamba yapımında yatay halkaları (`Z-1..Z-m`) toplu olarak içeri çekme (`offset(-inwardAmount)`).
   * Merkez delik açma özelliği sağdaki **Cut Layout (`#partActionBar`)** kısmına taşındı; seçilen her bir yatay parça (`Z-1`, `Z-2`...) için ayrı delik çapı (`partCenterHoles[id]`) girilebilir ve aktif delikli parçalar etiket (pill) olarak listelenir.
6. **Radial Dilim Seçim Bugfix (`R-*` ve `Z-*`):**
   * 3B sahnede gizli dowel düzlemlerinin tıklamaları yakalaması, 3B rib sürükleme eşiği, tek yönlü yüzey (`FrontSide`) raycast sorunu ve 2B SVG'de delik içlerinin alttaki parçaya tıklamayı engellemesi (`fill-rule="evenodd"`) düzeltildi.
7. **Harici Dil Dosyaları (`locales/en.js`, `locales/tr.js`), Tek Dilli Dinamik Help, Hayalet Çizgiler & Box 8 `Guides`:**
   * Dil metinleri `locales/` klasöründeki harici dil dosyalarına taşındı; Help penceresi aktif dilde tek dilli olarak çalışır.
   * 3B model yüklendiğinde havada asılı kalan `EdgesGeometry` tel kafesi kaldırıldı ve Box 8 `Guides (Blue)` devre dışı bırakıldı.

### 📐 Faz 3: 3D & Vektör Export Optimizasyonları (Aktif)
1. **✅ Preview Sahnesini 3D Dışa Aktarma (`.3mf` / `.stl`):**
   * Dilimlenmiş tüm parçaları 3D yazıcı dilimleyicilerine (Bambu Studio, OrcaSlicer, PrusaSlicer, Cura) aktarabilecek çok gövdeli üretim formatları Box 8'e eklendi (`📦 3D (.3MF)` ve `⬇ 3D (.STL)`).
   * `.3mf`: Her parçayı (`Z-1`, `X-1`, `R-1`, `W-1`...) ayrı ve isimlendirilmiş birer `<object>` olarak barındırır. Dilimleyiciye atıldığında "Split to Objects / Parts" yapılarak tek tıkla 3D yazıcı tablasına otomatik dizilebilir (Auto-Arrange).
   * `.stl`: Montajlı tüm parçaları dünya koordinatlarında tek parça binary STL olarak kaydeder.
2. **Vektör Arc / Bézier Fitting (Opsiyonel):**
   * DXF ve SVG çıktılarında düz çizgi yığınlarını pürüzsüz yaylara (`ARC` / `BULGE`) ve Bézier eğrilerine dönüştürme.

### 🛠️ Faz 4: Sahne Araçları, Performans, İpucu Kartları & Akıllı Geometri (Aktif)
1. **✅ Move Gizmo Koniler/Çubuklar Senkronizasyonu & Tabla Arka-Sol Köşe Triadı:**
   * Three.js `TransformControls` kütüphanesinin dik ve dar açılarda okları ters döndürürken çubuk ile ucu ayırması (detachment bug) tüm alt bileşenlerin ölçek mutlak değerleri (`Math.abs`) ve yön bayrakları kilitlenerek tamamen giderildi.
   * Tabladaki 3B XYZ eksen göstergesi (triad) modelin arka-sol köşesinde (`minX`, `maxY`) konumlandırıldı ve yeşil Y oku sahne içerisine/öne doğru (`(0, -1, 0)`) bakacak şekilde yönlendirildi.
2. **✅ Tabla Merkezleme (`🎯 Center (C)`), Belirgin Merkez Eksenleri, Orijin Noktası & Ana Gride Kilitli Alt Izgara:**
   * Üst araç çubuğuna `🎯 Center (C)` butonu ve klavye `C` kısayolu eklendi; modeli tablanın `(0, 0)` merkezine alır.
   * Merkez eksen çizgileri ($X=0$ ve $Y=0$) belirginleştirildi; `(0, 0)` noktasına dairesel CAD orijin halkası ve noktası eklendi.
   * Alt ızgara objeyi kaydırdıkça objeyle gezmez; ana grid orijinine tam kilitlidir. Objeyi kapsayacak şekilde sadece ana grid hücreleri katında (hücreden hücreye, kesilmeden) genişler ve çizgi aralığı o anki ana hücrenin tam yarısıdır (`mainCellSize / 2`).
3. **✅ Büyütülmüş 3B Yön Küpü (Autodesk ViewCube):**
   * Sağ üstteki 3B yön küpü 112px'den 140px'e büyütüldü (280×280 HiDPI canvas); 26 yüzey, kenar ve köşeye tıklama ergonomisi artırıldı.
4. **✅ Menü/Panel Elemanlarına Tıklanınca Otomatik `Select` Moduna Dönüş & Lay Flat Temizliği:**
   * Move, Rotate, Scale veya Lay Flat seçiliyken sol menü veya sağ paneldeki herhangi bir girdi kutusuna (`input`), açılır menüye (`select`) veya butona tıklandığı anda 3B araç anında güvenli `select` moduna döner.
   * Model yüklendiğinde veya başka araca geçildiğinde eski modele ait havada asılı beyaz aday plakalar anında silinir; aday plakaların köşelerine 6mm fillet yuvarlatma uygulandı.
5. **✅ Gizmo & Eksen Test Laboratuvarı (`index2.html`):**
   * Three.js `TransformControls` davranışını, eksen kopma/ters dönme durumlarını ve kamera dönüşlerini izole test etmek amacıyla ön, arka, sol, sağ, yukarı referansları ve yönlü asimetrik 3D test modeli içeren bağımsız test ortamı [index2.html](file:///d:/Projeler/Open%20Slicer/index2.html) oluşturuldu.
6. **Lay Flat Dış Teğet Temas Düzlemleri (Kırmızı Çizgi / Teğet Zarf Analizi - Notlara Alındı):**
   * Kavisli veya kademeli gövdelerde yerel iç üçgenler (mavi çizgiler) yerine, modelin masaya değeceği en dış temas noktalarını birleştiren dışbükey teğet düzlemlerin (kırmızı çizgi) aday olarak üretilmesi.
7. **Açılışta Merkezde Kare Model Yükleme / Sürükle-Bırak Alanı (Hızlı Açılış - Notlara Alındı):**
   * Sayfanın ilk açılışta hafif ve hızlı yüklenmesi; sahne merkezinde belirgin kare "Model Yükle / Buraya Sürükle" alanı yer alması; dosya sürüklendiğinde genişlemesi ve model yüklenince mevcut arayüze geçiş yapılması.
8. **Üretim Tekniği Seçilene Kadar Box 3 Alt Ayarlarının Sade/Gizli Kalması (Notlara Alındı):**
   * Kullanıcıyı açar açmaz onlarca ayara boğmamak için teknik seçilmediği sürece Box 3 altındaki parametrelerin gizlenmesi.
9. **ViewCube Temaları (SVG / PNG Kaplamaları - Notlara Alındı):**
   * Yılbaşı, endüstriyel, CAD gibi dönemsel temalara göre yön küpünün harici PNG/SVG kaplamalar alabilmesi.
10. **Sol Menü Özellik Açıklama Bildirimi (Contextual Hover Tooltip Popover):**
   * Sol menüdeki bir özelliğin üzerine gelindiğinde, sol panelin hemen sağında açıklayıcı bildirim kartı gösterilmesi.
11. **Yüksek Poligonlu / Büyük Dosyalı Modellerde Performans Optimizasyonu:**
   * Spatial Bucket / BVH aralık indekslemesi ile dilimleme hızını katlama.
12. **Akıllı Dowel Konumlandırma (Medial Axis / Et Kalınlığı Ortası):**
   * Dış duvar ile iç delik arasındaki orta hat üzerinden dowel dizilimi.
13. **Hazır Donanım Negatif Kütüphanesi (Boolean Cutouts):**
   * Ampul duyu, LED kiti, mıknatıs yuvası gibi hazır 3D modelleri boolean ile negatif çıkarma.
14. **Aynı (Özdeş) Parçaları Layout'a 1 Kez Ekleme (`Deduplicate Identical Parts` - Ertelendi, Notlara Alındı):**
   * Birbiriyle aynı olan parçaları tespit edip tek kopya (`×N`) basma seçeneği.

### ⏳ Faz 5: Gelecek Mimari Geliştirmeler (Beklemede)
1. **Folded Panels / Sac & Karton Katlama Açınımı (`FP`):**
   * 3D yüzeylerin katlama çizgileri, yapıştırma kulakçıkları ve kalınlık toleranslarıyla 2D sac/karton açınımı (Pepakura tarzı üretim). Geleceğin ana karmaşık mimari hedefi.
2. **Tam Kapsamlı Proje Kaydet / Yükle (`.3mf` / `.json`):** Sahnedeki modeli, dilimleme ayarlarını ve parça durumlarını saklama.
3. **Gelişmiş True-Polygon Nesting & Web Worker:** Sıfır fire yerleşimi ve arka plan iş parçacığı.
4. **Curve Slices (`CS`):** Eğri boyunca dilimleme.
4. **Folded Panels / Pepakura (`FP`):** Sac/karton katlama açınımı.

---

## 6. İletişim ve Çalışma Prensipleri
* Kullanıcıyla her zaman **Türkçe**, net ve özlü iletişim kur.
* Dosya ve sembol referanslarında tıklanabilir markdown linkleri (`[dosya.uzanti](file:///...)`) kullan.
* Kullanıcı *"kod yazma, önce konuşalım"* dediğinde hiçbir dosya düzenleme aracı çağırma; tasarımı ve mimariyi netleştir, kullanıcı onay verdiğinde toplu geçişle uygula.
* **Sürüm numarasını** yalnızca kullanıcı açıkça talep ettiğinde arttır.
* **Yedekleme Kuralı:** Kullanıcı aksini belirtmediği sürece ana dosyada (`index.html`) yapılacak her önemli düzenlemeden önce `./Backup` klasörüne çalışan son sürüm yedeklenir.
