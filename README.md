####

1. Cluster
- Bir Elasticsearch kümesidir.
- Birden fazla node içerir.
- Her cluster’ın bir ismi vardır (örn: es-logs-cluster).
- Tek bir entry point gibi çalışır. Kullanıcı için bir Elasticsearch cluster, tek bir sistem gibi görünür.

2. Node
- Elasticsearch'ün çalıştığı her bir sunucuya (veya instance’a) node denir.
- Her node, veriyi tutabilir, arama yapabilir, shard barındırabilir.
- Node tipleri olabilir: master node, data node, coordinating node, vs.

3. Index
- Elasticsearch'teki en temel mantıksal veri yapısıdır.
- SQL'deki tablo gibidir ama NoSQL mantığıyla işler.
- Örnek: logs-2025-05-03 adında bir index, bugünün log verilerini barındırabilir.
- Index, veriyi shard’lara böler.

4. Shard
- Her index, primary ve replica shard’lara bölünür.
- Her shard aslında küçük bir Lucene instance’dır.
- Shard’lar sayesinde Elasticsearch ölçeklenebilir ve yüksek performanslı olur.
- Örnek: Bir index 5 primary shard ve her biri için 1 replica ile oluşturulursa toplamda 10 shard olur.

5. Document
- Elasticsearch'e gönderilen her veri bir document’tir.
- JSON formatındadır.
- Her document’ın bir _id si vardır.
Örnek:
```
{
  "timestamp": "2025-05-03T11:45:00Z",
  "message": "User login failed",
  "user_id": "abc123"

}
```
#### 6. Mapping (Şema)
- Document’lerdeki alanların veri tiplerinin tanımıdır.
- SQL’deki şemaya benzer.
- Otomatik yapılabilir (dynamic mapping) veya manuel olarak tanımlanabilir.

#### 7. Analyzer / Tokenizer
- Full-text arama için veriyi işlerken kullanılan metin analiz araçlarıdır.
- Kelimeleri küçük harfe çevirme, köklerine ayırma (stemming), durdurma kelimeleri çıkarma gibi görevleri yapar.


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### B.Örnek ile Açıklama

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

- Hız Indexi: 120 Veri-mesaj
```
Primary Shard:
40A, 40B, 40C

Replica Shard:
40A1, 40A2
40B1, 40B2
40C1, 40C2
```

- Işık Indexi: 120 Veri-mesaj
```
Primary Shard:
40K, 40L, 40M

Replica Shard:
40K1, 40K2
40L1, 40L2
40M1, 40M2
```

- Nem Indexi: 120 Veri-mesaj
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


#### 4. Node’lara Dağıtım
- Aynı shard’tan (örneğin 40A, 40A1, 40A2) aynı node’a koymamak
- Her node’a eşit yük dağıtmak
- 
- 3 Node ile Dağılım 
- 3 Primary
- 2 Replica
- 120/4=30
```
| Shard    | Node1 | Node2 | Node3 |
| -------- | ----- | ----- | ----- |
| **Hız**  | 40A   | 40B   | 40C   |
|          | 40B1  | 40C1  | 40A1  |
|          | 40C2  | 40A2  | 40B2  |

| **Işık** | 40K   | 40L   | 40M   |
|          | 40L1  | 40M1  | 40K1  |
|          | 40M2  | 40K2  | 40L2  |
          
| **Nem**  | 40X   | 40Y   | 40Z   |
|          | 40Y1  | 40Z1  | 40X1  |
|          | 40Z2  | 40X2  | 40Y2  |

```
- 6 Node ile Dağılım 
- 4 Primary
- 2 Replica
- 120/4=30 
```
| **Shard**       | **Node1** | **Node2** | **Node3** | **Node4** | **Node5** | **Node6** |
| --------------- | --------- | --------- | --------- | --------- | --------- | --------- |
| **Hız Indexi**  | 30A       | 30B       | 30C       | 30D       | 30A1      | 30A2      |
|                 | 30B1      | 30B2      | 30C1      | 30C2      | 30D1      | 30D2      |

| **Işık Indexi** | 30K       | 30L       | 30M       | 30N       | 30K1      | 30K2      |
|                 | 30L1      | 30L2      | 30M1      | 30M2      | 30N1      | 30N2      |

| **Nem Indexi**  | 30X       | 30Y       | 30Z       | 30W       | 30X1      | 30X2      |
|                 | 30Y1      | 30Y2      | 30Z1      | 30Z2      | 30W1      | 30W2      |
```

- 6 Node ile Dağılım 
- 6 Primary
- 2 Replica
- 120/6=20
```
| **Shard**     | **Node1** | **Node2** | **Node3** | **Node4** | **Node5** | **Node6** |
| ------------- | --------- | --------- | --------- | --------- | --------- | --------- |
| **Hız (P)**   | 20A       | 20B       | 20C       | 20D       | 20E       | 20F       |
|               | 20A1      | 20B1      | 20C1      | 20D1      | 20E1      | 20F1      |
|               | 20A2      | 20B2      | 20C2      | 20D2      | 20E2      | 20F2      |

| **Işık (P)**  | 20K       | 20L       | 20M       | 20N       | 20O       | 20P       |
|               | 20K1      | 20L1      | 20M1      | 20N1      | 20O1      | 20P1      |
|               | 20K2      | 20L2      | 20M2      | 20N2      | 20O2      | 20P2      |

| **Nem (P)**   | 20X       | 20Y       | 20Z       | 20W       | 20V       | 20U       |
|               | 20X1      | 20Y1      | 20Z1      | 20W1      | 20V1      | 20U1      |
|               | 20X2      | 20Y2      | 20Z2      | 20W2      | 20V2      | 20U2      |
```

#### 5. Okuma ve Yazma İşlemleri
- Yazma: Her veri sadece primary shard'a yazılır, ardından replica shard’lara otomatik kopyalanır.
- Okuma: Hem primary, hem de replica shard’lar kullanılabilir.









































































