# POS KopiTiam Project

POS KopiTiam adalah sistem Point of Sale (POS) sederhana untuk kafe KopiTiam, dibangun menggunakan bahasa pemrograman Go. Sistem ini menyediakan API RESTful untuk mengelola pesanan, data master, dan laporan penjualan.

## Fitur Utama

### 1. Manajemen Pesanan (Orders)
- **Buat Pesanan Baru**: Membuat pesanan dengan daftar item menu, quantity, dan otomatis menghitung total harga.
- **Lihat Semua Pesanan**: Mengambil daftar semua pesanan yang ada.
- **Lihat Pesanan Berdasarkan ID**: Mengambil detail pesanan tertentu berdasarkan ID unik.
- **Laporan Pesanan**: Menampilkan laporan penjualan dengan detail item per pesanan.

### 2. Data Master
- **Menu**: Daftar menu makanan dan minuman dengan harga dan stok harian.
- **Kategori**: Kategori untuk mengelompokkan menu (misalnya: Makanan, Minuman).
- **Karyawan**: Daftar karyawan yang dapat ditugaskan untuk pesanan.

### 3. Fitur Tambahan
- **Health Check**: Endpoint untuk memeriksa status kesehatan API.
- **Database Migration**: Migrasi skema database otomatis.
- **Seeding Data**: Pengisian data awal untuk development (hanya di environment dev).
- **Validasi Stok**: Memastikan stok harian cukup sebelum membuat pesanan.
- **Transaksi Database**: Menggunakan transaksi untuk memastikan konsistensi data saat membuat pesanan.

## Teknologi yang Digunakan

- **Bahasa Pemrograman**: Go 1.21
- **Web Framework**: Gin
- **ORM**: GORM
- **Database**: MySQL
- **UUID**: Untuk generate ID unik pesanan
- **Environment Config**: Menggunakan .env file

## Struktur Proyek

```
pos_kopitiam_db_baru/
├── cmd/
│   ├── api/v1/
│   │   └── main.go          # Entry point API
│   └── db/
│       └── runMigration.go  # Script migrasi database
├── internal/
│   ├── config/
│   │   ├── config.go        # Konfigurasi aplikasi
│   │   ├── database.go      # Koneksi database
│   │   ├── db_logic.go      # Logika database
│   │   ├── fk_constraint.go # Foreign key constraints
│   │   └── seeder.go        # Data seeding
│   ├── dto/
│   │   └── report.go        # Data Transfer Objects
│   ├── models/
│   │   └── domain.go        # Model domain
│   └── service/
│       └── order_service.go # Business logic untuk order
├── go.mod                   # Go module file
├── pos_kopitiam_full.sql    # Schema database
└── README.md                # Dokumentasi ini
```

## Setup dan Instalasi

### Prasyarat
- Go 1.21 atau lebih baru
- MySQL Server
- Git

### Langkah Instalasi

1. **Clone Repository**
   ```bash
   git clone https://github.com/Lapnes/pos-kopitiam.git
   cd pos-kopitiam
   ```

2. **Install Dependencies**
   ```bash
   go mod tidy
   ```

3. **Setup Database**
   - Buat database MySQL baru
   - Import schema dari `pos_kopitiam_full.sql`
   - Atau jalankan migrasi:
     ```bash
     go run cmd/db/runMigration.go
     ```

4. **Konfigurasi Environment**
   Buat file `.env` di root directory:
   ```env
   DB_HOST=localhost
   DB_PORT=3306
   DB_USER=your_username
   DB_PASSWORD=your_password
   DB_NAME=pos_kopitiam
   APP_ENV=dev  # atau prod
   ```

5. **Jalankan Aplikasi**
   ```bash
   go run cmd/api/v1/main.go
   ```

   API akan berjalan di `http://localhost:8080`

## Dokumentasi API

Base URL: `http://localhost:8080`

### Health Check

#### GET /health
Memeriksa status kesehatan API.

**Response:**
```json
{
  "status": "ok",
  "message": "POS KopiTiam API is running"
}
```

### Orders

#### POST /api/v1/orders
Membuat pesanan baru.

**Request Body:**
```json
{
  "employee_id": 1,
  "items": [
    {
      "menu_id": 1,
      "qty": 2
    },
    {
      "menu_id": 2,
      "qty": 1
    }
  ]
}
```

**Response:**
```json
{
  "order_id": "550e8400-e29b-41d4-a716-446655440000",
  "employee_id": 1,
  "total_price": 50000,
  "items": [
    {
      "menu_id": 1,
      "menu_name": "Nasi Goreng",
      "quantity": 2,
      "unit_price": 20000,
      "subtotal": 40000
    },
    {
      "menu_id": 2,
      "menu_name": "Teh Manis",
      "quantity": 1,
      "unit_price": 10000,
      "subtotal": 10000
    }
  ]
}
```

#### GET /api/v1/orders
Mengambil semua pesanan.

**Response:**
```json
[
  {
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "employee_id": 1,
    "total_price": 50000,
    "employee": {
      "employee_id": 1,
      "employee_name": "John Doe",
      "phone_number": "08123456789"
    },
    "order_details": [
      {
        "order_detail_id": "550e8400-e29b-41d4-a716-446655440001",
        "menu_id": 1,
        "quantity": 2,
        "unit_price": 20000,
        "subtotal": 40000,
        "menu": {
          "menu_id": 1,
          "menu_name": "Nasi Goreng",
          "price": 20000,
          "daily_stock": 48
        }
      }
    ]
  }
]
```

#### GET /api/v1/orders/{id}
Mengambil pesanan berdasarkan ID.

**Response:** Sama dengan response GET /api/v1/orders untuk satu item.

#### GET /api/v1/orders/report
Mengambil laporan penjualan.

**Response:**
```json
[
  {
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "employee_name": "John Doe",
    "menu_name": "Nasi Goreng",
    "quantity": 2,
    "unit_price": 20000,
    "subtotal": 40000,
    "total_price": 50000
  }
]
```

### Master Data

#### GET /api/v1/menus
Mengambil semua menu.

**Response:**
```json
[
  {
    "menu_id": 1,
    "category_id": 1,
    "menu_name": "Nasi Goreng",
    "price": 20000,
    "daily_stock": 50,
    "category": {
      "category_id": 1,
      "category_name": "Makanan"
    }
  }
]
```

#### GET /api/v1/categories
Mengambil semua kategori.

**Response:**
```json
[
  {
    "category_id": 1,
    "category_name": "Makanan",
    "menus": [
      {
        "menu_id": 1,
        "menu_name": "Nasi Goreng",
        "price": 20000,
        "daily_stock": 50
      }
    ]
  }
]
```

#### GET /api/v1/employees
Mengambil semua karyawan.

**Response:**
```json
[
  {
    "employee_id": 1,
    "employee_name": "John Doe",
    "phone_number": "08123456789"
  }
]
```

## Model Data

### Category
```json
{
  "category_id": 1,
  "category_name": "Makanan"
}
```

### Menu
```json
{
  "menu_id": 1,
  "category_id": 1,
  "menu_name": "Nasi Goreng",
  "price": 20000,
  "daily_stock": 50
}
```

### Employee
```json
{
  "employee_id": 1,
  "employee_name": "John Doe",
  "phone_number": "08123456789"
}
```

### Order
```json
{
  "order_id": "550e8400-e29b-41d4-a716-446655440000",
  "employee_id": 1,
  "total_price": 50000
}
```

### OrderDetail
```json
{
  "order_detail_id": "550e8400-e29b-41d4-a716-446655440001",
  "order_id": "550e8400-e29b-41d4-a716-446655440000",
  "menu_id": 1,
  "quantity": 2,
  "unit_price": 20000,
  "subtotal": 40000
}
```

## Kontribusi

1. Fork repository
2. Buat branch fitur baru (`git checkout -b feature/AmazingFeature`)
3. Commit perubahan (`git commit -m 'Add some AmazingFeature'`)
4. Push ke branch (`git push origin feature/AmazingFeature`)
5. Buat Pull Request

## Lisensi

Distributed under the MIT License. See `LICENSE` for more information.