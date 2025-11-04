# praktikum 1
### 🧑‍💻 Disusun oleh:

**Fakhri Afif Muhaimin (312310632)**

Mata Kuliah: Big Data

Dosen Pengampu: Agung Nugroho, S.Kom., M.Kom

Universitas: [pelita bangsa]
---

````markdown
# Laporan Praktikum 1 - Big Data

**Nama:** Fakhri Afif Muhaimin  
**NIM:** 312310632  
**Mata Kuliah:** Big Data  
**Dosen Pengampu:** Agung Nugroho, S.Kom., M.Kom  

---

## 🧩 Bagian 1: Praktikum HDFS

### Latihan  
Coba upload file besar (>100MB) dan periksa apakah file tersebut terpecah menjadi blok-blok kecil di HDFS.

### Langkah-Langkah  

1. **Membuat file dummy 150MB**
   ```bash
   dd if=/dev/urandom of=largefile.dat bs=1M count=150
````

Command ini menghasilkan file berisi data random sebesar **150 MB** untuk menguji mekanisme penyimpanan HDFS.

2. **Upload file ke HDFS**

   ```bash
   hdfs dfs -mkdir /praktikum
   hdfs dfs -put largefile.dat /praktikum/
   ```

   Proses upload berjalan lancar dengan kecepatan sekitar **1.3 GB/s** karena dilakukan pada environment **single node lokal**.

3. **Pemeriksaan blok di HDFS**

   * File 150MB tersebut **terpecah menjadi 2 blok (128MB + 22MB)**.
   * Hal ini terjadi karena **default block size HDFS adalah 128MB**.
   * File yang lebih besar dari ukuran tersebut akan otomatis dipecah agar dapat:

     * Didistribusikan ke multiple DataNode untuk **parallel processing**.
     * Menyediakan **fault tolerance** jika salah satu node gagal.

---

## 🍃 Bagian 2: Praktikum MongoDB

### Latihan

Coba simpan data dalam bentuk **nested JSON** (misalnya biodata dengan alamat dan kontak).

### Langkah-Langkah

1. **Menjalankan MongoDB dengan Docker**

   ```bash
   docker run -d -p 27017:27017 --name mongodb mongo
   ```

2. **Menyimpan data nested JSON**

   ```javascript
   db.mahasiswa.insertOne({
     nama: "Fakhri Afif Muhaimin",
     nim: "312310632",
     alamat: {
       jalan: "Jl. Mawar No. 12",
       kota: "Bandung",
       kode_pos: "40123"
     },
     kontak: {
       email: "fakhri@example.com",
       telepon: "081234567890"
     }
   })
   ```

   Output:

   ```
   { acknowledged: true, insertedId: ObjectId("...") }
   ```

   Menandakan proses insert berhasil, dan MongoDB membuat **ObjectId unik** secara otomatis.

3. **Menampilkan data**

   ```javascript
   db.mahasiswa.find().pretty()
   ```

   Perintah `.pretty()` memberikan tampilan terstruktur dan mudah dibaca dengan indentasi jelas, sehingga **nested object** dapat terlihat dengan rapi.

---

## 🪶 Bagian 3: Praktikum Cassandra

### Latihan

Jalankan Cassandra dalam **2-node cluster** menggunakan Docker Compose dan amati distribusi data.

### Langkah-Langkah

1. **Menjalankan Cassandra cluster**

   * Pastikan Docker aktif, lalu buat file `docker-compose.yml` berikut:

     ```yaml
     version: '3'
     services:
       cassandra-node1:
         image: cassandra
         container_name: cassandra-node1
         environment:
           - CASSANDRA_CLUSTER_NAME=MyCluster
           - CASSANDRA_LISTEN_ADDRESS=cassandra-node1
           - CASSANDRA_SEEDS=cassandra-node1
         networks:
           - cassandra-net

       cassandra-node2:
         image: cassandra
         container_name: cassandra-node2
         environment:
           - CASSANDRA_CLUSTER_NAME=MyCluster
           - CASSANDRA_LISTEN_ADDRESS=cassandra-node2
           - CASSANDRA_SEEDS=cassandra-node1
         depends_on:
           - cassandra-node1
         networks:
           - cassandra-net

     networks:
       cassandra-net:
         driver: bridge
     ```

   Jalankan cluster:

   ```bash
   docker-compose up -d
   ```

   Jika berhasil, log akan menampilkan pesan:

   ```
   Created default superuser role
   ```

2. **Masuk ke Node 1**

   ```bash
   docker exec -it cassandra-node1 cqlsh
   ```

3. **Membuat keyspace dan tabel**

   ```sql
   CREATE KEYSPACE latihan WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 2};

   USE latihan;

   CREATE TABLE users (
       id UUID PRIMARY KEY,
       nama text,
       umur int,
       kota text
   );
   ```

4. **Insert data dummy**

   ```sql
   INSERT INTO users (id, nama, umur, kota)
   VALUES (uuid(), 'Fakhri Afif Muhaimin', 22, 'Bandung');
   ```

5. **Verifikasi data**

   ```sql
   SELECT * FROM users;
   ```

6. **Periksa status cluster**

   ```bash
   nodetool status
   ```

   Hasil menunjukkan kedua node berstatus **UN (Up/Normal)**, menandakan cluster berjalan stabil dengan distribusi data merata.

---

## 📚 Kesimpulan

* **HDFS** memecah file besar menjadi blok-blok kecil untuk mendukung distribusi dan fault tolerance.
* **MongoDB** menyimpan data dalam format JSON fleksibel dan mendukung struktur nested.
* **Cassandra** mendukung arsitektur **multi-node cluster** yang memungkinkan skalabilitas horizontal dan replikasi data otomatis.


```
