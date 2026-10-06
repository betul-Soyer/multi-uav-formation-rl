# Akış Diyagramları

## 1. Proje akışı

```mermaid
flowchart TD
    A["Ay 1-2<br/>Temel yetkinlik<br/>SITL kurulumu"] --> G1{"2. ay<br/>SITL ve ilk<br/>eğitim tamam?"}
    G1 --> B["Ay 3-4<br/>Tek İHA ortamı<br/>PID, kalibrasyon"]
    B --> G2{"4. ay<br/>ortam hazır?"}
    G2 --> C["Ay 5-6<br/>Çok ajanlı ortam<br/>3 algoritma, HITL deneme"]
    C --> G3{"6. ay<br/>3 algoritma<br/>eğitiliyor?"}
    G3 -->|evet| D["Ay 7-9<br/>Hiperparametre arama<br/>ana deneyler"]
    G3 -->|hayır| R["Kapsam daraltma<br/>2 ajan, 3 tohum"]
    R --> D
    D --> E["Ay 10<br/>SITL + HITL<br/>doğrulama"]
    E --> F["Ay 11-12<br/>Analiz, rapor, yayın"]
```

Her kontrol noktası bir sonraki aşamaya geçmeden önce karşılanması gereken çıktıyı gösterir; 6. ayda algoritmalar eğitilemiyorsa kapsam daraltılır ama takvim korunur.

## 2. Deney hattı

```mermaid
flowchart TD
    cfg["YAML yapılandırma"] --> hp["Hiperparametre araması<br/>eşit bütçe"]
    hp --> train["Eğitim<br/>5 tohum × yöntem × senaryo"]
    train --> ev["Sabit senaryo setiyle<br/>değerlendirme"]
    ev --> log["Metrikler + yapılandırma<br/>+ git commit kaydı"]
    log --> st["İstatistik<br/>test, etki büyüklüğü"]
    st --> best["En iyi politika"]
    best --> sitl["SITL doğrulama<br/>çok ajan"]
    best --> hitl["HITL doğrulama<br/>tek ajan"]
    st --> rep["Rapor"]
    sitl --> rep
    hitl --> rep
```

Her sonuç, onu üreten yapılandırma ve kod sürümüyle birlikte kaydedildiği için rapordaki her sayı geriye doğru izlenebilir.

## 3. HITL veri akışı

```mermaid
flowchart LR
    ws["İş istasyonu<br/>jMAVSim"] -- "sensör verisi (USB)" --> fc["Cube Orange<br/>PX4 HITL firmware"]
    fc -- "aktüatör çıkışları (USB)" --> ws
    fc -- "konum ve hız (MAVLink, TELEM2)" --> pi["Raspberry Pi 5<br/>politika"]
    pi -- "hız komutu (Offboard)" --> fc
    nb["Komşu ajan bilgisi<br/>(açık karar)"] -.-> pi
```

Uçuş fiziği iş istasyonunda, uçuş kontrol yazılımı gerçek kartta, politika ise Raspberry Pi'da çalışır. Kesikli çizgi, TRD'de açık bırakılan komşu ajan bilgisinin nereden geleceği sorusunu gösterir.
