####
#### 1. Veri Kaynakları (Kafka Topic’leri)
Sistemimize 3 farklı Kafka topic’inden veri geliyor:

- Hiz-topic:  Hız sensöründen veri — toplam 120 kayıt

- Isik-topic: Işık sensöründen veri — toplam 120 kayıt

- Nem-topic:  Nem sensöründen veri — toplam 120 kayıt

Bu veriler Elasticsearch’e aktarılıyor ve her biri ayrı bir index olarak tutuluyor:
- Hız Indexi
- Işık Indexi
- Nem Indexi


#### 2. Elasticsearch Index Yapısı
Her index için:
- 3 primary shard tanımlanıyor (veri eşit bölünecek)
- 2 replica shard tanımlanıyor (her primary shard’ın 2 kopyası olacak)



#### 3.Shard Yapısı

- Hız Indexi:
```
Primary Shard:
40A, 40B, 40C

Replica Shard:
40A1, 40A2
40B1, 40B2
40C1, 40C2
```

- Işık Indexi:
```
Primary Shard:
40K, 40L, 40M

Replica Shard:
40K1, 40K2
40L1, 40L2
40M1, 40M2
```

- Nem Indexi:
```
Primary Shard:
40X, 40Y, 40Z

Replica Shard:
40X1, 40X2
40Y1, 40Y2
40Z1, 40Z2
```

Toplam shard sayısı:
Her index için 9 shard → 3 index = 27 shard















































































