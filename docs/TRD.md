# TRD — Teknik Gereksinimler

Bu belge sistemin nasıl kurulacağını tanımlar. "Öneri" yazılan maddeler teknik öneridir; "Açık karar" yazılanlar danışmanla birlikte, belirtilen aya kadar netleştirilecek.

## Mimari

| Katman | Bileşen | Görev |
| --- | --- | --- |
| Eğitim | Projeye özel Python ortamı | Hızlı simülasyon; tüm eğitim ve karşılaştırma burada |
| Yazılımsal doğrulama | PX4 SITL + Gazebo Harmonic, çok araçlı | Çok ajanlı politikanın gerçek PX4 yazılımıyla testi |
| Donanımlı doğrulama | 1 × Cube Orange + Raspberry Pi 5 + jMAVSim (veya SIH) | Tek ajanın gerçek uçuş kontrolcüsü ve gömülü bilgisayarda testi |

## Yazılım yığını

| Bileşen | Seçim | Not |
| --- | --- | --- |
| İşletim sistemi | Ubuntu 22.04 | Mevcut |
| Dil | Python 3.10 | Ubuntu 22.04'ün varsayılan sürümü |
| Ortam arayüzü | Gymnasium; çok ajan için PettingZoo `ParallelEnv` (öneri) | |
| Derin öğrenme | PyTorch | |
| DDPG, PPO | stable-baselines3 | SB3'te DDPG ve PPO var |
| MADDPG | Projede uygulanacak | SB3'te ve SB3-Contrib'de MADDPG yok |
| PX4 iletişimi | MAVSDK-Python veya pymavlink (açık karar, 2. ay) | |
| Simülatörler | Gazebo Harmonic (SITL), jMAVSim (HITL) | PX4 multicopter HITL'i yalnızca jMAVSim ve Gazebo Classic ile destekliyor |
| Analiz | NumPy, pandas, SciPy, matplotlib | |
| Kayıt | TensorBoard + CSV | |
| Sürüm sabitleme | `requirements.txt`; PX4 bir sürüm etiketine sabitlenir (açık karar, 2. ay) | |

## Kritik tasarım kararı: MADDPG uygulaması

SB3'te MADDPG olmadığı için MADDPG'yi biz yazacağız. Bu, projenin asıl iddiasına dokunuyor: DDPG hazır kütüphaneden, MADDPG kendi kodumuzdan gelirse, aradaki fark algoritmadan değil uygulamadan kaynaklanabilir. Önlemler (öneri):

- **Ortak ağ mimarisi:** tüm yöntemlerde aynı katman sayısı, genişlik ve optimizer.
- **Doğrulama testi:** tek ajanlı MADDPG teorik olarak DDPG'ye indirgenir. Bizim MADDPG'miz tek ajanla SB3 DDPG'ye yakın sonuç vermelidir; vermezse uygulamada hata vardır.
- **Alternatif:** üç algoritmayı da tek bir kod tabanında yazmak. Adalet açısından daha temiz ama iş yükü daha fazla. **Açık karar, 4. ay.**

Ayrıca DDPG ve PPO çok ajanlı ortamda "bağımsız öğrenen" olarak kullanılacak. Her ajanın ayrı ağı mı olacak, yoksa ajanlar aynı ağı mı paylaşacak? Bu seçim tüm yöntemlerde aynı olmalı. **Açık karar, 4. ay.**

## Ortam tasarımı

**Aksiyon uzayı (öneri): hız komutu (vx, vy, vz).** PX4 Offboard modu, yardımcı bilgisayardan MAVLink üzerinden hız komutu kabul ediyor (`SET_POSITION_TARGET_LOCAL_NED`). Politika hız komutu üretirse aynı politika eğitim ortamında, SITL'de ve HITL'de değişmeden kullanılabilir; düşük seviyeli motor kontrolünü PX4 yapar. Motor itkisi üreten bir politika ise PX4'e aktarılamaz.

**Dinamik model (öneri):** hız komutunu belirli bir gecikme ve hız/ivme sınırlarıyla takip eden basitleştirilmiş bir model. Modelin parametreleri 3-4. ayda SITL'de toplanan uçuş kayıtlarından kalibre edilir; böylece eğitim ortamı PX4'ün gerçek tepkisine yaklaşır. Tam rijit gövde modeli (gym-pybullet-drones benzeri) alternatiftir. **Açık karar, 3. ay.**

| Bileşen | Tanım (öneri) |
| --- | --- |
| Gözlem | Kendi konum ve hızı; komşu ajanların göreli konum ve hızı; kendi formasyon hatası vektörü; hedefe yön ve uzaklık; en yakın engellere uzaklıklar (sabit boyutlu) |
| Ödül | Formasyon hatası cezası + çarpışma cezası + hedefe ilerleme ödülü + aksiyon düzgünlüğü cezası (ani komut değişimlerini azaltır, donanıma aktarımı kolaylaştırır) |
| Epizot sonu | Hedefe ulaşma, çarpışma veya zaman aşımı |
| Engeller | Statik silindirler; yerleşim tohumdan üretilir |
| Kontrol frekansı | Eğitim ortamında ve PX4'e komut gönderirken aynı olmalı. PX4 Offboard modunda komutların kesintisiz akışı gerekiyor; akış `COM_OF_LOSS_T` süresinden uzun kesilirse araç Offboard'dan çıkıyor. **Değer açık karar, 3. ay.** |

## Deney protokolü

| Bileşen | Tanım |
| --- | --- |
| Yapılandırma | Her deney bir YAML dosyasıyla tanımlanır; dosya ve git commit kimliği sonuçlarla birlikte kaydedilir |
| Hiperparametre araması | Algoritma başına eşit sayıda rastgele deneme (ör. 10) |
| Eğitim bütçesi | Algoritma başına eşit toplam ortam adımı (sayı ortam hızı ölçülünce, 4. ay) |
| Tohumlar | Yöntem × senaryo başına en az 5; paralel süreçler |
| Değerlendirme | Eğitimden bağımsız, sabit tohumlu bir senaryo seti; tüm yöntemler aynı setle değerlendirilir |

**Metrik tanımları (öneri):**

- **Çarpışma oranı:** herhangi bir çarpışmanın olduğu epizotların oranı.
- **Formasyon hatası (RMSE):** her ajanın formasyon merkezine göre konumu ile hedef konumu arasındaki farkın, zaman ve ajanlar üzerinden karekök ortalama karesi.
- **Görev tamamlama oranı:** çarpışmadan ve süresinde hedefe ulaşılan epizotların oranı.
- **Örnek verimliliği:** görev tamamlama oranının belirli bir eşiğe ulaştığı ortam adımı sayısı (eşik açık karar).
- **Eğitim süresi:** duvar saati süresi.
- **Çıkarım gecikmesi:** politikanın bir gözlemden aksiyon üretme süresi; iş istasyonunda ve Raspberry Pi'da ayrı ölçülür, ortanca ve üst yüzdelik raporlanır.

**İstatistik:** Tohumlar arası dağılımlarda Welch t-testi veya Mann-Whitney U, etki büyüklüğü Cohen's d. Dört yöntem ikili karşılaştırılacağı için çoklu karşılaştırma düzeltmesi (ör. Holm) uygulanmalı (öneri). 5 tohumla testlerin gücü sınırlıdır; bu nedenle güven aralıkları da raporlanmalı ve bu kısıt raporda açıkça yazılmalı.

## SITL köprüsü

- Her ajan ayrı bir PX4 SITL örneği olarak çalışır (Gazebo'da çok araçlı simülasyon resmî olarak destekleniyor).
- Köprü betikleri her aracın konum ve hızını PX4'ten okur, eğitim ortamındakiyle aynı biçimde gözleme çevirir, politikayı çalıştırır ve hız komutunu Offboard modunda kesintisiz gönderir.
- Engeller Gazebo dünyasına ve gözlem hesabına aynı yapılandırmadan konur.

## HITL kurulumu

| Adım | Detay |
| --- | --- |
| Firmware | Cube Orange için PX4 kaynaktan derlenir; yapılandırmaya `CONFIG_MODULES_SIMULATION_PWM_OUT_SIM=y` eklenir |
| Gövde ayarı | HIL Quadcopter X (`SYS_AUTOSTART` 1001) |
| Simülatör | jMAVSim, iş istasyonunda; Cube Orange'a USB ile bağlı. Alternatif: SIH (fizik kartın içinde) |
| Politika | Raspberry Pi 5 üzerinde; PyTorch (CPU) veya ONNX Runtime (açık karar, 6. ay) |
| Pi – Cube bağlantısı | JST-GH kablo ile TELEM2 portu üzerinden MAVLink; portun MAVLink için etkin olduğu kurulumda kontrol edilir |
| Güvenlik | Kart motorlara bağlı değil; mevcut parametreler QGroundControl ile yedeklenir, iş bitince geri yüklenir |

**Açık karar (6. ay): HITL'de tek ajan komşuları nasıl görecek?** Politika komşu ajanların bilgisiyle eğitiliyor; HITL'de ise tek gerçek kart var. Seçenekler:

1. **Sanal komşular:** iş istasyonu diğer ajanları kendi ortamında eşzamanlı simüle eder ve durumlarını Pi'ye gönderir. Gerçekçi ama ek bir senkronizasyon katmanı gerektirir.
2. **Tek ajanlı görev:** HITL'de yalnızca hedefe gitme ve engelden kaçınma test edilir; komşu gözlemleri sabit verilir. Basit ama formasyon davranışını ölçmez.

## Önerilen repo yapısı

```
multi-uav-formation-rl/
├── README.md
├── docs/            PRD, TRD, akış diyagramları
├── configs/         ortam, algoritma ve deney YAML dosyaları
├── src/uavform/
│   ├── envs/        formasyon ortamı
│   ├── controllers/ PID temel çizgisi
│   ├── algos/       MADDPG uygulaması
│   └── eval/        metrikler ve değerlendirme
├── scripts/         eğitim, tohum çalıştırma, analiz
├── px4/
│   ├── sitl/        SITL köprüsü ve Gazebo dünyaları
│   └── hitl/        Raspberry Pi tarafı ve kurulum notları
├── tests/
└── results/         özet sonuçlar (büyük dosyalar git'e eklenmez)
```

## Test stratejisi

- [ ] Dinamik: sıfır komutla araç sabit kalır; komut verilen hıza sınırlar içinde yaklaşır.
- [ ] Çarpışma tespiti: bilinen bir çarpışma senaryosu doğru yakalanır.
- [ ] Ödül: her bileşenin işareti ve ölçeği beklendiği gibi.
- [ ] Belirlenimcilik: aynı tohum aynı epizodu üretir.
- [ ] PID: engelsiz senaryoda formasyonu korur.
- [ ] MADDPG: tek ajanla SB3 DDPG'ye yakın sonuç verir.
- [ ] SITL köprüsü: aynı gözlemde eğitim ortamı ve köprü aynı aksiyonu üretir.
