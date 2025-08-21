# 📊 User Behavior Analysis - Data Analyst Technical Test

## 📌 Deskripsi Proyek
Proyek ini merupakan technical test untuk posisi **Data Analyst**, yang bertujuan untuk menganalisis perilaku pengguna (user behavior) menggunakan dataset publik. Analisis dilakukan dengan pembersihan data (data cleaning) menggunakan SQL, pembuatan query analisis, serta visualisasi hasil melalui **Looker Studio**.

Deliverables proyek ini meliputi:
- **SQL Queries** untuk data cleaning dan analisis.
- **README file** sebagai dokumentasi cara menjalankan proyek.
- **Presentation file (PDF)** berisi insight hasil analisis.
- **Dashboard (Looker Studio)** untuk visualisasi data.

---

## 📂 Sumber Data
Dataset publik terkait user behavior.  
Contoh: *Users Data, Cards Data, Transactions Data*.
link: https://drive.google.com/drive/folders/190CuIAO12_uu_9KgfIQl6QiGU00qj8h8

---

## 🛠 Tools yang Digunakan
- **MySQL Workbench – Untuk query SQL dan data cleaning.
- **Google Looker Studio** – Untuk membuat dashboard.
- **CSV** – Sebagai sumber data awal.

---

## ▶️ Cara Menjalankan Proyek

### 1. **Persiapan Data**
- Simpan dataset dalam folder `Uploads` MySQL atau path sesuai konfigurasi kamu.
- Pastikan dataset memiliki struktur:
  - `users_data.csv`
  - `cards_data.csv`
  - `transactions_data.csv`

### 2. **Import Data ke MySQL**
- users_data.csv dan cards_data.csv langsung import dengan klik "table data import wizard" dan browse sesuai letak data disimpan
- transaction_data.csv dikarenakan data yang besar sehingga diperlukan cara khusus yaitu dengan query sebagai berikut:
    #MEMBUAT TABEL TRANSACTIONS_DATA DAN MEMASUKKAN DATA 
CREATE TABLE transactions_data (
    id INT,
    date VARCHAR(100),
    client_id VARCHAR(100),
    card_id VARCHAR(100),
    amount VARCHAR(100),
    use_chip VARCHAR(100),
    merchant_id INT,
    merchant_city VARCHAR(100),
    merchant_state VARCHAR(100),
    mcc INT,
    error VARCHAR(100)
);
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/transactions_data1.csv'
INTO TABLE transactions_data
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, date, client_id, card_id, amount, use_chip, merchant_id, merchant_city, merchant_state, mcc, error);

### 2. **Data Cleaning**
- **-- CARD DATA --**
#MENGUBAH CREDIT_LIMIT MENJADI INTEGER
UPDATE cards_data
SET credit_limit = REPLACE(credit_limit, '$', '');
ALTER TABLE cards_data MODIFY credit_limit INT;
ALTER TABLE cards_data 
RENAME COLUMN id TO card_id;

#MENGUBAH EXPIRES MENJADI DATE
ALTER TABLE cards_data ADD COLUMN expires_date DATE;
UPDATE cards_data
SET expires_date = STR_TO_DATE(CONCAT('01/', expires), '%d/%m/%Y');

#MENGUBAH ACCT_OPEN_DATE MENJADI DATE
ALTER TABLE cards_data ADD COLUMN acct_open_date1 DATE;
UPDATE cards_data
SET acct_open_date1 = STR_TO_DATE(CONCAT('01/', acct_open_date), '%d/%m/%Y');

- **-- USERS DATA --**
#MENGUBAH per_capita_income MENJADI INTEGER
UPDATE users_data
SET per_capita_income = REPLACE(per_capita_income, '$', '');
ALTER TABLE users_data MODIFY per_capita_income INT;

#MENGUBAH yearly_income MENJADI INTEGER
UPDATE users_data
SET yearly_income = REPLACE(yearly_income, '$', '');
ALTER TABLE users_data MODIFY yearly_income INT;

#MENGUBAH total_debt MENJADI INTEGER
UPDATE users_data
SET total_debt = REPLACE(total_debt, '$', '');
ALTER TABLE users_data MODIFY total_debt INT;

- **-- TRANSACTION DATA --**
  #MENGUBAH CLIENT_ID DAN CARD_ID MENJADI INTEGER
ALTER TABLE transactions_data MODIFY client_id INT;
ALTER TABLE transactions_data MODIFY card_id INT;

  #Memberikan Index, Primary Key, dan Foreign Key
ALTER TABLE users_data ADD INDEX (id);
ALTER TABLE cards_data ADD INDEX (card_id);
ALTER TABLE transactions_data ADD INDEX (client_id);
ALTER TABLE transactions_data ADD INDEX (amount);
ALTER TABLE transactions_data ADD INDEX (date);
ALTER TABLE transactions_data ADD INDEX (card_id);

#PRIMARY KEY DAN FOREIGN KEY
ALTER TABLE users_data
ADD PRIMARY KEY (id);
ALTER TABLE cards_data
ADD PRIMARY KEY (card_id);
ALTER TABLE cards_data
ADD CONSTRAINT fk_card_data
FOREIGN KEY (client_id) REFERENCES users_data(id);
ALTER TABLE transactions_data
ADD CONSTRAINT fk_transactions_data
FOREIGN KEY (client_id) REFERENCES users_data(id);
ALTER TABLE transactions_data
ADD CONSTRAINT fk_transactions_data2
FOREIGN KEY (card_id) REFERENCES cards_data(card_id);

#MENGUBAH AMOUNT MENJADI INTEGER
UPDATE transactions_data
SET amount = REPLACE(amount, '$', '');
ALTER TABLE transactions_data MODIFY amount INT;
#MENGUBAH date MENJADI DATETIME
ALTER TABLE transactions_data MODIFY date datetime;
      **--        DATA SUDAH CLEAN        --**

### 3. **Melakukan Analisa Data**
#ANALISIS 1 (Analisa Gender Yang Memiliki Credit Score Terbanyak)
SELECT
    CASE 
        WHEN gender = 'female' Then 'female'
        WHEN gender = 'male' Then 'male'
    END AS gender,
    SUM(CASE WHEN `credit_score` < 661 THEN 1 ELSE 0 END) AS jumlah_low_credit_score22
FROM users_data
GROUP BY gender;

#ANALISIS 2 (Peluang Data Salah Karena Nasabah Tanpa Hutang dan Pendapatan Per kapita < Total Transaksi)
SELECT users_data.id, total_debt AS "jumlah Hutang ($)", YEAR(transactions_data.date) AS tahun, COUNT(amount) AS banyak_transaksi, sum(amount) AS "jumlah_nominal_transaksi ($)", per_capita_income AS "per_capita_income ($)"
FROM users_data
JOIN transactions_data
ON users_data.id = transactions_data.client_id
GROUP BY users_data.id, total_debt, per_capita_income, YEAR(transactions_data.date)
HAVING total_debt=0 AND sum(amount) > per_capita_income
ORDER BY per_capita_income, tahun;

#ANALISIS 3 (Segmentasi Nasabah Berdasarkan Usia Untuk Menentukan Strategi Pemasaran)
SELECT
    CASE 
        WHEN current_age BETWEEN 0 AND 20 THEN '0-20'
        WHEN current_age BETWEEN 21 AND 40 THEN '21-40'
        WHEN current_age BETWEEN 41 AND 60 THEN '41-60'
        ELSE '61+' 
    END AS age_range,
    SUM(CASE WHEN `credit_score` < 661 THEN 1 ELSE 0 END) AS jumlah_low_credit_score,
    COUNT(*) AS jumlah_nasabah,
    ROUND(
        SUM(CASE WHEN `credit_score` < 661 THEN 1 ELSE 0 END) / COUNT(*) * 100, 2) AS persentase_low_credit_score,
    AVG(credit_score) AS rerata_credit_score,
    AVG(per_capita_income) AS rerata_per_capita
FROM users_data
GROUP BY age_range
ORDER BY persentase_low_credit_score DESC;

#ANALISIS 4 (Pengaruh Perbandingan Antara Pendapatan per kapita dan Hutang Terhadap Credit Score)
SELECT 
    id AS client_id,
    per_capita_income AS "per_capita_income ($)",
    total_debt AS "Total_Hutang ($)",
    ROUND(total_debt / NULLIF(per_capita_income,0) *100, 2) AS Persentase,
    credit_score,
    CASE 
        WHEN (total_debt / NULLIF(per_capita_income,0)) < 0.35 THEN 'Sehat'
        WHEN (total_debt / NULLIF(per_capita_income,0)) BETWEEN 0.35 AND 0.6 THEN 'Cukup'
        ELSE 'Berisiko'
    END AS kategori
FROM users_data;
-- Apakah Kategori Sehat Memiliki Credit Score Lebih Baik Daripada Kategori Lainnya --
SELECT 
    CASE 
        WHEN (total_debt / NULLIF(per_capita_income,0)) < 0.35 THEN 'Sehat'
        WHEN (total_debt / NULLIF(per_capita_income,0)) BETWEEN 0.35 AND 0.6 THEN 'Cukup'
        ELSE 'Berisiko'
    END AS kategori_dti,
    COUNT(*) AS jumlah_nasabah,
    AVG(credit_score) AS rata_credit_score
FROM users_data
GROUP BY kategori_dti
ORDER BY rata_credit_score DESC;
-- Kategori Sehat Memiliki Credit Score Lebih Baik Dibanding Kategori Lainnya--

#ANALISIS 5 (Analisis Nasabah High Risk)
SELECT *
FROM 
(SELECT transactions_data.client_id, SUM(amount) AS jumlah_nominal_transaksi, COUNT(*) AS banyak_transaksi,
 per_capita_income AS "per_capita_income ($)", credit_score, card_type,
(SELECT SUM(cards_data.credit_limit) 
     FROM cards_data
     WHERE cards_data.client_id = transactions_data.client_id
       AND cards_data.card_type = 'Credit') AS total_credit_limit
FROM transactions_data
JOIN users_data
ON transactions_data.client_id = users_data.id
JOIN cards_data
ON transactions_data.card_id = cards_data.card_id
WHERE card_type = 'Credit'
GROUP BY transactions_data.client_id, per_capita_income, card_type
HAVING  total_credit_limit < jumlah_nominal_transaksi
ORDER BY credit_score ASC) 
AS nasabah_dicurigai
WHERE credit_score < 661;

#ANALISIS 6 (Mengetahui Total Transaksi Per Bulan)
SELECT 
    DATE_FORMAT(date, '%Y-%m') AS tahun_bulan,
    SUM(amount) AS total_amount
FROM transactions_data
GROUP BY tahun_bulan
ORDER BY tahun_bulan;
-- SELESAI --

### 4. **Melakukan visualisasi Data**
-Menggunakan tools Looker studio
### 5. **Membuat Presentasi Data**
-Menggunakan Canva atau Power Point
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
