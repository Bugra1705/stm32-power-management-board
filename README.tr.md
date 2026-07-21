# STM32 Tabanlı Entegre Kontrol ve Güç Yönetim Sistemi

**Diller:** [English](README.md) | Türkçe


7.4 V (2S LiPo) batarya beslemesi altında çalışan, yüksek verimli güç yönetimi ve hassas telemetri özelliklerine sahip çok amaçlı bir gömülü kontrol platformu. Kart, kontrol ünitesini ve çevre birimlerini LM2576 tabanlı anahtarlamalı regülatörlerle üretilen **3.3 V ve 5 V hatları** üzerinden besler. **ACS712** ile gerçek zamanlı akım takibi ve ADC tabanlı pil voltajı ölçümü güç tüketimini kontrol altında tutarken; **nRF24L01** kablosuz desteği, servo çıkışları ve genişletme header'ları kartı farklı uygulamalar için açık ve programlanabilir bir taban haline getirir.

`7.4V Giriş | 5V & 3.3V Çıkış | ACS712 Akım Ölçümü | nRF24L01 Kablosuz | STM32 Kontrol`

![Kart — 3D render](images/hero_3d.png)

---

## İçindekiler
- [Genel Bakış](#genel-bakış)
- [Öne Çıkan Özellikler](#öne-çıkan-özellikler)
- [Güç Yönetimi ve Voltaj Regülasyonu](#güç-yönetimi-ve-voltaj-regülasyonu)
- [MCU Güç Beslemesi ve Filtreleme](#mcu-güç-beslemesi-ve-filtreleme)
- [Mikrodenetleyici, Sistem Yönetimi ve Kablosuz Haberleşme](#mikrodenetleyici-sistem-yönetimi-ve-kablosuz-haberleşme)
- [Akım Ölçümü](#akım-ölçümü)
- [Yüksek Akım PCB Tasarımı](#yüksek-akım-pcb-tasarımı)
- [Kazanımlar](#kazanımlar)

## Genel Bakış
Bu, **STM32F103** çevresinde tasarlanmış genel amaçlı bir gömülü kontrol ve güç yönetim kartıdır. 2S LiPo (7.4 V) girişini alıp anahtarlamalı regülatörlerle 3.3 V ve 5 V hatları üretir; böylece hem MCU hem de çevre birimleri temiz ve verimli bir kaynaktan beslenir. Kart üzerindeki akım ölçümü ve pil voltajı ölçümü, yazılıma güç tüketimi konusunda tam görünürlük sağlar. Genişletme header'ları, servo çıkışları ve nRF24L01 kablosuz arayüzü sayesinde kart, bir araç sürücüsünden genel bir otonom "ana kart"a kadar farklı projelerde yeniden kullanılabilir.

## Öne Çıkan Özellikler
- **Verimli çift hatlı güç:** LM2576-ADJ anahtarlamalı buck regülatörler 3.3 V ve 5 V üretir; çıkış voltajları geri besleme direnç bölücüleriyle (R1-R2, R8-R10) hassas biçimde ayarlanır.
- **30 A akım ölçümü:** ACS712 (30 A) ESC/motor akımını gerçek zamanlı izler; ADC'ye temiz sinyal için RC filtre (R11, R12, C22) kullanılır.
- **Pil telemetrisi:** Direnç bölücü üzerinden ADC tabanlı 2S LiPo voltaj ölçümü.
- **Temiz MCU beslemesi:** Pin başına dekuplaj ve hassas ADC ölçümleri için ferrite bead + alçak geçiren filtre ile izole edilmiş analog hat (VDDA).
- **Kablosuz bağlantı:** Uzaktan kontrol ve canlı telemetri için nRF24L01 (2.4 GHz); anten altında kararlı empedans için toprak düzlemi boşluğu bırakılmıştır.
- **Genişletilebilir:** Servo çıkışları, ESC/BEC konnektörü, ST-Link header'ı, reset devresi ve birden fazla genişletme header'ı.

## Güç Yönetimi ve Voltaj Regülasyonu
İki adet **LM2576-ADJ** anahtarlamalı regülatör, 7.4 V girişten 3.3 V ve 5 V hatlarını üretir. Ayarlanabilir geri besleme bölücüleri her çıkışı hassas biçimde belirler. Regülatörlerin çevresinde geniş bakır alanlar (polygon pour) ısınmayı ve gerilim düşümünü yönetir; filtre kapasitörleri, yüksek frekanslı gürültüyü bastırmak ve çıkış dalgalanmasını (ripple) en aza indirmek için regülatör giriş/çıkış pinlerine fiziksel olarak mümkün olan en yakın konuma yerleştirilmiştir.

![Güç regülasyon şematiği](images/power_sch.png)

## MCU Güç Beslemesi ve Filtreleme
STM32'nin güvenilir çalışması için özenli bir filtreleme mimarisi uygulanmıştır:
- **Dekuplaj kapasitörleri:** Datasheet'e uygun olarak her besleme pini için ayrı 100 nF kapasitör; yüksek frekanslı gürültüyü çekirdekten uzak tutar.
- **Analog besleme (VDDA) izolasyonu:** VDDA hattı, hassas ADC ölçümleri için dijital gürültüden izole edilmek üzere bir ferrite bead (FB1) ve alçak geçiren filtre katı (100 nF & 1 µF) üzerinden beslenir.
- **Durum ve çıkış:** Enerji göstergesi olarak yeşil LED (D6) ve çevre birimleri için J2 güç çıkış header'ı.
- **Komponent yakınlığı:** Tüm dekuplaj kapasitörleri MCU besleme pinlerine mümkün olan en kısa yollarla bağlanır; bu, hat endüktansını en aza indirerek güç kararlılığını artırır.

## Mikrodenetleyici, Sistem Yönetimi ve Kablosuz Haberleşme
- **STM32F103** — projenin ihtiyaç duyduğu PWM, ADC ve SPI yeteneklerini karşıladığı için tercih edildi.
- **Kristal:** 8 MHz harici kristal, 22 pF yük kapasitörleriyle kararlı ve doğru zamanlama sağlar.
- **nRF24L01 (2.4 GHz):** Uzaktan kontrolü sağlar ve kritik telemetriyi — ACS712'den gelen akım ile pil voltajı — gerçek zamanlı olarak kullanıcıya geri gönderir. RF tasarım karmaşıklığını (anten empedans eşleme vb.) azaltmak, maliyeti düşürmek ve hızlı değişime imkân tanımak için harici modül olarak kullanılır. Layout'ta, modülün anteninin altındaki katmanlarda toprak düzlemi bilerek dökülmemiştir; bu boşluk anten empedansını kararlı tutarak menzili en üst düzeye çıkarır.

![MCU ve nRF24L01 şematiği](images/mcu_sch.png)

## Akım Ölçümü
Bir **ACS712 (30 A)** sensörü, bağlı ESC/motor sürücüsünün çektiği akımı gerçek zamanlı takip eder. Sensör 30 A'e kadar ölçüm sunarak yüksek güçlü motor kullanımı için güvenli bir aralık sağlar. Bir RC filtre (R11, R12, C22) ölçüm gürültüsünü sönümler ve MCU'ya temiz bir sinyal iletir.

![Akım ölçüm şematiği](images/current_sch.png)

## Yüksek Akım PCB Tasarımı
ACS712 çevresinde 30 A yükleri güvenle taşımak için özel bir layout stratejisi uygulanmıştır:
- **Akım taşıma kapasitesi:** Sensörün giriş/çıkış pinleri (ACS_PI+, ACS_PI-), direnci ve ısınmayı en aza indirmek için maksimum genişlikte bakır alanlarla desteklenmiştir.
- **Katman birleştirme:** Top ve bottom bakır alanlar çok sayıda stitching via ile birbirine bağlanarak etkin iletim kesiti artırılmış ve katmanlar arası direnç düşürülmüştür.
- **Lehim takviyesi:** Bottom katmandaki bakır çıplak (solder-mask opening) bırakılmıştır; böylece montaj sırasında bu alanlara lehim dökülerek 30 A altında maksimum performans için iletkenlik artırılır.

| Top katman — sinyal yolları ve yerleşim | Bottom katman — ground plane ve güç dağıtımı |
|---|---|
| ![Top katman](images/pcb_top.png) | ![Bottom katman](images/pcb_bottom.png) |

## Kazanımlar
- **Altium ve donanım tasarımı:** Bu kartta Altium'u uçtan uca kullandım — kütüphane oluşturmaktan buck-converter güç katlarını kurgulamaya kadar — ve her kararın arkasındaki mantığı işledim: hangi pin neden, hangi kapasitör değeri neden.
- **Yüksek akım ve termal:** 30 A yollarında, geniş bakırın tek başına yetmediğini; via stitching ve bottom katmanda solder-mask opening bırakmanın gerçek akım taşımada önemli olduğunu bizzat uygulayarak öğrendim.
- **Sistem entegrasyonu:** Bu kart devam eden bir projenin parçası — sonraki adım, nRF hattı üzerinden kartla haberleşen bir joystick tasarlayarak kablosuz kontrol döngüsünü tamamlamak.
- **Yazılım platformu:** Genişletme pinleri ve sensör arayüzleriyle kart, gelecekteki otonom projelerde STM32 firmware'i için yeniden kullanabileceğim esnek bir "ana kart" görevi görüyor.
