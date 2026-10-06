# Çoklu İHA Formasyon Kontrolünde Derin Pekiştirmeli Öğrenme

Çoklu İHA formasyon kontrolü ve çarpışma önleme için **DDPG, PPO ve MADDPG** algoritmalarının, projeye özel bir simülasyon ortamında ve tek bir deney protokolü altında karşılaştırılması; çok ajanlı davranışın PX4 yazılım simülasyonunda (SITL), en başarılı politikanın ise gerçek uçuş kontrolcüsü donanımıyla tek ajan üzerinden **HITL (Hardware-in-the-Loop)** testinde doğrulanması.

- **Araştırmacı:** Betül Soyer, Konya Teknik Üniversitesi
- **Program:** TÜBİTAK 2209-A Üniversite Öğrencileri Araştırma Projeleri (başvuru aşamasında)
- **Süre:** 12 ay
- **Durum:** Planlama — ortam ve kod henüz geliştirilmedi

## Neden bu proje?

Literatürde çoklu İHA görevleri için derin pekiştirmeli öğrenme algoritmalarını karşılaştıran çalışmalar çelişkili sonuçlar raporluyor: bir çalışma DDPG'yi çarpışma önlemede üstün bulurken [1], başka bir çalışma MADDPG tabanlı bir yöntemin DDPG'yi geçtiğini gösteriyor [2]. Bu çalışmalar farklı ortam, ödül fonksiyonu ve uygulama kullandığı için farkın algoritmadan mı yoksa deney kurulumundan mı geldiği anlaşılamıyor. Ayrıca derin pekiştirmeli öğrenmede sonuçların rastgele tohuma göre belirgin biçimde değiştiği bilinmektedir [3][4].

Bu proje, deney kurulumunu sabitleyerek algoritmaları adil ve tekrarlanabilir biçimde karşılaştırmayı hedefler.

## Katkılar

1. **Formasyon görevine özel, açık kaynak çok ajanlı simülasyon ortamı** (Gymnasium uyumlu).
2. **Kontrollü karşılaştırma protokolü:** ortam, ödül fonksiyonu, gözlem/aksiyon uzayı ve hiperparametre arama bütçesi tüm algoritmalar için aynı; her yöntem en az 5 bağımsız tohumla eğitilir ve sonuçlar istatistiksel testlerle raporlanır.
3. **İki aşamalı doğrulama:** çok ajanlı senaryolar PX4'ün çok araçlı SITL ortamında test edilir. Donanım tarafında tek ajanın politikası Raspberry Pi üzerinde çalışır ve gerçek bir Cube Orange uçuş kontrolcüsüne komut verir (HITL). Simülasyondan donanıma geçişteki performans kaybı ve gömülü çıkarım gecikmesi ölçülür.

Gerçek uçuş testi kapsam dışıdır.

## Yöntem (özet)

| Bileşen | Karar |
| --- | --- |
| Görev | Üçgen formasyonu koruyarak, statik engelli 3B bir koridorda hedefe ulaşmak |
| Karşılaştırılan yöntemler | DDPG, PPO, MADDPG ve klasik PID formasyon kontrolcüsü (temel çizgi) |
| Senaryolar | 2 ve 3 ajanlı |
| Tohum sayısı | Her yöntem × senaryo için en az 5 |
| Metrikler | Çarpışma oranı, formasyon hatası (RMSE), görev tamamlama oranı, örnek verimliliği, eğitim süresi, çıkarım gecikmesi |
| İstatistik | Welch t-testi veya Mann-Whitney U; etki büyüklüğü (Cohen's d) |

## Altyapı

| Katman | Araç | Amaç |
| --- | --- | --- |
| Eğitim | Projeye özel Python ortamı + stable-baselines3 | Algoritma karşılaştırmasının tamamı |
| Yazılımsal doğrulama (SITL) | PX4 + Gazebo Harmonic, çok araçlı | Çok ajanlı politikaların gerçek PX4 yazılımıyla testi |
| Donanımlı doğrulama (HITL) | 1 × Cube Orange + Raspberry Pi 5; jMAVSim veya SIH | Tek ajanla gerçek uçuş kontrolcüsü ve gömülü bilgisayarda ölçüm |

Geliştirme ortamı: Ubuntu 22.04, Gazebo Harmonic.

## Yol haritası

- [ ] **Ay 1-2:** Pekiştirmeli öğrenme temelleri, Gymnasium ve stable-baselines3 ile ilk eğitimler, PX4 SITL kurulumu
- [ ] **Ay 3-4:** Tek İHA ortamı, PID temel çizgisi, dinamik modelin doğrulanması
- [ ] **Ay 5-6:** Çok ajanlı formasyon görevi; DDPG, PPO, MADDPG kurulumu; HITL ön denemesi
- [ ] **Ay 7-9:** Hiperparametre araması ve ana deneyler
- [ ] **Ay 10:** Çok ajanlı SITL ve tek ajanlı HITL doğrulaması
- [ ] **Ay 11-12:** İstatistiksel analiz, rapor, ortamın ve kodun yayımı

## Kaynaklar

1. M. A. Ali, A. Maqsood, U. Athar, H. R. Khanzada, "Comparative Evaluation of Reinforcement Learning Algorithms for Multi-Agent Unmanned Aerial Vehicle Path Planning in 2D and 3D Environments," *Drones*, 9(6), 438, 2025. https://doi.org/10.3390/drones9060438
2. F. Zhang, Q. Wang, X. Ma, "Multi-UAV Cooperative Path Planning Method Based on an Improved MADDPG Algorithm," *Electronics*, 15(8), 1632, 2026. https://doi.org/10.3390/electronics15081632
3. P. Henderson vd., "Deep Reinforcement Learning that Matters," AAAI 2018. https://arxiv.org/abs/1709.06560
4. R. Agarwal vd., "Deep Reinforcement Learning at the Edge of the Statistical Precipice," NeurIPS 2021. https://arxiv.org/abs/2108.13264
