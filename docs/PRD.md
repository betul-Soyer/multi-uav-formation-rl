# PRD — Ürün Gereksinimleri

Bu proje bir yazılım ürünü değil, bir araştırma projesi; "ürün" burada projenin teslim edeceği ortam, kod, deney sonuçları ve rapordur.

## Amaç

DDPG, PPO ve MADDPG algoritmalarını çoklu İHA formasyon kontrolü görevinde, deney kurulumu sabitlenmiş tek bir protokolle karşılaştırmak; sonuçları PX4 yazılım simülasyonunda (çok ajan) ve gerçek uçuş kontrolcüsü donanımında (tek ajan) doğrulamak.

## Paydaşlar

| Paydaş | Bu projeden beklentisi |
| --- | --- |
| Araştırmacı (Betül Soyer) | Projeyi yürütmek, bu süreçte alanı öğrenmek |
| Danışman | Planı onaylamak, ilerlemeyi GitHub üzerinden izlemek |
| TÜBİTAK hakemleri | Özgün değer, yöntem, fizibilite ve bütçe gerekçesi |
| Ders sorumlusu (Uygulamalı Yapay Zeka) | Aynı proje dönem projesi olarak; düzenli commit geçmişi, makale formatında rapor |
| Gelecekteki kullanıcılar | Ortamı ve protokolü kendi çalışmalarında yeniden kullanmak |

## Kapsam

**Kapsamda:** çok ajanlı formasyon simülasyon ortamı; PID temel çizgisi; DDPG, PPO, MADDPG; 2 ve 3 ajanlı senaryolar; istatistiksel karşılaştırma; çok ajanlı SITL doğrulaması; tek ajanlı HITL doğrulaması; açık kaynak yayım.

**Kapsam dışı:** gerçek uçuş testi; yeni bir RL algoritması geliştirmek; görüntü tabanlı algılama; dinamik (hareketli) engeller; 3'ten fazla ajan (zaman kalırsa isteğe bağlı).

## Fonksiyonel gereksinimler

| No | Gereksinim |
| --- | --- |
| FR-1 | Ortam N ajan (en az 2 ve 3) için formasyon görevini simüle eder: hedefe ilerleme, üçgen formasyon, statik engeller, ajanlar arası çarpışma tespiti. |
| FR-2 | Ortam aynı tohumla çalıştırıldığında aynı başlangıç koşullarını ve engel yerleşimini üretir. |
| FR-3 | Gözlem, aksiyon ve ödül tanımları tüm algoritmalar için ortak ve tek bir yapılandırma dosyasından gelir. |
| FR-4 | PID formasyon kontrolcüsü aynı ortamda, aynı metriklerle değerlendirilir. |
| FR-5 | DDPG, PPO ve MADDPG eşit toplam ortam adımı ve eşit hiperparametre arama bütçesiyle eğitilir. |
| FR-6 | Her yöntem × senaryo en az 5 bağımsız tohumla eğitilir; tohumlar paralel süreçler olarak çalıştırılabilir. |
| FR-7 | Altı metrik her değerlendirmede kaydedilir: çarpışma oranı, formasyon hatası (RMSE), görev tamamlama oranı, örnek verimliliği, eğitim süresi, çıkarım gecikmesi. |
| FR-8 | Analiz betiği tohumlar arası ortalama, standart sapma, anlamlılık testi ve etki büyüklüğü hesaplar; tablo ve grafik üretir. |
| FR-9 | Eğitilmiş politika PX4 SITL'de (Gazebo Harmonic, çok araçlı) çalıştırılabilir. |
| FR-10 | Eğitilmiş politika Raspberry Pi üzerinde çalışır ve Cube Orange'a HITL'de komut verir; çıkarım gecikmesi ölçülür. |
| FR-11 | Her deney, kullandığı yapılandırma ve kod sürümüyle (git commit) birlikte kaydedilir. |

## Fonksiyonel olmayan gereksinimler

| No | Gereksinim |
| --- | --- |
| NFR-1 | Yeniden üretilebilirlik: README'deki adımlarla başka biri aynı sonuçları üretebilir. |
| NFR-2 | Tüm eğitimler kişisel iş istasyonunda (i7-13650HX, 16 GB RAM) çalışabilir; GPU zorunlu değildir. |
| NFR-3 | Ortam, bir eğitimin takvime sığacak hızda çalışır; hız 1-2. ayda ölçülüp hedef konur. |
| NFR-4 | Kod açık kaynak yayımlanır; lisans danışmanla birlikte seçilir. |
| NFR-5 | Ders kuralları: düzenli, anlamlı mesajlı commit'ler; repo teslimde public. |
| NFR-6 | Donanım güvenliği: HITL'de motor çıkışı yok; laboratuvar kartının parametreleri iş öncesi yedeklenir ve iş sonrası geri yüklenir. |

## Başarı ölçütleri

Proje, sonuçtan bağımsız olarak şu koşullar sağlanırsa başarılıdır: FR-1…FR-8 karşılanmış, 3 ajanlı senaryo için dört yöntemin sonuçları istatistiksel testlerle raporlanmış, en iyi politika SITL'de ve tek ajanla HITL'de doğrulanmış, ortam ve kod yayımlanmış olmalı. "Anlamlı fark yok" da geçerli bir sonuçtur.

## Kilometre taşları

| Ay sonu | Çıktı |
| --- | --- |
| 2 | SITL çalışıyor; Gymnasium/SB3 ile ilk eğitimler; protokol yazılı |
| 4 | Tek İHA ortamı ve PID hazır; ortam hızı ölçüldü |
| 6 | Üç algoritma çok ajanlı ortamda eğitiliyor; HITL'de tek kart döngüsü kuruldu |
| 9 | Ana deneyler tamam |
| 10 | SITL ve HITL doğrulaması tamam |
| 12 | Rapor, ortam ve kod yayımlandı |

## Riskler

| Risk | Etki | Önlem |
| --- | --- | --- |
| Eğitim süresi takvime sığmaz | Deneyler gecikir | Ajan sayısı 2'ye, tohum sayısı 3'e iner; gerekirse bulut hesaplama |
| Politikalar formasyonu öğrenemez | Karşılaştırma anlamsızlaşır | Önce tek ajan ve engelsiz senaryoda doğrulama; ödül tasarımı erken test edilir |
| Cube Orange'da HITL kurulamaz | Donanım doğrulaması yapılamaz | SIH yerine jMAVSim; o da olmazsa SITL doğrulamasıyla yetinilir ve raporda belirtilir |
| Laboratuvar kartı erişilemez olur | HITL gecikir | HITL denemesi 5-6. aya, asıl doğrulama 10. aya konuldu; arada esneklik var |
| Araştırmacının alana yeni olması | Başlangıç yavaş olur | İlk 2 ay temel yetkinlik dönemi olarak planlandı |

## Varsayımlar ve açık sorular

- [ ] Danışman revize planı onaylayacak.
- [ ] Laboratuvardaki bir Cube Orange proje boyunca kullanılabilecek.
- [ ] Raspberry Pi 5 başvuru zamanında stokta olacak (29 Eylül'de çoğu satıcıda yoktu).
- [ ] Bütçe kalemlerinin sınıflandırması üniversitenin proje biriminden teyit edilecek.
- [ ] Açık kaynak lisansı seçilecek.
