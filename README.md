# **PROJECT MODUL 2**

# **1. GENERAL INFORMATION**

## **1.1 Context** 
Northwind merupakan perusahaan yang bergerak pada bidang impor-ekspor makanan dan minuman berlokasi di Amerika dan tidak memproduksi barnagnya sendiri, Perusahaan Northwind menjual Kembali produk dari supplier ke konsumen. Produk yang dijual perushaan Northwind berupa makanan dan minuman dengan 77 jenis produk yang berbeda, yang di kelompokan atau dikategorikan menjadi 8 kategori. Perushaan ini mengimpor dari 29 perusahaan dari berbagai negara. Dan memiliki total pelanggan sebanyak 91 perusahan di berbagai negara. Dari database tersebut saya akan berfokus kepada supplier dan mengambil insight-insight yang berhubungan dengan perusahan-perusahaan supplier.

## **1.2 Database Information**

sumber database : https://drive.google.com/drive/folders/1fTHrwh_gcLsOFKXHnUzUGEu_APxLoD9i?usp=sharing

Database Menginformasikan data tentang detail pembelian atau order, data karyawan yang menanganai pembelian tersebut, data customers dan data supplier pada perusahaan Northwind Tahun 1996 sampai 1998

Database memiliki 11 tabel yang memiliki data dan informasi :

- Region : menyimpan informasi region ID dan deskripsi Region.
- Territories : Menyimpan informasi Teritori beserta classifikasi berdasarkan Region ID.
- EmployeeTerritories : Menyimpan informasi territori ID dari setiap nomer ID karyawan.
- Employees : Menyimpan semua informasi karyawan, seperti nama, jabatan, tanggal lahir, tanggal perekrutan, dan alamat.
- Orders : Menyimpan informasi pembelian atau order seperti tanggal order, tanggal pengiriman, lokasi pengiriman, dan nomer ID perushaan pengiriman. dan terdapat informasi informasi siapa karyawan (sales) yang bertanggung jawab atas order tersebut
- Shippers : Menyimpan informasi no ID dan nama Perusahaan jasa pengiriman serta nomer telepon perusahaan
- Costumers: Menyimpan semua informasi perusahaan pelanggan serta lokasi dan nomer telepon perusahaan
- OrderDetails : Menyimpan informasi produk ID beserta jumlah dan harga satuan produk yang telah order
- Products : Menyimpan semua informasi produk seperti kuantity per unit, harga, beserta stock dan jumlah produk yang sedang dalam proses order atau pembelian. Dan informasi produk yang sudah tidak di stock
- Categories : Menyimpan informasi kategori produk dan deskripsi dari kategori produk
- Suppliers : Menyimpan semua informasi perusahaan supplier yang bekerjasama, Beserta lokasi dan nomer telepon perusahaan 

Database memiliki 2 tabel yang kosong :

- CustomerCustomerDemo
- CustomerDemographics

Setiap tabel yang tertera pada database dapat terhubung, baik secara langsung maupun tidak langsung, sehingga setiap informasi dari database ini akan dapat saling berkaitan.

## **1.3 Table Relationship**

![Northwind ERD.png](attachment:15678301-1621-438b-a403-838d45226e52.png)

## **1.4 DATABASE**

## **1.4.Library**


```python
# Connect SQL to Python
import mysql.connector as sqlcon

# Data Manipulation dan General Data
import pandas as pd
import numpy as np

# Visualisasi 
import plotly.express as px
import matplotlib.pyplot as plt
import seaborn as sns

# Statistic
import scipy.stats as stats
from scipy.stats import shapiro
from scipy.stats import kruskal

```

# **2. SQL**

## **2.1 Menghubungkan ke Database**
Proses ini diperlukan untuk menghubungkan database pada server sql ke jupyterlab, sehingga kita dapat mengexplore data melalui function pyhton


```python
# Connect To Database

mydb = sqlcon.connect(
    host = 'localhost',
    user = 'root',
    passwd = '12345',
    database = 'Northwind'
)
```

## **2.2 Membuat Function Quary**
Proses ini berguna agar kita dapat memanggil tabel dan data Northwind dengan menuliskan ``quary`` sql pada function, sehingga kita dapat membuat Dataframe dari database Northwind. Sehingga memudahkan dalam proses analisa data. nantinya kita tidak hanya akan menggunakan 1 tabel saja melainkan kita akan menggunakan hubungan antar tabelnya.


```python
# Quary Function

mycursor = mydb.cursor()
def sql_df(yourQuary) :
    mycursor.execute(yourQuary)
    myresult = mycursor.fetchall()
    df = pd.DataFrame(myresult, columns = mycursor.column_names)
    return df
```

### **2.2.1 Data Order, Detail Supplier & Total Penjualan**

**Pada Analasis ini saya akan melakukan analisa yang berfokus kepada ```Supplier```**

Data pertama ini merupakan data utama yang nantinya akan dianalisa lebih lanjut. Data ini merupakan gabungan dari 4 tabel, yaitu tabel ```Orders```, ```OrderDetails```, ```Product```, dan ```Suppliers```. Masing-masing dari setiap tabel tersebut diambil beberapa kolomnya dan tidak diambil secara keseluruhan. Informasi-informasi yang dianggap penting saja lah yang diambil. Informasi yang diambil antara lain adalah :

- OrderID dari tabel Orders
- OrderDate dari tabel Orders
- RequiredDate dari tabel Orders
- ShippedDate dari tabel Orders
- SupplierID dari tabel Suppliers
- Supplier_CompanyName adalah CompanyName dari tabel Suppliers
- ContactName dari tabel Suppliers
- ContactTitle dari tabel Suppliers
- Address dari tabel Supplier
- City dari tabel Supplier
- Region dari tabel Supplier
- Country dari tabel Supplier
- ProductID dari tabel Products
- ProductName dari tabel Products
- Harga_Jual adalah UnitPrice dari Tabel Products yang di **definisikan sebagai harga jual perusahaan kepada customer**
- Harga_Beli adalah UnitPrice dari Tabel OrderDetails yang di **definisikan sebagai harga beli perusahaan dari supplier**
- Quantity dari tabel OrderDetails


Selain dari tabel, terdapat sebuah kolom juga yang dinamakan Total_Harga yang merupakan hasil perkalian dari Harga_Jual dengan Quantity.

Semua informasi tersebut kemudian dijadikan dalam sebuah DataFrame yang nantinya akan diolah informasinya.


```python
# Quary 1

tabel1 = sql_df('''
Select  O.OrderID, O.OrderDate, O.RequiredDate, O.ShippedDate,
		S.SupplierID, S.CompanyName as Supplier_CompanyName, S.ContactName, S.ContactTitle, S.Address, S.City, S.Region, S.Country, 
        P.ProductID, P.ProductName,
		OD.UnitPrice as Harga_Beli, P.UnitPrice as Harga_Jual, OD.Quantity
From Orders O
Left Join OrderDetails OD
	On O.OrderID = OD.OrderID
Left Join Products P
	On OD.ProductID = P.ProductID
Left Join Suppliers S
	On P.SupplierID = S.SupplierID
Order by supplierID, ProductID;
                ''')
tabel1.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Region</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10838</td>
      <td>1998-01-19</td>
      <td>1998-02-16</td>
      <td>1998-01-23</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10847</td>
      <td>1998-01-22</td>
      <td>1998-02-05</td>
      <td>1998-02-10</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10863</td>
      <td>1998-02-02</td>
      <td>1998-03-02</td>
      <td>1998-02-17</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10869</td>
      <td>1998-02-04</td>
      <td>1998-03-04</td>
      <td>1998-02-09</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10905</td>
      <td>1998-02-24</td>
      <td>1998-03-24</td>
      <td>1998-03-06</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
</div>



### **2.2.2 Total Keuntungan setiap Produk**

Data kedua ini merupakan data yang menginformasikan total keuntungan atau profit setiap produk dari supplier. Data ini merupakan gabungan dari 4 tabel, yaitu tabel ```Orders```, ```OrderDetails```, ```Product```, dan ```Suppliers```. Masing-masing dari setiap tabel tersebut diambil beberapa kolomnya dan tidak diambil secara keseluruhan. Informasi-informasi yang dianggap penting saja lah yang diambil. Informasi yang diambil antara lain adalah :


- Supplier_CompanyName adalah CompanyName dari tabel Suppliers
- ProductName dari tabel Products
- Harga_Jual adalah UnitPrice dari Tabel Products yang di **definisikan sebagai harga jual perusahaan kepada customer**
- Harga_Beli adalah UnitPrice dari Tabel OrderDetails yang di **definisikan sebagai harga beli perusahaan dari supplier**
    - Harga beli produk tidak selalu sama, terkadang mengalami penurunan ataupun kenaikan
- Total_Quantity adalah jumlah penjual tiap produk dari tabel OrderDetails


Selain dari tabel, terdapat sebuah kolom juga yang dinamakan Profit_Each yang merupakan selisih dari Harga_Jual dengan Harga_beli. dan terdapat kolom Profit yang merukapan perkalian dari Profit_Each dengan Total_Quantity

Semua informasi tersebut kemudian dijadikan dalam sebuah DataFrame yang nantinya akan diolah informasinya.


```python
# Quary 2

tabel2 = sql_df("""
Select  S.CompanyName as Supplier_CompanyName,
		P.ProductName,
		(OD.UnitPrice) as Harga_beli, P.UnitPrice as Harga_Jual, (P.UnitPrice-(OD.UnitPrice)) as Profit_Each, 
        sum(OD.Quantity) as Total_Quantity, OD.Quantity, (P.UnitPrice * OD.Quantity) as Total_Penjualan, ((P.UnitPrice-(OD.UnitPrice)) * sum(OD.Quantity)) as Profit
From Orders O
Left Join OrderDetails OD
	On O.OrderID = OD.OrderID
Left Join Products P
	On OD.ProductID = P.ProductID
Left Join Suppliers S
	On P.SupplierID = S.SupplierID
Group by Supplier_CompanyName, ProductName, Harga_beli
Order by S.supplierID, P.ProductID;
select * from suppliers;
                """)
tabel2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Supplier_CompanyName</th>
      <th>ProductName</th>
      <th>Harga_beli</th>
      <th>Harga_Jual</th>
      <th>Profit_Each</th>
      <th>Total_Quantity</th>
      <th>Quantity</th>
      <th>Total_Penjualan</th>
      <th>Profit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Exotic Liquids</td>
      <td>Chai</td>
      <td>14.4000</td>
      <td>18.0000</td>
      <td>3.6000</td>
      <td>174</td>
      <td>45</td>
      <td>810.0000</td>
      <td>626.4000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Exotic Liquids</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>0.0000</td>
      <td>654</td>
      <td>35</td>
      <td>630.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Exotic Liquids</td>
      <td>Chang</td>
      <td>15.2000</td>
      <td>19.0000</td>
      <td>3.8000</td>
      <td>401</td>
      <td>50</td>
      <td>950.0000</td>
      <td>1523.8000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Exotic Liquids</td>
      <td>Chang</td>
      <td>19.0000</td>
      <td>19.0000</td>
      <td>0.0000</td>
      <td>656</td>
      <td>10</td>
      <td>190.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Exotic Liquids</td>
      <td>Aniseed Syrup</td>
      <td>8.0000</td>
      <td>10.0000</td>
      <td>2.0000</td>
      <td>100</td>
      <td>50</td>
      <td>500.0000</td>
      <td>200.0000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>151</th>
      <td>Gai pturage</td>
      <td>Camembert Pierrot</td>
      <td>34.0000</td>
      <td>34.0000</td>
      <td>0.0000</td>
      <td>1087</td>
      <td>70</td>
      <td>2380.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Forts d'rables</td>
      <td>Sirop d'rable</td>
      <td>28.5000</td>
      <td>28.5000</td>
      <td>0.0000</td>
      <td>472</td>
      <td>5</td>
      <td>142.5000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>153</th>
      <td>Forts d'rables</td>
      <td>Sirop d'rable</td>
      <td>22.8000</td>
      <td>28.5000</td>
      <td>5.7000</td>
      <td>131</td>
      <td>16</td>
      <td>456.0000</td>
      <td>746.7000</td>
    </tr>
    <tr>
      <th>154</th>
      <td>Forts d'rables</td>
      <td>Tarte au sucre</td>
      <td>39.4000</td>
      <td>49.3000</td>
      <td>9.9000</td>
      <td>360</td>
      <td>25</td>
      <td>1232.5000</td>
      <td>3564.0000</td>
    </tr>
    <tr>
      <th>155</th>
      <td>Forts d'rables</td>
      <td>Tarte au sucre</td>
      <td>49.3000</td>
      <td>49.3000</td>
      <td>0.0000</td>
      <td>723</td>
      <td>40</td>
      <td>1972.0000</td>
      <td>0.0000</td>
    </tr>
  </tbody>
</table>
<p>156 rows × 9 columns</p>
</div>



### **RECONNECT TO DATABASE**
Menghubungkan kembali data dengan database di lakukan untuk meminimalisir ```databaseError : 2014 (HY000): Commands out of sync; you can't run this command now```


```python
# ReConnect To Database

mydb = sqlcon.connect(
    host = 'localhost',
    user = 'root',
    passwd = '12345',
    database = 'Northwind'
)
# Quary Function

mycursor = mydb.cursor()
def sql_df(yourQuary) :
    mycursor.execute(yourQuary)
    myresult = mycursor.fetchall()
    df = pd.DataFrame(myresult, columns = mycursor.column_names)
    return df
```

### **2.2.3 Negara Customer dan Supplier**

Data ketiga ini menginformasikan bahwa terdapat Order yang memiliki lokasi negara customer dan negara supplier sama


```python
# Quary 3

tabel3 = sql_df("""
Select  O.OrderID, C.CustomerID,
        S.SupplierID, S.CompanyName as Supplier_CompanyName, S.Country as Supplier_Country, S.City as Supplier_City,
        C.CompanyName as Customer_CompanyName, C.Country as Customer_Country, C.City as Customer_City, P.UnitsInStock
From Orders O
Left Join OrderDetails OD
	On O.OrderID = OD.OrderID
Left Join Products P
	On OD.ProductID = P.ProductID
Left Join Suppliers S
	On P.SupplierID = S.SupplierID
Left Join Customers C
	On O.CustomerID = C.CustomerID
WHERE S.Country = C.Country;
                """)
tabel3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>CustomerID</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>Supplier_Country</th>
      <th>Supplier_City</th>
      <th>Customer_CompanyName</th>
      <th>Customer_Country</th>
      <th>Customer_City</th>
      <th>UnitsInStock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10752</td>
      <td>NORTS</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>UK</td>
      <td>London</td>
      <td>North/South</td>
      <td>UK</td>
      <td>London</td>
      <td>39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10869</td>
      <td>SEVES</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>UK</td>
      <td>London</td>
      <td>Seven Seas Imports</td>
      <td>UK</td>
      <td>London</td>
      <td>39</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11047</td>
      <td>EASTC</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>UK</td>
      <td>London</td>
      <td>Eastern Connection</td>
      <td>UK</td>
      <td>London</td>
      <td>39</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10435</td>
      <td>CONSH</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>UK</td>
      <td>London</td>
      <td>Consolidated Holdings</td>
      <td>UK</td>
      <td>London</td>
      <td>17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10741</td>
      <td>AROUT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>UK</td>
      <td>London</td>
      <td>Around the Horn</td>
      <td>UK</td>
      <td>London</td>
      <td>17</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>129</th>
      <td>10339</td>
      <td>MEREP</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Canada</td>
      <td>Ste-Hyacinthe</td>
      <td>Mre Paillarde</td>
      <td>Canada</td>
      <td>Montral</td>
      <td>17</td>
    </tr>
    <tr>
      <th>130</th>
      <td>10389</td>
      <td>BOTTM</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Canada</td>
      <td>Ste-Hyacinthe</td>
      <td>Bottom-Dollar Markets</td>
      <td>Canada</td>
      <td>Tsawassen</td>
      <td>17</td>
    </tr>
    <tr>
      <th>131</th>
      <td>10505</td>
      <td>MEREP</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Canada</td>
      <td>Ste-Hyacinthe</td>
      <td>Mre Paillarde</td>
      <td>Canada</td>
      <td>Montral</td>
      <td>17</td>
    </tr>
    <tr>
      <th>132</th>
      <td>10949</td>
      <td>BOTTM</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Canada</td>
      <td>Ste-Hyacinthe</td>
      <td>Bottom-Dollar Markets</td>
      <td>Canada</td>
      <td>Tsawassen</td>
      <td>17</td>
    </tr>
    <tr>
      <th>133</th>
      <td>11027</td>
      <td>BOTTM</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Canada</td>
      <td>Ste-Hyacinthe</td>
      <td>Bottom-Dollar Markets</td>
      <td>Canada</td>
      <td>Tsawassen</td>
      <td>17</td>
    </tr>
  </tbody>
</table>
<p>134 rows × 10 columns</p>
</div>



terlihat dari data di atas bahwa terdapat 134 order sekitar 6% dari total order yang memiliki kesamaan lokasi antara supplier dan costumer.

# **3. DATA MANIPULATION**

data yang digunakan untuk dianalisis adalah data pada ```tabel1```. Sebelum melakukan analisis lebih lanjut, hal yang harus dilakukan adalah mengecek informasi serta anomali pada data. Jika memang terdapat hal-hal yang dianggap 'kotor' pada data, maka yang perlu dilakukan adalah melakukan penanganan pada bagian tersebut. Pada bagian ini, data akan 'dibersihkan', sehingga output akhir yang diharapkan adalah terdapat sebuah dataset yang bersih yang dapat dianalisis lebih lanjut dengan menampilkan visualisasi, serta melihat statistics-nya.


```python
# Check Info Tabel1

tabel1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2155 entries, 0 to 2154
    Data columns (total 17 columns):
     #   Column                Non-Null Count  Dtype         
    ---  ------                --------------  -----         
     0   OrderID               2155 non-null   int64         
     1   OrderDate             2155 non-null   datetime64[ns]
     2   RequiredDate          2155 non-null   datetime64[ns]
     3   ShippedDate           2082 non-null   datetime64[ns]
     4   SupplierID            2155 non-null   int64         
     5   Supplier_CompanyName  2155 non-null   object        
     6   ContactName           2155 non-null   object        
     7   ContactTitle          2155 non-null   object        
     8   Address               2155 non-null   object        
     9   City                  2155 non-null   object        
     10  Region                731 non-null    object        
     11  Country               2155 non-null   object        
     12  ProductID             2155 non-null   int64         
     13  ProductName           2155 non-null   object        
     14  Harga_Beli            2155 non-null   object        
     15  Harga_Jual            2155 non-null   object        
     16  Quantity              2155 non-null   int64         
    dtypes: datetime64[ns](3), int64(4), object(10)
    memory usage: 286.3+ KB
    

## **3.1 Melihat Data Sekilas Dari General Info**

Terlihat sekilas bahwa secara keseliruhan terdapat 2155 baris data dengan total 18 kolom. Setiap kolomnya memiliki tipe data yang berbeda-beda. Ada Object, integer, dan datetime.

Pertama-tama mari berfokus pada non-null values atau data yang tersedia pada setiap kolomnya.  Jika melihat informasi tersebut, terdapat 1 kolom atau feature yang memiliki data tidak lengkap yaitu pada kolom ```Region```. Feature ```Region``` kehilangan lebih dari 65% data.  
#### ___**Kesimpulan pertama adalah bahwa terdapat *missing value* yang harus ditanggulangi.**___

Kedua kita akan Berfokus pada features berikut ini: 
1. Harga_Beli
2. Harga_Jual
3. Total_Harga

Pada tipe data Harga_Jual, Harga_Beli, dan juga Total_Harga. Ketiga feature ini merupakan feature yang seharusnya memiliki tipe data numerik (dibuktikan pada preview data di bagian sebelumnya), sedangkan yang terbaca tipe data dari ketiga feature ini adalah object. Artinya, ketiga feature ini tidak dianggap memiliki komponen data yang numerik. Tentu saja hal tersebut harus ditanggulangi, mengingat ke depannya data yang bersifat numerik ini akan digunakan. 
#### ___**Kesimpulan keduanya adalah terdapat features yang memiliki tipe data yang salah dan harus diubah sesuai dengan tipe data seharusnya.**___


```python
# Check Missing Value Percentage

tabel1.isna().sum()
```




    OrderID                    0
    OrderDate                  0
    RequiredDate               0
    ShippedDate               73
    SupplierID                 0
    Supplier_CompanyName       0
    ContactName                0
    ContactTitle               0
    Address                    0
    City                       0
    Region                  1424
    Country                    0
    ProductID                  0
    ProductName                0
    Harga_Beli                 0
    Harga_Jual                 0
    Quantity                   0
    dtype: int64



## **3.2 DATA TYPE, MISSING VALUE & ANOMALY**

### **3.2.1 Missing Values**

Pertama Pada fuature ```Region``` total missing value 1424 data lebih dari 65% dari total. Artinya, jika missing valuenya dihilangkan dengan melihat row atau barisnya, lebih dari setengah dari data yang dimiliki akan hilang, yang berarti akan mengakibatkan hilangnya banyak informasi. Tentu saja hal tersebut tidak dibenarkan. Untuk mengatasi hal tersebut, maka feature tersebut akan dihapus dan tidak akan dimasukkan ke dalam data yang akan dianalisis.

Kedua pada Future ```ShippedDate``` Total missing value 73 kurang dari 5%. Untuk missing value ini kita akan melakukan drop data pada masing value (hal tersebut karena 73 dari 2155 data hanya kurang lebih 5% data, sehingga tidak akan mengurangi informasi secara signifikan)

Untuk feature atau kolom lain tidak memiliki missing value sehingga dapat digunakan untuk analisa lebih lanjut


```python
# Check data missing falue pada feature ShippedDate

tabel1[tabel1['ShippedDate'].isna()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Region</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>15</th>
      <td>11070</td>
      <td>1998-05-05</td>
      <td>1998-06-02</td>
      <td>NaT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0000</td>
      <td>18.0000</td>
      <td>40</td>
    </tr>
    <tr>
      <th>78</th>
      <td>11070</td>
      <td>1998-05-05</td>
      <td>1998-06-02</td>
      <td>NaT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>19.0000</td>
      <td>19.0000</td>
      <td>20</td>
    </tr>
    <tr>
      <th>79</th>
      <td>11072</td>
      <td>1998-05-05</td>
      <td>1998-06-02</td>
      <td>NaT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>19.0000</td>
      <td>19.0000</td>
      <td>8</td>
    </tr>
    <tr>
      <th>80</th>
      <td>11075</td>
      <td>1998-05-06</td>
      <td>1998-06-03</td>
      <td>NaT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>19.0000</td>
      <td>19.0000</td>
      <td>10</td>
    </tr>
    <tr>
      <th>81</th>
      <td>11077</td>
      <td>1998-05-06</td>
      <td>1998-06-03</td>
      <td>NaT</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>None</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>19.0000</td>
      <td>19.0000</td>
      <td>24</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2078</th>
      <td>11058</td>
      <td>1998-04-29</td>
      <td>1998-05-27</td>
      <td>NaT</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>None</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0000</td>
      <td>34.0000</td>
      <td>21</td>
    </tr>
    <tr>
      <th>2079</th>
      <td>11059</td>
      <td>1998-04-29</td>
      <td>1998-06-10</td>
      <td>NaT</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>None</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0000</td>
      <td>34.0000</td>
      <td>35</td>
    </tr>
    <tr>
      <th>2081</th>
      <td>11061</td>
      <td>1998-04-30</td>
      <td>1998-06-11</td>
      <td>NaT</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>None</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0000</td>
      <td>34.0000</td>
      <td>15</td>
    </tr>
    <tr>
      <th>2082</th>
      <td>11077</td>
      <td>1998-05-06</td>
      <td>1998-06-03</td>
      <td>NaT</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>None</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0000</td>
      <td>34.0000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2093</th>
      <td>11058</td>
      <td>1998-04-29</td>
      <td>1998-05-27</td>
      <td>NaT</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Qubec</td>
      <td>Canada</td>
      <td>61</td>
      <td>Sirop d'rable</td>
      <td>28.5000</td>
      <td>28.5000</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
<p>73 rows × 17 columns</p>
</div>



### **3.2.2 Handling Missing Values**

Setelah mengetahui hal-hal yang harus 'dibereskan' terlebih dahulu, maka pada bagian ini, hal-hal tersebut harus diatasi.
Pertama, masalah missing value. Sebenarnya pada bagian sebelumnya sudah diberi tahu apa saja yang harus dilakukan untuk mengatasi masalah tersebut. Bagian pertama jelas akan dilakukan drop features ```Region```. Setelah drop feature tersebut, baru di lanjitkan analisa lebih lanjut.


```python
# Remove Missing Value 1

tabel1.drop(['Region'], axis=1, inplace = True)
```


```python
# Remove Missing Value 2

tabel1.dropna(inplace = True)
```


```python
# Recheck info

tabel1.isnull().sum()
```




    OrderID                 0
    OrderDate               0
    RequiredDate            0
    ShippedDate             0
    SupplierID              0
    Supplier_CompanyName    0
    ContactName             0
    ContactTitle            0
    Address                 0
    City                    0
    Country                 0
    ProductID               0
    ProductName             0
    Harga_Beli              0
    Harga_Jual              0
    Quantity                0
    dtype: int64



### **3.2.3 Recheck Missing Value Information**

Setelah melakukan proses drop missing value, baik itu drop terhadap features, selanjutnya adalah melakukan pengecekan terhadap data yang dimiliki untuk memastikan apakah sudah tidak ada missing value lagi. Benar saja, jika melihat data pada output di atas, sudah tidak terdapat lagi missing value sama sekali, dan pada feature ```Region``` sudah tidak ada juga (karena sudah di-drop). Untuk masing-masing feature juga sudah memiliki 0 missing value. 

**Kesimpulan masalah missing value sudah teratasi.**

## **3.3 Mengubah Tipe Data Yang Salah**

Oke, telah disebutkan juga sebelumnya bahwa ada tipe data yang tidak sesuai. ketiga features tersebut terlebih dahulu diubah agar fungsionalitasnya kembali ke hakekatnya. Numerik akan diperlakukan sebagai tipe data numerik. Tujuannya tentu saja agar features tersebut dapat dipergunakan sebagaimana mestinya.


```python
# Change Spesific Column To Numeric

tabel1['Harga_Jual'] = pd.to_numeric(tabel1['Harga_Jual'])
tabel1['Harga_Beli'] = pd.to_numeric(tabel1['Harga_Beli'])
```


```python
#Rechack Info

tabel1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2082 entries, 0 to 2154
    Data columns (total 16 columns):
     #   Column                Non-Null Count  Dtype         
    ---  ------                --------------  -----         
     0   OrderID               2082 non-null   int64         
     1   OrderDate             2082 non-null   datetime64[ns]
     2   RequiredDate          2082 non-null   datetime64[ns]
     3   ShippedDate           2082 non-null   datetime64[ns]
     4   SupplierID            2082 non-null   int64         
     5   Supplier_CompanyName  2082 non-null   object        
     6   ContactName           2082 non-null   object        
     7   ContactTitle          2082 non-null   object        
     8   Address               2082 non-null   object        
     9   City                  2082 non-null   object        
     10  Country               2082 non-null   object        
     11  ProductID             2082 non-null   int64         
     12  ProductName           2082 non-null   object        
     13  Harga_Beli            2082 non-null   float64       
     14  Harga_Jual            2082 non-null   float64       
     15  Quantity              2082 non-null   int64         
    dtypes: datetime64[ns](3), float64(2), int64(4), object(7)
    memory usage: 276.5+ KB
    


```python
# Check Dupliacate

tabel1[tabel1.duplicated()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



## **3.4 Data Duplicate**

Anomali berikutnya yang bisa ditemui adalah data yang duplikat. Tentu saja data yang bersifat duplikat ini akan menjadi sesuatu hal yang akan mengganggu proses analisis data. Jika memang nantinya terdapat data yang duplikat, sebaiknya data duplikatnya dihapus dan disisakan data yang unique saja. Untuk data saat ini, melihat output di atas artinya tidak terdapat data yang duplikat. Dengan begitu tidak perlu ada action yang dilakukan.

## **3.5 Membuat Feature**

### **3.5.1 Feature Profit_Each, Total_Harga, Profit**
- Profit Each = Menginformasikan keuntungan per produk tiap transaksi (Harga Jual - Harga Beli)
- Total Harga = Menginformasikan total penjualan tiap transaksi (Harga Jual x Quantity)
- Profit = Menginformasikan keuntungan tiap transaksi (Profit Each x Quantity)


```python
# Add New Column (Profit_Each, Total_Harga, Profit)

tabel1['Profit_Each'] =  tabel1['Harga_Jual'] - tabel1['Harga_Beli']
tabel1['Total_Harga'] = tabel1['Harga_Jual']*tabel1['Quantity']
tabel1['Profit'] = tabel1['Profit_Each']*tabel1['Quantity']
```

### **3.5.2 Feature 'ProcessingDate'**

Data awal menunjukan terdapat 2 features yang merupakan tipe data datetime. Artinya, kita dapat melakukan ekstraksi informasi tambahan dari kedua features tersebut. Sebelumnya, kita perlu tahu dulu definisi dari kedua tabel tersebut. requiredDate secara singkat dapat diartikan sebagai waktu atau kapan barang tersebut dibutuhkan, sedangkat shippedDate adalah waktu dikirimkannya barang tersebut. 

Melihat kedua definisi tersebut, sebuah informasi dapat diambil, yakni seberapa lama waktu proses barangnya dari waktu pengiriman hingga dibutuhkan. Oleh karena itu, untuk mendapatkan informasinya, maka perlu dilakukan pengurangan antara requiredDate dan juga shippedDate. Mungkin akan timbul pertanyaan, apakah waktu dapat dikurangkan? Jawabannya, bisa. Output yang keluar nantinya akan berupa selisih atau lamanya waktu proses tersebut dalam satuan hari.


```python
# Add New Column (Processing Day)

tabel1['ProcessingDate'] = tabel1['RequiredDate'] - tabel1['ShippedDate']
tabel1.sample(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
      <th>Profit_Each</th>
      <th>Total_Harga</th>
      <th>Profit</th>
      <th>ProcessingDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1459</th>
      <td>10351</td>
      <td>1996-11-11</td>
      <td>1996-12-09</td>
      <td>1996-11-20</td>
      <td>19</td>
      <td>New England Seafood Cannery</td>
      <td>Robb Merchant</td>
      <td>Wholesale Account Agent</td>
      <td>Order Processing Dept.\r\n2100 Paul Revere Blvd.</td>
      <td>Boston</td>
      <td>USA</td>
      <td>41</td>
      <td>Jack's New England Clam Chowder</td>
      <td>7.7</td>
      <td>9.65</td>
      <td>13</td>
      <td>1.95</td>
      <td>125.45</td>
      <td>25.35</td>
      <td>19 days</td>
    </tr>
    <tr>
      <th>39</th>
      <td>10258</td>
      <td>1996-07-17</td>
      <td>1996-08-14</td>
      <td>1996-07-23</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>15.2</td>
      <td>19.00</td>
      <td>50</td>
      <td>3.80</td>
      <td>950.00</td>
      <td>190.00</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>457</th>
      <td>10550</td>
      <td>1997-05-28</td>
      <td>1997-06-25</td>
      <td>1997-06-06</td>
      <td>7</td>
      <td>Pavlova, Ltd.</td>
      <td>Ian Devling</td>
      <td>Marketing Manager</td>
      <td>74 Rose St.\r\nMoonie Ponds</td>
      <td>Melbourne</td>
      <td>Australia</td>
      <td>17</td>
      <td>Alice Mutton</td>
      <td>39.0</td>
      <td>39.00</td>
      <td>8</td>
      <td>0.00</td>
      <td>312.00</td>
      <td>0.00</td>
      <td>19 days</td>
    </tr>
    <tr>
      <th>701</th>
      <td>10621</td>
      <td>1997-08-05</td>
      <td>1997-09-02</td>
      <td>1997-08-11</td>
      <td>9</td>
      <td>PB Knckebrd AB</td>
      <td>Lars Peterson</td>
      <td>Sales Agent</td>
      <td>Kaloadagatan 13</td>
      <td>Gteborg</td>
      <td>Sweden</td>
      <td>23</td>
      <td>Tunnbrd</td>
      <td>9.0</td>
      <td>9.00</td>
      <td>10</td>
      <td>0.00</td>
      <td>90.00</td>
      <td>0.00</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>619</th>
      <td>10368</td>
      <td>1996-11-29</td>
      <td>1996-12-27</td>
      <td>1996-12-02</td>
      <td>8</td>
      <td>Specialty Biscuits, Ltd.</td>
      <td>Peter Wilson</td>
      <td>Sales Representative</td>
      <td>29 King's Way</td>
      <td>Manchester</td>
      <td>UK</td>
      <td>21</td>
      <td>Sir Rodney's Scones</td>
      <td>8.0</td>
      <td>10.00</td>
      <td>5</td>
      <td>2.00</td>
      <td>50.00</td>
      <td>10.00</td>
      <td>25 days</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check Data Anomalies in Date Time Fomrat

tabel1['ProcessingDate'].value_counts()
```




    21 days     227
    22 days     191
    23 days     165
    25 days     160
    19 days     159
    24 days     158
    26 days     145
    20 days     139
    18 days      98
    16 days      39
    7 days       37
    10 days      37
    27 days      35
    17 days      34
    5 days       31
    36 days      31
    8 days       30
    4 days       27
    9 days       24
    34 days      23
    35 days      23
    11 days      22
    6 days       21
    -1 days      20
    13 days      18
    -7 days      16
    39 days      16
    -6 days      15
    32 days      15
    15 days      15
    3 days       11
    0 days       10
    -4 days       8
    14 days       8
    -2 days       6
    -5 days       6
    33 days       6
    1 days        6
    30 days       6
    37 days       5
    -16 days      5
    12 days       5
    2 days        5
    -18 days      4
    -9 days       4
    41 days       3
    38 days       3
    -8 days       2
    -11 days      2
    -3 days       2
    28 days       2
    -23 days      1
    -17 days      1
    Name: ProcessingDate, dtype: int64



### **3.5.3 Anomali Pada Processing Date**

Melihat output unique values beserta dengan banyaknya data di setiap unique values tersebut, apakah terlihat sesuatu yang aneh? Yap benar, terdapat sebuah waktu yang menunjukan nilai minus, -56 days, dan terdapat 18 data di dalamnya. Ada apa? Sebenarnya ada 2 asumsi yang bisa diambil. Asumsi pertama adalah murni kesalahan input saat memasukan ke dalam database, atau asumsi yang kedua adalah pengirimannya mengalami keterlambatan. 

Untuk asumsi yang pertama, cara mengatasinya cukup dengan drop 18 data yang salah input. Dengan kata lain, kita menganggap bahwa data tersebut 'salah' dan dapat dibuang (karena jumlahnya yang tidak banyak). Untuk asumsi kedua, data ini bisa saja dipertahankan dan bisa dilakukan analisis lebih lanjut untuk mengetahui letak permasalahannya.

Di sini, asumsi yang akan digunakan adalah asumsi yang kedua, yaitu kita akan mengasumsikan terdapat keterlambatan dalam pengiriman. Nantinya, kita akan coba melakukan analisis untuk data ini.


```python
# Handling Date Time Format Anomalies (Check Data First)

tabel1[tabel1['ProcessingDate'] < '0 days']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
      <th>Profit_Each</th>
      <th>Total_Harga</th>
      <th>Profit</th>
      <th>ProcessingDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>10847</td>
      <td>1998-01-22</td>
      <td>1998-02-05</td>
      <td>1998-02-10</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>1</td>
      <td>Chai</td>
      <td>18.0</td>
      <td>18.00</td>
      <td>80</td>
      <td>0.00</td>
      <td>1440.0</td>
      <td>0.0</td>
      <td>-5 days</td>
    </tr>
    <tr>
      <th>40</th>
      <td>10264</td>
      <td>1996-07-24</td>
      <td>1996-08-21</td>
      <td>1996-08-23</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>15.2</td>
      <td>19.00</td>
      <td>35</td>
      <td>3.80</td>
      <td>665.0</td>
      <td>133.0</td>
      <td>-2 days</td>
    </tr>
    <tr>
      <th>94</th>
      <td>10309</td>
      <td>1996-09-19</td>
      <td>1996-10-17</td>
      <td>1996-10-23</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>4</td>
      <td>Chef Anton's Cajun Seasoning</td>
      <td>17.6</td>
      <td>22.00</td>
      <td>20</td>
      <td>4.40</td>
      <td>440.0</td>
      <td>88.0</td>
      <td>-6 days</td>
    </tr>
    <tr>
      <th>108</th>
      <td>10726</td>
      <td>1997-11-03</td>
      <td>1997-11-17</td>
      <td>1997-12-05</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>4</td>
      <td>Chef Anton's Cajun Seasoning</td>
      <td>22.0</td>
      <td>22.00</td>
      <td>25</td>
      <td>0.00</td>
      <td>550.0</td>
      <td>0.0</td>
      <td>-18 days</td>
    </tr>
    <tr>
      <th>134</th>
      <td>10451</td>
      <td>1997-02-19</td>
      <td>1997-03-05</td>
      <td>1997-03-12</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>65</td>
      <td>Louisiana Fiery Hot Pepper Sauce</td>
      <td>16.8</td>
      <td>21.05</td>
      <td>28</td>
      <td>4.25</td>
      <td>589.4</td>
      <td>119.0</td>
      <td>-7 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2047</th>
      <td>10515</td>
      <td>1997-04-23</td>
      <td>1997-05-07</td>
      <td>1997-05-23</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0</td>
      <td>34.00</td>
      <td>84</td>
      <td>0.00</td>
      <td>2856.0</td>
      <td>0.0</td>
      <td>-16 days</td>
    </tr>
    <tr>
      <th>2060</th>
      <td>10709</td>
      <td>1997-10-17</td>
      <td>1997-11-14</td>
      <td>1997-11-20</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0</td>
      <td>34.00</td>
      <td>10</td>
      <td>0.00</td>
      <td>340.0</td>
      <td>0.0</td>
      <td>-6 days</td>
    </tr>
    <tr>
      <th>2067</th>
      <td>10847</td>
      <td>1998-01-22</td>
      <td>1998-02-05</td>
      <td>1998-02-10</td>
      <td>28</td>
      <td>Gai pturage</td>
      <td>Eliane Noz</td>
      <td>Sales Representative</td>
      <td>Bat. B\r\n3, rue des Alpes</td>
      <td>Annecy</td>
      <td>France</td>
      <td>60</td>
      <td>Camembert Pierrot</td>
      <td>34.0</td>
      <td>34.00</td>
      <td>45</td>
      <td>0.00</td>
      <td>1530.0</td>
      <td>0.0</td>
      <td>-5 days</td>
    </tr>
    <tr>
      <th>2137</th>
      <td>10779</td>
      <td>1997-12-16</td>
      <td>1998-01-13</td>
      <td>1998-01-14</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.3</td>
      <td>49.30</td>
      <td>20</td>
      <td>0.00</td>
      <td>986.0</td>
      <td>0.0</td>
      <td>-1 days</td>
    </tr>
    <tr>
      <th>2140</th>
      <td>10816</td>
      <td>1998-01-06</td>
      <td>1998-02-03</td>
      <td>1998-02-04</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.3</td>
      <td>49.30</td>
      <td>20</td>
      <td>0.00</td>
      <td>986.0</td>
      <td>0.0</td>
      <td>-1 days</td>
    </tr>
  </tbody>
</table>
<p>92 rows × 20 columns</p>
</div>



## **3.6 Check Cleaned Data**

### **3.6.1 Preview Cleaned Data**

Setelah semua anomalies sudah diselesaikan, artinya data yang dimiliki sudah bersih. Di bawah ini adalah sample data yang dianggap sudah bersih setelah melewati proses-proses sebelumnya.


```python
# Clean Data

tabel1.sample(20)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
      <th>Profit_Each</th>
      <th>Total_Harga</th>
      <th>Profit</th>
      <th>ProcessingDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1256</th>
      <td>10573</td>
      <td>1997-06-19</td>
      <td>1997-07-17</td>
      <td>1997-06-20</td>
      <td>16</td>
      <td>Bigfoot Breweries</td>
      <td>Cheryl Saylor</td>
      <td>Regional Account Rep.</td>
      <td>3400 - 8th Avenue\r\nSuite 210</td>
      <td>Bend</td>
      <td>USA</td>
      <td>34</td>
      <td>Sasquatch Ale</td>
      <td>14.00</td>
      <td>14.00</td>
      <td>40</td>
      <td>0.00</td>
      <td>560.00</td>
      <td>0.0</td>
      <td>27 days</td>
    </tr>
    <tr>
      <th>1391</th>
      <td>10455</td>
      <td>1997-02-24</td>
      <td>1997-04-07</td>
      <td>1997-03-03</td>
      <td>18</td>
      <td>Aux joyeux ecclsiastiques</td>
      <td>Guylne Nodier</td>
      <td>Sales Manager</td>
      <td>203, Rue des Francs-Bourgeois</td>
      <td>Paris</td>
      <td>France</td>
      <td>39</td>
      <td>Chartreuse verte</td>
      <td>14.40</td>
      <td>18.00</td>
      <td>20</td>
      <td>3.60</td>
      <td>360.00</td>
      <td>72.0</td>
      <td>35 days</td>
    </tr>
    <tr>
      <th>629</th>
      <td>10513</td>
      <td>1997-04-22</td>
      <td>1997-06-03</td>
      <td>1997-04-28</td>
      <td>8</td>
      <td>Specialty Biscuits, Ltd.</td>
      <td>Peter Wilson</td>
      <td>Sales Representative</td>
      <td>29 King's Way</td>
      <td>Manchester</td>
      <td>UK</td>
      <td>21</td>
      <td>Sir Rodney's Scones</td>
      <td>10.00</td>
      <td>10.00</td>
      <td>40</td>
      <td>0.00</td>
      <td>400.00</td>
      <td>0.0</td>
      <td>36 days</td>
    </tr>
    <tr>
      <th>1485</th>
      <td>10890</td>
      <td>1998-02-16</td>
      <td>1998-03-16</td>
      <td>1998-02-18</td>
      <td>19</td>
      <td>New England Seafood Cannery</td>
      <td>Robb Merchant</td>
      <td>Wholesale Account Agent</td>
      <td>Order Processing Dept.\r\n2100 Paul Revere Blvd.</td>
      <td>Boston</td>
      <td>USA</td>
      <td>41</td>
      <td>Jack's New England Clam Chowder</td>
      <td>9.65</td>
      <td>9.65</td>
      <td>14</td>
      <td>0.00</td>
      <td>135.10</td>
      <td>0.0</td>
      <td>26 days</td>
    </tr>
    <tr>
      <th>1737</th>
      <td>10335</td>
      <td>1996-10-22</td>
      <td>1996-11-19</td>
      <td>1996-10-24</td>
      <td>24</td>
      <td>G'day, Mate</td>
      <td>Wendy Mackenzie</td>
      <td>Sales Representative</td>
      <td>170 Prince Edward Parade\r\nHunter's Hill</td>
      <td>Sydney</td>
      <td>Australia</td>
      <td>51</td>
      <td>Manjimup Dried Apples</td>
      <td>42.40</td>
      <td>53.00</td>
      <td>48</td>
      <td>10.60</td>
      <td>2544.00</td>
      <td>508.8</td>
      <td>26 days</td>
    </tr>
    <tr>
      <th>1847</th>
      <td>10794</td>
      <td>1997-12-24</td>
      <td>1998-01-21</td>
      <td>1998-01-02</td>
      <td>25</td>
      <td>Ma Maison</td>
      <td>Jean-Guy Lauzon</td>
      <td>Marketing Manager</td>
      <td>2960 Rue St. Laurent</td>
      <td>Montral</td>
      <td>Canada</td>
      <td>54</td>
      <td>Tourtire</td>
      <td>7.45</td>
      <td>7.45</td>
      <td>6</td>
      <td>0.00</td>
      <td>44.70</td>
      <td>0.0</td>
      <td>19 days</td>
    </tr>
    <tr>
      <th>809</th>
      <td>10723</td>
      <td>1997-10-30</td>
      <td>1997-11-27</td>
      <td>1997-11-25</td>
      <td>11</td>
      <td>Heli Swaren GmbH &amp; Co. KG</td>
      <td>Petra Winkler</td>
      <td>Sales Manager</td>
      <td>Tiergartenstrae 5</td>
      <td>Berlin</td>
      <td>Germany</td>
      <td>26</td>
      <td>Gumbr Gummibrchen</td>
      <td>31.23</td>
      <td>31.23</td>
      <td>15</td>
      <td>0.00</td>
      <td>468.45</td>
      <td>0.0</td>
      <td>2 days</td>
    </tr>
    <tr>
      <th>49</th>
      <td>10469</td>
      <td>1997-03-10</td>
      <td>1997-04-07</td>
      <td>1997-03-14</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>15.20</td>
      <td>19.00</td>
      <td>40</td>
      <td>3.80</td>
      <td>760.00</td>
      <td>152.0</td>
      <td>24 days</td>
    </tr>
    <tr>
      <th>2120</th>
      <td>10449</td>
      <td>1997-02-18</td>
      <td>1997-03-18</td>
      <td>1997-02-27</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>39.40</td>
      <td>49.30</td>
      <td>35</td>
      <td>9.90</td>
      <td>1725.50</td>
      <td>346.5</td>
      <td>19 days</td>
    </tr>
    <tr>
      <th>1706</th>
      <td>10749</td>
      <td>1997-11-20</td>
      <td>1997-12-18</td>
      <td>1997-12-19</td>
      <td>23</td>
      <td>Karkki Oy</td>
      <td>Anne Heikkonen</td>
      <td>Product Manager</td>
      <td>Valtakatu 12</td>
      <td>Lappeenranta</td>
      <td>Finland</td>
      <td>76</td>
      <td>Lakkalikri</td>
      <td>18.00</td>
      <td>18.00</td>
      <td>10</td>
      <td>0.00</td>
      <td>180.00</td>
      <td>0.0</td>
      <td>-1 days</td>
    </tr>
    <tr>
      <th>424</th>
      <td>10932</td>
      <td>1998-03-06</td>
      <td>1998-04-03</td>
      <td>1998-03-24</td>
      <td>7</td>
      <td>Pavlova, Ltd.</td>
      <td>Ian Devling</td>
      <td>Marketing Manager</td>
      <td>74 Rose St.\r\nMoonie Ponds</td>
      <td>Melbourne</td>
      <td>Australia</td>
      <td>16</td>
      <td>Pavlova</td>
      <td>17.45</td>
      <td>17.45</td>
      <td>30</td>
      <td>0.00</td>
      <td>523.50</td>
      <td>0.0</td>
      <td>10 days</td>
    </tr>
    <tr>
      <th>114</th>
      <td>10848</td>
      <td>1998-01-23</td>
      <td>1998-02-20</td>
      <td>1998-01-29</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>5</td>
      <td>Chef Anton's Gumbo Mix</td>
      <td>21.35</td>
      <td>21.35</td>
      <td>30</td>
      <td>0.00</td>
      <td>640.50</td>
      <td>0.0</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>1555</th>
      <td>11023</td>
      <td>1998-04-14</td>
      <td>1998-04-28</td>
      <td>1998-04-24</td>
      <td>20</td>
      <td>Leka Trading</td>
      <td>Chandra Leka</td>
      <td>Owner</td>
      <td>471 Serangoon Loop, Suite #402</td>
      <td>Singapore</td>
      <td>Singapore</td>
      <td>43</td>
      <td>Ipoh Coffee</td>
      <td>46.00</td>
      <td>46.00</td>
      <td>30</td>
      <td>0.00</td>
      <td>1380.00</td>
      <td>0.0</td>
      <td>4 days</td>
    </tr>
    <tr>
      <th>409</th>
      <td>10671</td>
      <td>1997-09-17</td>
      <td>1997-10-15</td>
      <td>1997-09-24</td>
      <td>7</td>
      <td>Pavlova, Ltd.</td>
      <td>Ian Devling</td>
      <td>Marketing Manager</td>
      <td>74 Rose St.\r\nMoonie Ponds</td>
      <td>Melbourne</td>
      <td>Australia</td>
      <td>16</td>
      <td>Pavlova</td>
      <td>17.45</td>
      <td>17.45</td>
      <td>10</td>
      <td>0.00</td>
      <td>174.50</td>
      <td>0.0</td>
      <td>21 days</td>
    </tr>
    <tr>
      <th>46</th>
      <td>10418</td>
      <td>1997-01-17</td>
      <td>1997-02-14</td>
      <td>1997-01-24</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>15.20</td>
      <td>19.00</td>
      <td>60</td>
      <td>3.80</td>
      <td>1140.00</td>
      <td>228.0</td>
      <td>21 days</td>
    </tr>
    <tr>
      <th>939</th>
      <td>10634</td>
      <td>1997-08-15</td>
      <td>1997-09-12</td>
      <td>1997-08-21</td>
      <td>12</td>
      <td>Plutzer Lebensmittelgromrkte AG</td>
      <td>Martin Bein</td>
      <td>International Marketing Mgr.</td>
      <td>Bogenallee 51</td>
      <td>Frankfurt</td>
      <td>Germany</td>
      <td>75</td>
      <td>Rhnbru Klosterbier</td>
      <td>7.75</td>
      <td>7.75</td>
      <td>2</td>
      <td>0.00</td>
      <td>15.50</td>
      <td>0.0</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>373</th>
      <td>10555</td>
      <td>1997-06-02</td>
      <td>1997-06-30</td>
      <td>1997-06-04</td>
      <td>6</td>
      <td>Mayumi's</td>
      <td>Mayumi Ohno</td>
      <td>Marketing Representative</td>
      <td>92 Setsuko\r\nChuo-ku</td>
      <td>Osaka</td>
      <td>Japan</td>
      <td>14</td>
      <td>Tofu</td>
      <td>23.25</td>
      <td>23.25</td>
      <td>30</td>
      <td>0.00</td>
      <td>697.50</td>
      <td>0.0</td>
      <td>26 days</td>
    </tr>
    <tr>
      <th>1826</th>
      <td>10375</td>
      <td>1996-12-06</td>
      <td>1997-01-03</td>
      <td>1996-12-09</td>
      <td>25</td>
      <td>Ma Maison</td>
      <td>Jean-Guy Lauzon</td>
      <td>Marketing Manager</td>
      <td>2960 Rue St. Laurent</td>
      <td>Montral</td>
      <td>Canada</td>
      <td>54</td>
      <td>Tourtire</td>
      <td>5.90</td>
      <td>7.45</td>
      <td>10</td>
      <td>1.55</td>
      <td>74.50</td>
      <td>15.5</td>
      <td>25 days</td>
    </tr>
    <tr>
      <th>716</th>
      <td>10893</td>
      <td>1998-02-18</td>
      <td>1998-03-18</td>
      <td>1998-02-20</td>
      <td>10</td>
      <td>Refrescos Americanas LTDA</td>
      <td>Carlos Diaz</td>
      <td>Marketing Manager</td>
      <td>Av. das Americanas 12.890</td>
      <td>So Paulo</td>
      <td>Brazil</td>
      <td>24</td>
      <td>Guaran Fantstica</td>
      <td>4.50</td>
      <td>4.50</td>
      <td>10</td>
      <td>0.00</td>
      <td>45.00</td>
      <td>0.0</td>
      <td>26 days</td>
    </tr>
    <tr>
      <th>132</th>
      <td>10401</td>
      <td>1997-01-01</td>
      <td>1997-01-29</td>
      <td>1997-01-10</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>65</td>
      <td>Louisiana Fiery Hot Pepper Sauce</td>
      <td>16.80</td>
      <td>21.05</td>
      <td>20</td>
      <td>4.25</td>
      <td>421.00</td>
      <td>85.0</td>
      <td>19 days</td>
    </tr>
  </tbody>
</table>
</div>



### **3.6.2 General Info Cleaned Data**


```python
# Check Some Info

listItem = []
for col in tabel1.columns :
    listItem.append([col, tabel1[col].dtype, len(tabel1),tabel1[col].isna().sum(), round((tabel1[col].isna().sum()/len(tabel1[col])) * 100,2),
                    tabel1[col].nunique(), (tabel1[col].drop_duplicates().sample(2).values)])

tabel1Desc = pd.DataFrame(columns=['Column Name', 'Data Type', 'Data Count', 'Missing Value', 
    'Missing Value Percentage', 'Number of Unique', 'Unique Sample'],
                     data=listItem)
tabel1Desc
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Column Name</th>
      <th>Data Type</th>
      <th>Data Count</th>
      <th>Missing Value</th>
      <th>Missing Value Percentage</th>
      <th>Number of Unique</th>
      <th>Unique Sample</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>OrderID</td>
      <td>int64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>809</td>
      <td>[10289, 10747]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>OrderDate</td>
      <td>datetime64[ns]</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>478</td>
      <td>[1997-10-21T00:00:00.000000000, 1998-01-16T00:...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>RequiredDate</td>
      <td>datetime64[ns]</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>450</td>
      <td>[1997-12-03T00:00:00.000000000, 1998-01-09T00:...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ShippedDate</td>
      <td>datetime64[ns]</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>387</td>
      <td>[1997-03-03T00:00:00.000000000, 1997-10-13T00:...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SupplierID</td>
      <td>int64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>29</td>
      <td>[22, 1]</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Supplier_CompanyName</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>29</td>
      <td>[Mayumi's, Specialty Biscuits, Ltd.]</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ContactName</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>29</td>
      <td>[Petra Winkler, Mayumi Ohno]</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ContactTitle</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>15</td>
      <td>[Sales Representative, International Marketing...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Address</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>29</td>
      <td>[2960 Rue St. Laurent, Brovallavgen 231]</td>
    </tr>
    <tr>
      <th>9</th>
      <td>City</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>29</td>
      <td>[Sandvika, Manchester]</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Country</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>17</td>
      <td>[USA, Denmark]</td>
    </tr>
    <tr>
      <th>11</th>
      <td>ProductID</td>
      <td>int64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>77</td>
      <td>[11, 20]</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ProductName</td>
      <td>object</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>77</td>
      <td>[Tunnbrd, Gravad lax]</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Harga_Beli</td>
      <td>float64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>116</td>
      <td>[12.75, 17.6]</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Harga_Jual</td>
      <td>float64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>62</td>
      <td>[81.0, 9.2]</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Quantity</td>
      <td>int64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>54</td>
      <td>[1, 100]</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Profit_Each</td>
      <td>float64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>62</td>
      <td>[16.200000000000003, 2.4000000000000004]</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Total_Harga</td>
      <td>float64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>712</td>
      <td>[325.5, 770.0]</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Profit</td>
      <td>float64</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>410</td>
      <td>[330.0, 867.6500000000002]</td>
    </tr>
    <tr>
      <th>19</th>
      <td>ProcessingDate</td>
      <td>timedelta64[ns]</td>
      <td>2082</td>
      <td>0</td>
      <td>0.0</td>
      <td>53</td>
      <td>[-345600000000000 nanoseconds, 777600000000000...</td>
    </tr>
  </tbody>
</table>
</div>




```python
tabel1_sum = tabel1[['Supplier_CompanyName', 'Quantity', 'Total_Harga']].groupby('Supplier_CompanyName').sum()
tabel1_sum.rename({'Total_Harga':'Total_Penjulan', 'old_col2':'new_col2'}, inplace = True)
tabel1_sum
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
      <th>Total_Harga</th>
    </tr>
    <tr>
      <th>Supplier_CompanyName</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aux joyeux ecclsiastiques</th>
      <td>1414</td>
      <td>178398.50</td>
    </tr>
    <tr>
      <th>Bigfoot Breweries</th>
      <td>1439</td>
      <td>23582.00</td>
    </tr>
    <tr>
      <th>Cooperativa de Quesos 'Las Cabras'</th>
      <td>1038</td>
      <td>27612.00</td>
    </tr>
    <tr>
      <th>Escargots Nouveaux</th>
      <td>534</td>
      <td>7075.50</td>
    </tr>
    <tr>
      <th>Exotic Liquids</th>
      <td>2107</td>
      <td>36329.00</td>
    </tr>
    <tr>
      <th>Formaggi Fortini s.r.l.</th>
      <td>2479</td>
      <td>54733.30</td>
    </tr>
    <tr>
      <th>Forts d'rables</th>
      <td>1682</td>
      <td>70463.40</td>
    </tr>
    <tr>
      <th>G'day, Mate</th>
      <td>2072</td>
      <td>72525.60</td>
    </tr>
    <tr>
      <th>Gai pturage</th>
      <td>3000</td>
      <td>133416.00</td>
    </tr>
    <tr>
      <th>Grandma Kelly's Homestead</th>
      <td>1397</td>
      <td>44210.00</td>
    </tr>
    <tr>
      <th>Heli Swaren GmbH &amp; Co. KG</th>
      <td>1436</td>
      <td>43991.69</td>
    </tr>
    <tr>
      <th>Karkki Oy</th>
      <td>1650</td>
      <td>30243.25</td>
    </tr>
    <tr>
      <th>Leka Trading</th>
      <td>1842</td>
      <td>46471.45</td>
    </tr>
    <tr>
      <th>Lyngbysild</th>
      <td>1020</td>
      <td>10970.00</td>
    </tr>
    <tr>
      <th>Ma Maison</th>
      <td>1636</td>
      <td>27099.75</td>
    </tr>
    <tr>
      <th>Mayumi's</th>
      <td>1352</td>
      <td>15877.75</td>
    </tr>
    <tr>
      <th>New England Seafood Cannery</th>
      <td>2041</td>
      <td>29346.90</td>
    </tr>
    <tr>
      <th>New Orleans Cajun Delights</th>
      <td>1733</td>
      <td>36034.55</td>
    </tr>
    <tr>
      <th>Nord-Ost-Fisch Handelsgesellschaft mbH</th>
      <td>608</td>
      <td>15741.12</td>
    </tr>
    <tr>
      <th>Norske Meierier</th>
      <td>2480</td>
      <td>49803.00</td>
    </tr>
    <tr>
      <th>PB Knckebrd AB</th>
      <td>926</td>
      <td>12510.00</td>
    </tr>
    <tr>
      <th>Pasta Buttini s.r.l.</th>
      <td>1669</td>
      <td>55911.00</td>
    </tr>
    <tr>
      <th>Pavlova, Ltd.</th>
      <td>3867</td>
      <td>122376.40</td>
    </tr>
    <tr>
      <th>Plutzer Lebensmittelgromrkte AG</th>
      <td>3808</td>
      <td>156091.79</td>
    </tr>
    <tr>
      <th>Refrescos Americanas LTDA</th>
      <td>1095</td>
      <td>4927.50</td>
    </tr>
    <tr>
      <th>Specialty Biscuits, Ltd.</th>
      <td>2817</td>
      <td>51749.10</td>
    </tr>
    <tr>
      <th>Svensk Sjfda AB</th>
      <td>1221</td>
      <td>22910.00</td>
    </tr>
    <tr>
      <th>Tokyo Traders</th>
      <td>1133</td>
      <td>35156.00</td>
    </tr>
    <tr>
      <th>Zaanse Snoepfabriek</th>
      <td>623</td>
      <td>6367.00</td>
    </tr>
  </tbody>
</table>
</div>



## **3.7 Data Outlier**
Outlier adalah data yang menginformasikan suatu data yang jauh berbeda dibandingkan terhadap keseluruhan data. Tidak setiap outlier itu buruk dan harus di hapus atau di hilangkan. Dalam kasus ini outlier menunjukan penjualan suatu produk yang memiliki nilai atau value yang tinggi dan data ini bagus untuk perusahaan
**Kesimpulan data Outlier akan tetap di gunakan untuk analisa**


```python
# Outlier Check With Function

Q1_amount = tabel1['Total_Harga'].describe()['25%']
Q3_amount = tabel1['Total_Harga'].describe()['75%']
iqr = Q3_amount - Q1_amount

outlier_index = tabel1[(tabel1['Total_Harga'] < Q1_amount - (1.5 * iqr)) | (tabel1['Total_Harga']> Q3_amount + (1.5 * iqr)) ].index
not_outlier_index = tabel1[(tabel1['Total_Harga'] > Q1_amount - (1.5 * iqr)) & (tabel1['Total_Harga']< Q3_amount + (1.5 * iqr)) ].index
tabel1.loc[outlier_index]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderID</th>
      <th>OrderDate</th>
      <th>RequiredDate</th>
      <th>ShippedDate</th>
      <th>SupplierID</th>
      <th>Supplier_CompanyName</th>
      <th>ContactName</th>
      <th>ContactTitle</th>
      <th>Address</th>
      <th>City</th>
      <th>Country</th>
      <th>ProductID</th>
      <th>ProductName</th>
      <th>Harga_Beli</th>
      <th>Harga_Jual</th>
      <th>Quantity</th>
      <th>Profit_Each</th>
      <th>Total_Harga</th>
      <th>Profit</th>
      <th>ProcessingDate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>75</th>
      <td>11030</td>
      <td>1998-04-17</td>
      <td>1998-05-15</td>
      <td>1998-04-27</td>
      <td>1</td>
      <td>Exotic Liquids</td>
      <td>Charlotte Cooper</td>
      <td>Purchasing Manager</td>
      <td>49 Gilbert St.</td>
      <td>London</td>
      <td>UK</td>
      <td>2</td>
      <td>Chang</td>
      <td>19.00</td>
      <td>19.00</td>
      <td>100</td>
      <td>0.0</td>
      <td>1900.0</td>
      <td>0.0</td>
      <td>18 days</td>
    </tr>
    <tr>
      <th>148</th>
      <td>10765</td>
      <td>1997-12-04</td>
      <td>1998-01-01</td>
      <td>1997-12-09</td>
      <td>2</td>
      <td>New Orleans Cajun Delights</td>
      <td>Shelley Burke</td>
      <td>Order Administrator</td>
      <td>P.O. Box 78934</td>
      <td>New Orleans</td>
      <td>USA</td>
      <td>65</td>
      <td>Louisiana Fiery Hot Pepper Sauce</td>
      <td>21.05</td>
      <td>21.05</td>
      <td>80</td>
      <td>0.0</td>
      <td>1684.0</td>
      <td>0.0</td>
      <td>23 days</td>
    </tr>
    <tr>
      <th>166</th>
      <td>10618</td>
      <td>1997-08-01</td>
      <td>1997-09-12</td>
      <td>1997-08-08</td>
      <td>3</td>
      <td>Grandma Kelly's Homestead</td>
      <td>Regina Murphy</td>
      <td>Sales Representative</td>
      <td>707 Oxford Rd.</td>
      <td>Ann Arbor</td>
      <td>USA</td>
      <td>6</td>
      <td>Grandma's Boysenberry Spread</td>
      <td>25.00</td>
      <td>25.00</td>
      <td>70</td>
      <td>0.0</td>
      <td>1750.0</td>
      <td>0.0</td>
      <td>35 days</td>
    </tr>
    <tr>
      <th>185</th>
      <td>10987</td>
      <td>1998-03-31</td>
      <td>1998-04-28</td>
      <td>1998-04-06</td>
      <td>3</td>
      <td>Grandma Kelly's Homestead</td>
      <td>Regina Murphy</td>
      <td>Sales Representative</td>
      <td>707 Oxford Rd.</td>
      <td>Ann Arbor</td>
      <td>USA</td>
      <td>7</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>30.00</td>
      <td>30.00</td>
      <td>60</td>
      <td>0.0</td>
      <td>1800.0</td>
      <td>0.0</td>
      <td>22 days</td>
    </tr>
    <tr>
      <th>186</th>
      <td>10988</td>
      <td>1998-03-31</td>
      <td>1998-04-28</td>
      <td>1998-04-10</td>
      <td>3</td>
      <td>Grandma Kelly's Homestead</td>
      <td>Regina Murphy</td>
      <td>Sales Representative</td>
      <td>707 Oxford Rd.</td>
      <td>Ann Arbor</td>
      <td>USA</td>
      <td>7</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>30.00</td>
      <td>30.00</td>
      <td>60</td>
      <td>0.0</td>
      <td>1800.0</td>
      <td>0.0</td>
      <td>18 days</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>10691</td>
      <td>1997-10-03</td>
      <td>1997-11-14</td>
      <td>1997-10-22</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.30</td>
      <td>49.30</td>
      <td>48</td>
      <td>0.0</td>
      <td>2366.4</td>
      <td>0.0</td>
      <td>23 days</td>
    </tr>
    <tr>
      <th>2143</th>
      <td>10852</td>
      <td>1998-01-26</td>
      <td>1998-02-09</td>
      <td>1998-01-30</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.30</td>
      <td>49.30</td>
      <td>50</td>
      <td>0.0</td>
      <td>2465.0</td>
      <td>0.0</td>
      <td>10 days</td>
    </tr>
    <tr>
      <th>2147</th>
      <td>10904</td>
      <td>1998-02-24</td>
      <td>1998-03-24</td>
      <td>1998-02-27</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.30</td>
      <td>49.30</td>
      <td>35</td>
      <td>0.0</td>
      <td>1725.5</td>
      <td>0.0</td>
      <td>25 days</td>
    </tr>
    <tr>
      <th>2150</th>
      <td>10949</td>
      <td>1998-03-13</td>
      <td>1998-04-10</td>
      <td>1998-03-17</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.30</td>
      <td>49.30</td>
      <td>60</td>
      <td>0.0</td>
      <td>2958.0</td>
      <td>0.0</td>
      <td>24 days</td>
    </tr>
    <tr>
      <th>2152</th>
      <td>10988</td>
      <td>1998-03-31</td>
      <td>1998-04-28</td>
      <td>1998-04-10</td>
      <td>29</td>
      <td>Forts d'rables</td>
      <td>Chantal Goulet</td>
      <td>Accounting Manager</td>
      <td>148 rue Chasseur</td>
      <td>Ste-Hyacinthe</td>
      <td>Canada</td>
      <td>62</td>
      <td>Tarte au sucre</td>
      <td>49.30</td>
      <td>49.30</td>
      <td>40</td>
      <td>0.0</td>
      <td>1972.0</td>
      <td>0.0</td>
      <td>18 days</td>
    </tr>
  </tbody>
</table>
<p>161 rows × 20 columns</p>
</div>




```python
# Memvisualkan Outlier

plt.figure(figsize = (20,10))
sns.boxplot(tabel1['Total_Harga'])
plt.show()
```

    C:\Users\Dwi Pamuji\anaconda3\lib\site-packages\seaborn\_decorators.py:43: FutureWarning:
    
    Pass the following variable as a keyword arg: x. From version 0.12, the only valid positional argument will be `data`, and passing other arguments without an explicit keyword will result in an error or misinterpretation.
    
    


    
![png](output_56_1.png)
    


# **Menghitung total produk yang terjual dan total penjualan tiap supplier**

# **4. DATA VISUALIZATION DAN STATISTIK**

Setelah memiliki data yang sudah 'bersih' kita dapat melakukan analisis data dengan menggunakan visual sebagai medianya. Di sini, kita akan melakukan visualisasi data yang berfokus pada **Supplier** untuk mendapatkan beberapa insight yang kemudian dapat menjadi landasan untuk pengambilan keputusan dan menyusun strategi yang kuat untuk mendapatkan profit yang sebesar-besarnya dengan kerugian yang sekecil-kecilnya

## **4.1 Data Visualization**

### **4.1.1Total Penjualan per Tahun**


```python
# Tabel Penjualan perTahun

penjualan_tahunan = tabel1[['ShippedDate', 'Quantity' ,'Total_Harga']].groupby(pd.DatetimeIndex(tabel1['ShippedDate']).year).sum()
penjualan_tahunan
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
      <th>Total_Harga</th>
    </tr>
    <tr>
      <th>ShippedDate</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1996</th>
      <td>8717</td>
      <td>262798.43</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>25460</td>
      <td>691261.40</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>15942</td>
      <td>467863.72</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Line Plot Penjualan tahun ke tahun

plt.style.use('seaborn')
plt.figure(figsize=(20,10))
plt.plot(penjualan_tahunan.index, penjualan_tahunan['Total_Harga'], 'ro-')
plt.title('Penjualan Makanan dan Minuman 1996-1998', size = 20)
plt.xlabel('Tahun', size = 18)
plt.ylabel('Penjualan', size = 18)
plt.xticks(penjualan_tahunan.index, size = 20)
plt.yticks(rotation = 45, size = 20)

for x,y in zip(penjualan_tahunan.index, round(penjualan_tahunan['Total_Harga'],1)) :
    plt.annotate(y,
    (x,y),
    textcoords = 'offset pixels',
    xytext = (1,15), size=15)

plt.show()
```


    
![png](output_63_0.png)
    


dari linechart diatas dapat di lihat bahwa penjualan terbesar berada pada tahun 1997, Setelah mengetahui total penjualan kita akan menganalisa beberapa data yang berfokus kepada perusahaan supplier

### **4.1.2 TOP 5 SUPPLIER BERDASARKAN PENJUALAN**


```python
# Membuat Tabel Top 5 Supplier

tabel_penjualan = tabel1[['Supplier_CompanyName', 'Total_Harga']].groupby('Supplier_CompanyName').sum().sort_values('Total_Harga', ascending = False)
tabel_penjualan.rename(columns = ({'Total_Harga':'Total_Penjualan'}), inplace = True)
tabel_penjualan.reset_index(inplace=True)
```


```python
supplier_top_5 = tabel_penjualan.head()
```


```python
# Membuat Grafik Top 5 Supplier

fig = px.bar(supplier_top_5, x='Supplier_CompanyName', y='Total_Penjualan',
            title = 'TOP 5 SUPPLIER')
fig.show()
```


<div>                            <div id="a7ca30cd-a933-44bf-b87d-154963a24365" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("a7ca30cd-a933-44bf-b87d-154963a24365")) {                    Plotly.newPlot(                        "a7ca30cd-a933-44bf-b87d-154963a24365",                        [{"alignmentgroup":"True","hovertemplate":"Supplier_CompanyName=%{x}<br>Total_Penjualan=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","offsetgroup":"","orientation":"v","showlegend":false,"textposition":"auto","x":["Aux joyeux ecclsiastiques","Plutzer Lebensmittelgromrkte AG","Gai pturage","Pavlova, Ltd.","G'day, Mate"],"xaxis":"x","y":[178398.5,156091.79,133416.0,122376.4,72525.59999999999],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Supplier_CompanyName"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Total_Penjualan"}},"legend":{"tracegroupgap":0},"title":{"text":"TOP 5 SUPPLIER"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('a7ca30cd-a933-44bf-b87d-154963a24365');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


Bisnis import-eksport makanan dan minuman tak terlepas dari perusahaan supplier, peranan supplier akan sangat penting untuk menyediakan produk yang laku di pasaran dan dapat menaikan profit dari perusahaan. Mari sedikit melihat data yang ada. Dari total sekitar 2000an transaksi yang terjadi pada tahun 1996 hingga 1998. Supplier yang memiliki nilai penjualan terbesar adalah perusahaan Aux joyeux ecclsiastiques, Plutzer Lebensmitteigromrkte AG, Gai pturage, pavlova Ltd, G'day. mate. 4 dari perusahaan diatas memiliki penjualan di atas 100,000 dollar.

Melihat adanya sektor di mana terdapat supplier yang memiliki hasil penjualan yang tinggi dan menghasilkan penjualan yang besar untuk perusahaan. Dengan data diatas kita dapat mentreatment perusahaan - perusahaan supplier dengan lebih baik. Seperti memberi bonus kepada perusahaan supplier, dan kerja sama yang lebih besar lagi. Penawaran ini dilakukan agar perusahaan supplier tidak berpindah ke lain hati dan terjalin kerja sama yang lebih erat.

### **4.1.3 WORST 5 SUPPLIER BERDASARKAN PENJUALAN**


```python
# Membuat Tabel Worst 5 Supplier

supplier_worst_5 = tabel_penjualan.tail()
supplier_worst_5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Supplier_CompanyName</th>
      <th>Total_Penjualan</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>PB Knckebrd AB</td>
      <td>12510.0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Lyngbysild</td>
      <td>10970.0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Escargots Nouveaux</td>
      <td>7075.5</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Zaanse Snoepfabriek</td>
      <td>6367.0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Refrescos Americanas LTDA</td>
      <td>4927.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Membuat Grafik Worst 5 Supplier

fig = px.bar(supplier_worst_5, x='Supplier_CompanyName', y='Total_Penjualan',
            title = 'WORST 5 SUPPLIER')
fig.show()
```


<div>                            <div id="e1fbb57a-55d1-432a-bf02-287b9dc40f02" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("e1fbb57a-55d1-432a-bf02-287b9dc40f02")) {                    Plotly.newPlot(                        "e1fbb57a-55d1-432a-bf02-287b9dc40f02",                        [{"alignmentgroup":"True","hovertemplate":"Supplier_CompanyName=%{x}<br>Total_Penjualan=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","offsetgroup":"","orientation":"v","showlegend":false,"textposition":"auto","x":["PB Knckebrd AB","Lyngbysild","Escargots Nouveaux","Zaanse Snoepfabriek","Refrescos Americanas LTDA"],"xaxis":"x","y":[12510.0,10970.0,7075.5,6367.0,4927.5],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Supplier_CompanyName"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Total_Penjualan"}},"legend":{"tracegroupgap":0},"title":{"text":"WORST 5 SUPPLIER"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('e1fbb57a-55d1-432a-bf02-287b9dc40f02');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


Bisnis import-eksport makanan dan minuman tak terlepas dari perusahaan supplier, peranan supplier akan sangat penting untuk menyediakan produk yang laku di pasaran dan dapat menaikan profit dari perusahaan. Mari sedikit melihat data yang ada. Dari total sekitar 2000an transaksi yang terjadi pada tahun 1996 hingga 1998. Supplier yang memiliki nilai penjualan yang relatif kecil adalah perusahaan PB Knckebrd AB, Lyngbysild, Escargots Nouveaux, Zaanse Snoepfabriek, Refrescos Americanas LTDA.

Melihat adanya sektor di mana terdapat supplier yang memiliki hasil penjualan yang relatif kecil. Dengan data diatas kita dapat mentreatment perusahaan - perusahaan supplier dengan lebih baik. Seperti memberi pendekatan kepada perusahaan supplier agar supplier dapat menaikan penjualannya.

### **4.1.4 10 Product Terlaris**


```python
# Membuat Grafik 10 product terlaris

product_best_10 = tabel1[['ProductName', 'Quantity']].groupby('ProductName').sum().sort_values('Quantity', ascending=False).head(10)
plt.figure(figsize = (20,10))
sns.barplot(x = product_best_10['Quantity'], y=product_best_10.index)
plt.grid(axis = 'x')
plt.title('Product Quantities', size = 30)
plt.xticks(size = 15)
plt.yticks(size = 15)
plt.xlabel('Quantities', size = 25)
plt.ylabel('Product', size = 25)
plt.show()
```


    
![png](output_75_0.png)
    


Dengan data diatas kita dapat melihat 10 produk terlaris di pasaran. Setelah mengetahui 10 produk terlaris selanjutnya kita akan mencari top supplier dengan product terlaris untuk melihat supplier mana yang memiliki product terlaris

### **4.1.5 Top 3 Supplier dengan Product Terlaris**


```python
# Membuat Tabel 3 Supplier dengan product terlaris

tabel1[['Supplier_CompanyName', 'Quantity']].groupby('Supplier_CompanyName').sum().sort_values('Quantity', ascending=False).head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>Supplier_CompanyName</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pavlova, Ltd.</th>
      <td>3867</td>
    </tr>
    <tr>
      <th>Plutzer Lebensmittelgromrkte AG</th>
      <td>3808</td>
    </tr>
    <tr>
      <th>Gai pturage</th>
      <td>3000</td>
    </tr>
  </tbody>
</table>
</div>




```python
pavlova_product = tabel1[tabel1['Supplier_CompanyName'] == 'Pavlova, Ltd.'][['ProductName', 'Quantity']].groupby(by='ProductName').sum().sort_values(by='Quantity', ascending=False)
plutzer_product = tabel1[tabel1['Supplier_CompanyName'] == 'Plutzer Lebensmittelgromrkte AG'][['ProductName', 'Quantity']].groupby(by='ProductName').sum().sort_values(by='Quantity', ascending=False)
gai_product = tabel1[tabel1['Supplier_CompanyName'] == 'Gai pturage'][['ProductName', 'Quantity']].groupby(by='ProductName').sum()
display(pavlova_product, plutzer_product, gai_product)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>ProductName</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pavlova</th>
      <td>1112</td>
    </tr>
    <tr>
      <th>Alice Mutton</th>
      <td>966</td>
    </tr>
    <tr>
      <th>Outback Lager</th>
      <td>805</td>
    </tr>
    <tr>
      <th>Carnarvon Tigers</th>
      <td>539</td>
    </tr>
    <tr>
      <th>Vegie-spread</th>
      <td>445</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>ProductName</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rhnbru Klosterbier</th>
      <td>1151</td>
    </tr>
    <tr>
      <th>Original Frankfurter grne Soe</th>
      <td>761</td>
    </tr>
    <tr>
      <th>Thringer Rostbratwurst</th>
      <td>746</td>
    </tr>
    <tr>
      <th>Wimmers gute Semmelkndel</th>
      <td>608</td>
    </tr>
    <tr>
      <th>Rssle Sauerkraut</th>
      <td>542</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Quantity</th>
    </tr>
    <tr>
      <th>ProductName</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Camembert Pierrot</th>
      <td>1504</td>
    </tr>
    <tr>
      <th>Raclette Courdavault</th>
      <td>1496</td>
    </tr>
  </tbody>
</table>
</div>



```python
plt.figure(figsize = (15,8))
plt.subplot(3,1,1)
sns.barplot(x=pavlova_product['Quantity'],y=pavlova_product.index)
plt.title('Pavlova, Ltd. : Product Quantities', size = 30)
plt.xticks(size = 15)
plt.yticks(size = 15)
plt.xlabel('Quantities', size = 25)
plt.ylabel('Product', size = 25)


plt.subplot(3,1,2)
sns.barplot(x=plutzer_product['Quantity'],y=plutzer_product.index)
plt.title('Plutzer Lebensmittelgromrkte AG : Product Quantities', size = 30)
plt.xticks(size = 15)
plt.yticks(size = 15)
plt.xlabel('Quantities', size = 25)
plt.ylabel('Product', size = 25)

plt.subplot(3,1,3)
sns.barplot(x=gai_product['Quantity'],y=gai_product.index)
plt.title('Gai pturage : Product Quantities', size = 30)
plt.xticks(size = 15)
plt.yticks(size = 15)
plt.xlabel('Quantities', size = 25)
plt.ylabel('Product', size = 25)
plt.tight_layout(h_pad = 2)
```


    
![png](output_80_0.png)
    


Terlihat top 3 supplier dengan product terlaris adalah perusahaan Pavlova Ltd., Plutzer Lebensmittelgromrkte AG, dan Gai pturage. Dapat dilihat dari ketiga perusahaan tersebut perusahaan yang paling efisien adalah Gai pturage, karna kedua produknya adalah produk terlaris. perusahaan ```Gai pturage``` memproduksi ```camembert Pierrot``` yang menempati posisi pertama product terlaris dengan penjualan sebanyak 1504 pcs, dan ```Racietta Courdavault``` menempati posisi kedua produk terlaris sebanyak 1496 pcs.

Sedangkan 2 supplier lain hanya memiliki masing-masing 1 produk yang masuk kedalam 10 produk terlaris. yaitu pruduct ```pavlova``` yang di produksi oleh perusahaan ```Pavlova, Ltd.``` yang menduduki posisi 6 produk terlaris dengan penjualan sebanyak 1112 pcs. dan ```Rhnbru Klosterbier``` yang di produksi oleh perusahaan ```Plutzer Lebensmittelgromrkte AG``` yang menduduki posisi 5 produk terlaris dengan penjualan sebanyak 1151 pcs.

Melihat keadaan tersebut, kita dapat melakukan riset lebih lanjut apakah produk ```pavlova``` dan ```Plutzer Lebensmittelgromrkte AG``` memiliki demand yang melebihi jumlah produksi. Bila memang demand melibihi jumlah produksi kita dapat meminta pihak perusahaan ```Pavlova, Ltd.``` dapat menjadikan produk  ```pavlova``` menjadi produksi yang diprioritaskan dan ```Plutzer Lebensmittelgromrkte AG``` dapat menjadikan produk ```Rhnbru Klosterbier``` menjadi produksi yang diprioritaskan.

### **4.1.6 Top Supplier berdasarkan profit yang dihasilkan dari penjualan produknya**

Pada analisa ini kita akan menggukanan tabel2 dari Quary 2 untuk melihat perusahaan supplier mana yang menghasilkan profit untuk perusahaan


```python
supplier_profit = tabel2[['Supplier_CompanyName', 'Profit']].groupby(by='Supplier_CompanyName').sum().sort_values(by='Profit', ascending=False)
supplier_profit
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Profit</th>
    </tr>
    <tr>
      <th>Supplier_CompanyName</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aux joyeux ecclsiastiques</th>
      <td>15299.5000</td>
    </tr>
    <tr>
      <th>Plutzer Lebensmittelgromrkte AG</th>
      <td>9424.0400</td>
    </tr>
    <tr>
      <th>Gai pturage</th>
      <td>9316.0000</td>
    </tr>
    <tr>
      <th>Pavlova, Ltd.</th>
      <td>8441.0500</td>
    </tr>
    <tr>
      <th>G'day, Mate</th>
      <td>4503.0000</td>
    </tr>
    <tr>
      <th>Forts d'rables</th>
      <td>4310.7000</td>
    </tr>
    <tr>
      <th>Formaggi Fortini s.r.l.</th>
      <td>3932.8000</td>
    </tr>
    <tr>
      <th>Pasta Buttini s.r.l.</th>
      <td>3528.0000</td>
    </tr>
    <tr>
      <th>Norske Meierier</th>
      <td>3419.8000</td>
    </tr>
    <tr>
      <th>Specialty Biscuits, Ltd.</th>
      <td>3358.3000</td>
    </tr>
    <tr>
      <th>Leka Trading</th>
      <td>3191.6500</td>
    </tr>
    <tr>
      <th>Heli Swaren GmbH &amp; Co. KG</th>
      <td>3173.6900</td>
    </tr>
    <tr>
      <th>New Orleans Cajun Delights</th>
      <td>2721.6000</td>
    </tr>
    <tr>
      <th>Ma Maison</th>
      <td>2663.7500</td>
    </tr>
    <tr>
      <th>Exotic Liquids</th>
      <td>2350.2000</td>
    </tr>
    <tr>
      <th>Karkki Oy</th>
      <td>2072.7500</td>
    </tr>
    <tr>
      <th>Bigfoot Breweries</th>
      <td>1777.2000</td>
    </tr>
    <tr>
      <th>Grandma Kelly's Homestead</th>
      <td>1726.0000</td>
    </tr>
    <tr>
      <th>Tokyo Traders</th>
      <td>1653.8000</td>
    </tr>
    <tr>
      <th>New England Seafood Cannery</th>
      <td>1615.4500</td>
    </tr>
    <tr>
      <th>Svensk Sjfda AB</th>
      <td>1150.2000</td>
    </tr>
    <tr>
      <th>Cooperativa de Quesos 'Las Cabras'</th>
      <td>1129.2000</td>
    </tr>
    <tr>
      <th>Nord-Ost-Fisch Handelsgesellschaft mbH</th>
      <td>1069.1400</td>
    </tr>
    <tr>
      <th>Mayumi's</th>
      <td>951.7000</td>
    </tr>
    <tr>
      <th>Lyngbysild</th>
      <td>517.5000</td>
    </tr>
    <tr>
      <th>Zaanse Snoepfabriek</th>
      <td>465.6500</td>
    </tr>
    <tr>
      <th>PB Knckebrd AB</th>
      <td>455.4000</td>
    </tr>
    <tr>
      <th>Escargots Nouveaux</th>
      <td>410.7500</td>
    </tr>
    <tr>
      <th>Refrescos Americanas LTDA</th>
      <td>279.9000</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize = (20,10))
sns.barplot(y=supplier_profit['Profit'], x=supplier_profit.index)
plt.grid(axis = 'y')
plt.title('Keuntungan dari Produk Supplier', size = 30)
plt.xticks(size = 15, rotation = '90')
plt.yticks(size = 15)
plt.ylabel('Profit', size = 25)
plt.xlabel('Supplier', size = 25)
plt.show()
```


    
![png](output_85_0.png)
    


Dari data tersebut dapat terlihat jelas bahwa perusahaan supplier ```Aux joyeux ecclsiastiques``` menghasilkan profit terbesar pertama di antara supplier lain sebesar 15299.5 dollar. bahkan hampir mencapai 2x lipat dari supplier nomer 2 yaitu ```Plutzer Lebensmittelgromrkte AG```. dan perusahaan supplier yang menghasilkan profit terkecil adalah ```Refrescos Americanas LTDA``` yaitu sebesar 279.9 dollar.

dengan data tersebut kita dapat membuat keputusan yang tegas kepada perusahaan supplier yang memiliki profit yang relatif kecil dan dapat berfokus kepada perushaan yang menghasilkan profit yang relatif besar agar dapat menaikan keuntungan perusahaan

## **4.2 STATISTIC**

### **4.2.1 Perbedaan Punjualan Produk(```Total_Harga```) Tiap Supplier**

Karna feature ```Total_Harga``` diketahui tidak berdistribusi normal dari Processing Data, tetapi kita akan tetap menguji distribusinya menggunakan metode shapiro

Maka, kita akan menguji median dari gaji, karena mean sensitif terhadap outlier.


```python
norm_total_harga, pval_total_harga = shapiro(tabel1['Total_Harga'])

if pval_total_harga < 0.05 :
    print (f'Tolak H0 Karena P-Value ({pval_total_harga} < 5%)')
    print ('DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({pval_total_harga} > 5%)')
    print ('DATA PENJUALAN BERDISTRIBUS NORMAL')
```

    Tolak H0 Karena P-Value (0.0 < 5%)
    DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL
    


```python
# Uji Perbandingan Nilai Penjualan (Total_Harga) Antar Setiap Supllier (Kruskal Wallis)

kw, p_value_kw = kruskal(
    tabel1[tabel1['Supplier_CompanyName'] == 'Aux joyeux ecclsiastiques']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Bigfoot Breweries']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == "Cooperativa de Quesos 'Las Cabras'"]['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Escargots Nouveaux']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Exotic Liquids']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Formaggi Fortini s.r.l.']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == "Forts d'rables"]['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == "G'day, Mate"]['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Gai pturage']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == "Grandma Kelly's Homestead"]['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Heli Swaren GmbH & Co. KG']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Karkki Oy']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Leka Trading']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Lyngbysild']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Ma Maison']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == "Mayumi's"]['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'New England Seafood Cannery']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'New Orleans Cajun Delights']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Nord-Ost-Fisch Handelsgesellschaft mbH']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Norske Meierier']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'PB Knckebrd AB']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Pasta Buttini s.r.l.']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Pavlova, Ltd.']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Plutzer Lebensmittelgromrkte AG']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Refrescos Americanas LTDA']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Specialty Biscuits, Ltd.']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Svensk Sjfda AB']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Tokyo Traders']['Total_Harga'],
    tabel1[tabel1['Supplier_CompanyName'] == 'Zaanse Snoepfabriek']['Total_Harga'])

if p_value_kw < 0.05 :
    print (f'Tolak H0 Karena P-Value ({p_value_kw} < 0.05)')
    print ('Terdapat Perbedaan Nilai Median Total Penjualan pada Setiap Supplier')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({p_value_kw} > 0.05)')
    print ('Tidak Terdapat Perbedaan Nilai Median Total Penjualan pada Setiap Supplier')
```

    Tolak H0 Karena P-Value (3.5987357936246638e-65 < 0.05)
    Terdapat Perbedaan Nilai Median Total Penjualan pada Setiap Supplier
    

### **4.2.2 Distribusi Feature**

untuk menaikan keuntungan perusahan sebasar-besarnya kita harus menaikan feature yang berhubungan kuat dengan ```Profit``` 
kita akan mencari feature mana yang miliki korelasi yang paling besar dengan profit supplier


```python
x1 = tabel1[['Supplier_CompanyName', 'Quantity', 'Total_Harga', 'Profit', ]].groupby('Supplier_CompanyName').sum()
```


```python
x2 = tabel1[['Supplier_CompanyName', 'ProductName']].groupby('Supplier_CompanyName').count()
```


```python
supplier_kor = x2.join(x1, how='outer')
supplier_kor.rename(columns = {'ProductName':'Total_Produk', 'Quantity':'Total_Quantity', 'Total_Harga':'Total_Penjualan'}, inplace=True)
supplier_kor.sample(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total_Produk</th>
      <th>Total_Quantity</th>
      <th>Total_Penjualan</th>
      <th>Profit</th>
    </tr>
    <tr>
      <th>Supplier_CompanyName</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Ma Maison</th>
      <td>67</td>
      <td>1636</td>
      <td>27099.75</td>
      <td>2663.75</td>
    </tr>
    <tr>
      <th>PB Knckebrd AB</th>
      <td>33</td>
      <td>926</td>
      <td>12510.00</td>
      <td>455.40</td>
    </tr>
    <tr>
      <th>Heli Swaren GmbH &amp; Co. KG</th>
      <td>59</td>
      <td>1436</td>
      <td>43991.69</td>
      <td>3173.69</td>
    </tr>
    <tr>
      <th>Nord-Ost-Fisch Handelsgesellschaft mbH</th>
      <td>31</td>
      <td>608</td>
      <td>15741.12</td>
      <td>1069.14</td>
    </tr>
    <tr>
      <th>Norske Meierier</th>
      <td>102</td>
      <td>2480</td>
      <td>49803.00</td>
      <td>3419.80</td>
    </tr>
  </tbody>
</table>
</div>




```python
supplier_kor.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 29 entries, Aux joyeux ecclsiastiques to Zaanse Snoepfabriek
    Data columns (total 4 columns):
     #   Column           Non-Null Count  Dtype  
    ---  ------           --------------  -----  
     0   Total_Produk     29 non-null     int64  
     1   Total_Quantity   29 non-null     int64  
     2   Total_Penjualan  29 non-null     float64
     3   Profit           29 non-null     float64
    dtypes: float64(2), int64(2)
    memory usage: 2.2+ KB
    


```python
# Uji Perbandingan Nilai profitEach Antar Setiap Product Line (Normalitas)

norm_produk, pval_produk = shapiro(supplier_kor['Total_Produk'])
norm_quantity, pval_quantity = shapiro(supplier_kor['Total_Quantity'])
norm_penjualan, pval_penjualan = shapiro(supplier_kor['Total_Penjualan'])
norm_profit, pval_profit = shapiro(supplier_kor['Profit'])

print ('DATA TOTAL PRODUK')
if pval_penjualan < 0.05 :
    print (f'Tolak H0 Karena P-Value ({pval_produk} < 5%)')
    print ('DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({pval_produk} > 5%)')
    print ('DATA PENJUALAN BERDISTRIBUS NORMAL')
    
print ('\nDATA TOTAL QUANTITY')
if pval_penjualan < 0.05 :
    print (f'Tolak H0 Karena P-Value ({pval_quantity} < 5%)')
    print ('DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({pval_quantity} > 5%)')
    print ('DATA PENJUALAN BERDISTRIBUS NORMAL')
    
print ('\nDATA TOTAL PENJUALAN')
if pval_penjualan < 0.05 :
    print (f'Tolak H0 Karena P-Value ({pval_penjualan} < 5%)')
    print ('DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({pval_penjualan} > 5%)')
    print ('DATA PENJUALAN BERDISTRIBUS NORMAL')

print ('\nDATA PROFIT')
if norm_profit < 0.05 :
    print (f'Tolak H0 Karena P-Value ({norm_profit} < 5%)')
    print ('DATA PROFIT TIDAK BERDISTRIBUS NORMAL')
else :
    print (f'Gagal Tolak H0 Karena P-Value ({norm_profit} > 5%)')
    print ('DATA PROFIT BERDISTRIBUS NORMAL')
```

    DATA TOTAL PRODUK
    Tolak H0 Karena P-Value (0.02576380781829357 < 5%)
    DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL
    
    DATA TOTAL QUANTITY
    Tolak H0 Karena P-Value (0.025382932275533676 < 5%)
    DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL
    
    DATA TOTAL PENJUALAN
    Tolak H0 Karena P-Value (7.248014298966154e-05 < 5%)
    DATA PENJUALAN TIDAK BERDISTRIBUS NORMAL
    
    DATA PROFIT
    Gagal Tolak H0 Karena P-Value (0.7543554902076721 > 5%)
    DATA PROFIT BERDISTRIBUS NORMAL
    

### **4.2.3 Korelasi antar feature**

di ketahui dari 4 feature 
terdapat 3 feature yang tidak berdistribusi normal yaitu ```Total_Produk```, ```Total_Quantity```, dan ```Total_Penjualan```
hanya feature ```Profit``` yang berdistribusi normal

dari data tersebut kita dapat mengambil kesimpulan untuk menggunakan metode correlasi non parametrik yaitu Correlasi **Spearman**


```python
# Hubungan antar kolom

supplier_kor[['Total_Produk', 'Total_Quantity', 'Total_Penjualan', 'Profit']].corr('spearman')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total_Produk</th>
      <th>Total_Quantity</th>
      <th>Total_Penjualan</th>
      <th>Profit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Total_Produk</th>
      <td>1.000000</td>
      <td>0.980774</td>
      <td>0.770274</td>
      <td>0.797387</td>
    </tr>
    <tr>
      <th>Total_Quantity</th>
      <td>0.980774</td>
      <td>1.000000</td>
      <td>0.794581</td>
      <td>0.818719</td>
    </tr>
    <tr>
      <th>Total_Penjualan</th>
      <td>0.770274</td>
      <td>0.794581</td>
      <td>1.000000</td>
      <td>0.966995</td>
    </tr>
    <tr>
      <th>Profit</th>
      <td>0.797387</td>
      <td>0.818719</td>
      <td>0.966995</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize = (12,4))
sns.heatmap(supplier_kor[['Total_Produk', 'Total_Quantity','Total_Penjualan', 'Profit']].corr(), annot=True)
plt.show()
```


    
![png](output_98_0.png)
    


Dari tabel heatmap di atas kita mendapat kesimpulan bahwa feature yang memiliki nilai korelasi terbesar dengan feature ```Profit``` adalah feature ```Total_Penjualan```


```python
plt.figure(figsize=(15,10))
sns.scatterplot(supplier_kor['Total_Penjualan'], supplier_kor['Profit'])
plt.show()
```

    C:\Users\Dwi Pamuji\anaconda3\lib\site-packages\seaborn\_decorators.py:43: FutureWarning:
    
    Pass the following variables as keyword args: x, y. From version 0.12, the only valid positional argument will be `data`, and passing other arguments without an explicit keyword will result in an error or misinterpretation.
    
    


    
![png](output_100_1.png)
    


scatterplot di atas menginfomasikan bahwa **Total Penjualan Produk Supplier** memiliki hubungan yang kuat dengan **Profit Produk Supplier** yang di hasilkan oleh masing masing supplier

insight :
Untuk menaikan **Profit Supplier** yang di hasilkan perusahaan kita perlu meningkatkan **Total_Penjualan** tiap Supplier

Dari grafik heatmap dan scatterplot di atas di meninfomasikan bahwa **Total Penjualan Produk Supplier** memiliki hubungan yang kuat dengan **Profit Produk Supplier** yang di hasilkan oleh masing masing supplier
