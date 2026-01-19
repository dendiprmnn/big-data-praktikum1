# **LAPORAN PRAKTIKUM – TOOLS BIG DATA (HDFS, MongoDB, Cassandra)**

---

## **BAGIAN 1 – PRAKTIKUM HDFS**

### **Langkah-langkah**

1. **Menjalankan Hadoop menggunakan Docker:**

   ```bash
   docker run -it --name hadoop --hostname hadoop-master -p 9870:9870 -p 8088:8088 bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8 bash
   ```

2. **Format NameNode:**

   ```bash
   hdfs namenode -format
   ```

3. **Menjalankan NameNode dan DataNode:**

   ```bash
   hdfs --daemon start namenode
   hdfs --daemon start datanode
   ```

4. **Mengecek status service:**

   ```bash
   jps
   ```

   Output:

   ```
   NameNode
   DataNode
   Jps
   ```

5. **Membuat direktori dan upload file:**

   ```bash
   hdfs dfs -mkdir /praktikum
   hdfs dfs -put /etc/hosts /praktikum/
   hdfs dfs -ls /praktikum/
   hdfs dfs -cat /praktikum/hosts
   ```

### **Latihan Tambahan (Upload File Besar dan Cek Blok)**

1. Membuat file besar (>100 MB):

   ```bash
   dd if=/dev/zero of=bigfile.txt bs=1M count=120
   ```

2. Upload ke HDFS:

   ```bash
   hdfs dfs -put bigfile.txt /praktikum/
   ```

3. Cek pemecahan blok di HDFS:

   ```bash
   hdfs fsck /praktikum/bigfile.txt -files -blocks -locations
   ```

   Hasil menunjukkan bahwa file `bigfile.txt` otomatis dipecah menjadi beberapa blok berukuran ±64MB, sesuai dengan mekanisme penyimpanan HDFS.

<img width="1920" height="1029" alt="Screenshot (222)" src="https://github.com/user-attachments/assets/0078554c-38bd-45da-929c-73925214c4d6" />
<img width="1920" height="190" alt="Screenshot (223)" src="https://github.com/user-attachments/assets/cbd1dd37-1900-4585-81e9-fa6edff51298" />

---

## **BAGIAN 2 – PRAKTIKUM MONGODB**

### **Langkah-langkah**

1. **Menjalankan MongoDB menggunakan Docker:**

   ```bash
   docker run -d -p 27017:27017 --name mongo mongo:latest
   ```

2. **Masuk ke shell MongoDB:**

   ```bash
   docker exec -it mongo mongosh
   ```

3. **Eksperimen Dasar:**

   ```javascript
   use praktikum
   db.mahasiswa.insertOne({ nim: "12345", nama: "Andi", jurusan: "Informatika" })
   db.mahasiswa.find()
   db.mahasiswa.find({ jurusan: "Informatika" })
   ```

4. **Eksperimen Lanjutan:**

   ```javascript
   db.mahasiswa.insertMany([
     { nim: "12346", nama: "Budi", jurusan: "Sistem Informasi" },
     { nim: "12347", nama: "Citra", jurusan: "Teknik Komputer" }
   ])

   db.mahasiswa.createIndex({ nim: 1 })
   db.mahasiswa.find().sort({ nama: 1 })
   ```

5. **Latihan (Nested JSON):**

   ```javascript
   db.mahasiswa.insertOne({
     nim: "312310657",
     nama: "Intan Virginia Aulia Putri",
     jurusan: "Teknik Informatika",
     biodata: {
       alamat: "Jl. Sukarno Hatta",
       kontak: { email: "virginia123@example.com", hp: "08123456789" }
     }
   })
   db.mahasiswa.find({ nim: "312310657" }).pretty()
   ```

### **Hasil**

<img width="1920" height="637" alt="Screenshot (228)" src="https://github.com/user-attachments/assets/e44ce32c-c80e-48bd-a0db-a032834986fb" />

Semua data berhasil dimasukkan dan ditampilkan dalam format JSON. MongoDB mendukung struktur **nested document**, sehingga cocok untuk data tidak terstruktur.

---

## **BAGIAN 3 – PRAKTIKUM CASSANDRA**

### **Langkah-langkah**

1. **Menjalankan Cassandra menggunakan Docker:**

   ```bash
   docker run --name cassandra -d -p 9042:9042 cassandra:latest
   ```

2. **Masuk ke CQL Shell:**

   ```bash
   docker exec -it cassandra cqlsh
   ```

3. **Eksperimen Dasar:**

   ```sql
   CREATE KEYSPACE praktikum
   WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

   USE praktikum;

   CREATE TABLE mahasiswa (
       nim text PRIMARY KEY,
       nama text,
       jurusan text
   );

   INSERT INTO mahasiswa (nim, nama, jurusan) VALUES ('12345', 'Budi', 'Informatika');
   SELECT * FROM mahasiswa;
   ```

4. **Eksperimen Lanjutan:**

   ```sql
   INSERT INTO mahasiswa (nim, nama, jurusan) VALUES ('12346', 'Citra', 'Sistem Informasi');
   INSERT INTO mahasiswa (nim, nama, jurusan) VALUES ('12347', 'Dewi', 'Teknik Komputer');

   SELECT * FROM mahasiswa WHERE jurusan='Informatika' ALLOW FILTERING;
   ```

5. **Latihan Replikasi:**

   ```sql
   ALTER KEYSPACE praktikum
   WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
   ```

<img width="1920" height="814" alt="Screenshot (230)" src="https://github.com/user-attachments/assets/928d571d-6c36-43bc-9131-096a32f17369" />

---

## **Analisis Perbandingan Tools**

| Aspek      | HDFS                                              | MongoDB                          | Cassandra                                                            |
| ---------- | ------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------- |
| Jenis Data | Terstruktur / File besar                          | Semi & tidak terstruktur (JSON)  | Terdistribusi, terstruktur (NoSQL tabel)                             |
| Arsitektur | File system berbasis blok                         | Document-oriented                | Wide-column store                                                    |
| Kelebihan  | Dapat menyimpan data sangat besar, fault-tolerant | Skema fleksibel, mudah digunakan | Skalabilitas tinggi, cepat untuk query besar                         |
| Kekurangan | Tidak cocok untuk data kecil                      | Lemah di query kompleks          | Tidak mendukung join                                                 |
| Use Case   | Penyimpanan data besar (log, dataset ML)          | Aplikasi web, analitik, IoT      | Sistem dengan data besar yang tersebar (telekomunikasi, sensor data) |
