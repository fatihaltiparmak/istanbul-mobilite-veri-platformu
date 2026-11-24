# 0001 Mimari: Batch, Streaming ve Tablo Formatı

Durum: Kabul Edildi
Tarih: 24 Kasım 2026

## Bağlam
İstanbul Mobilite Veri Platformu'na iki farklı tüketici ihtiyacı gelmektedir:
1. Aylık hat raporu (ay kapanışından sonra 1 gün gecikme toleranslı).
2. Durak yoğunluğu alarmı (en geç 5 dakikalık güncellik hedefi).
Veri kaynakları olarak aylık toplu ulaşım ve trafik CSV dosyaları, API'ler ve anlık kart geçiş olayları (Kafka) bulunmaktadır. Platformda DuckDB, Spark ve Structured Streaming motorları Polaris REST kataloğu üzerinden aynı verilere erişecektir.

## Karar
Aylık dosyalar batch olarak bronze, silver ve gold katmanlarından geçer. Geçiş olayları Kafka ve Structured Streaming ile aynı kataloğa yazılır. Tablo formatı Apache Iceberg, katalog ise Polaris REST katalog olarak seçilmiştir.

## Gerekçe
- Aylık hat raporu 1 gün gecikmeyi kabul ettiği için düşük maliyetli batch işleme yeterlidir.
- Durak yoğunluğu alarmı en geç 5 dakikada güncellenmelidir; bu nedenle akış işleme (streaming) zorunludur.
- DuckDB ve Spark, Iceberg açık tablo formatı ve Polaris REST kataloğu sayesinde veriyi kopyalamadan aynı tablolar üzerinde okuma ve yazma yapabilir.
- **Elenen Seçenek 1:** Tüm veriyi Kafka ve akış motoru üzerinden işlemek. Akış işi gün boyu çalışarak sürekli kaynak tüketir ve maliyeti artırır; aylık rapor için bu tazelik gereksizdir.
- **Elenen Seçenek 2:** Verileri sadece düz Parquet dosyalarında tutmak. Düz dosyalar yarıda kalan yazmalarda tutarsızlık yaratır, eşzamanlı yazmayı desteklemez ve zaman yolculuğu (snapshot) olanağı sunmaz.

## Sonuçlar
- Batch ve streaming hatlarının birlikte yönetilmesi operasyonel karmaşıklık getirir.
- Akış işinin gün boyu çalışması maliyet takibini ve izlemeyi zorunlu kılar.
- Polaris kataloğunun sürekliliği tüm işleme motorları için kritik bağımlılık haline gelir.