# Panduan Pembuatan Backend REST API untuk Aplikasi Blog "Median"

Dalam tutorial ini, Anda akan belajar bagaimana membangun backend REST API untuk aplikasi blog bernama "Median" (klon sederhana Medium). Anda akan memulai dengan membuat proyek NestJS baru. Selanjutnya, Anda akan memulai server PostgreSQL Anda sendiri dan menghubungkannya menggunakan Prisma. Terakhir, Anda akan membangun REST API dan mendokumentasikannya dengan Swagger.
![Alt text](.images/image.png)

### Teknologi yang Akan Digunakan
Anda akan menggunakan berbagai alat berikut untuk membangun aplikasi ini:
1. **NestJS** sebagai kerangka kerja backend.
2. **Prisma** sebagai Object-Relational Mapper (ORM).
3. **PostgreSQL** sebagai database.
4. **Swagger** sebagai alat dokumentasi API.
5. **TypeScript** sebagai bahasa pemrograman.

## Prasyarat
**Pengetahuan yang Dianggap**
Ini adalah tutorial yang cocok untuk pemula. Namun, tutorial ini mengasumsikan:

- Pengetahuan dasar tentang JavaScript atau TypeScript (disarankan).
- Pengetahuan dasar tentang NestJS.

## Lingkungan Pengembangan
Untuk mengikuti tutorial ini, Anda diharapkan memiliki:

- **Node.js** terinstal.
- **Docker** atau **PostgreSQL** terinstal.
- **Ekstensi VSCode Prisma** terinstal (opsional).
- Akses ke shell Unix (seperti terminal dalam Linux dan macOS) untuk menjalankan perintah yang diberikan dalam seri ini (opsional).

## Membuat Proyek NestJS
Langkah pertama yang perlu Anda lakukan adalah menginstal CLI NestJS. CLI NestJS sangat berguna ketika Anda bekerja dengan proyek NestJS. Ini dilengkapi dengan utilitas bawaan yang membantu Anda menginisialisasi, mengembangkan, dan merawat aplikasi NestJS Anda.

Anda dapat menggunakan CLI NestJS untuk membuat proyek kosong. Untuk memulai, jalankan perintah berikut di lokasi di mana Anda ingin meletakkan proyek:

```bash
npx @nestjs/cli new median
```

CLI akan meminta Anda memilih manajer paket untuk proyek Anda - pilih npm. Setelah itu, Anda akan memiliki proyek NestJS baru di direktori saat ini.

## Buka Proyek dalam Editor Kode Pilihan Anda (Kami Menyarankan VSCode)
Setelah Anda telah membuat proyek NestJS, buka proyek ini dalam editor kode pilihan Anda (kami merekomendasikan VSCode). Anda akan melihat berkas-berkas berikut:

```
median
  ├── node_modules
  ├── src
  │   ├── app.controller.spec.ts
  │   ├── app.controller.ts
  │   ├── app.module.ts
  │   ├── app.service.ts
  │   └── main.ts
  ├── test
  │   ├── app.e2e-spec.ts
  │   └── jest-e2e.json
  ├── README.md
  ├── nest-cli.json
  ├── package-lock.json
  ├── package.json
  ├── tsconfig.build.json
  └── tsconfig.json
```

Ini adalah struktur proyek dasar NestJS yang baru saja Anda buat. Anda akan bekerja dengan berkas-berkas ini selama tutorial ini.

Sebagian besar kode yang akan Anda kerjakan akan berada dalam direktori `src`. CLI NestJS telah membuat beberapa berkas untuk Anda. Beberapa yang menonjol adalah:

- `src/app.module.ts`: Modul utama dari aplikasi.
- `src/app.controller.ts`: Kontroler dasar dengan satu rute: `/`. Rute ini akan mengembalikan pesan 'Hello World!' sederhana.

Anda dapat memulai proyek Anda dengan menggunakan perintah berikut:

```bash
npm run start:dev
```

Perintah ini akan memantau berkas Anda, secara otomatis mengkompilasi ulang dan memuat ulang server setiap kali Anda membuat perubahan. Untuk memverifikasi bahwa server berjalan, buka URL http://localhost:3000/. Anda seharusnya melihat halaman kosong dengan pesan 'Hello World!'.

## Membuat Instansi PostgreSQL
Anda akan menggunakan PostgreSQL sebagai database untuk aplikasi NestJS Anda. Tutorial ini akan menunjukkan cara menginstal dan menjalankan PostgreSQL di mesin Anda melalui kontainer Docker.

**Langkah 1: Buat Berkas `docker-compose.yml`**
Pertama, buat berkas `docker-compose.yml` di direktori utama proyek Anda:

```bash
touch docker-compose.yml
```

## Berkas Konfigurasi `docker-compose.yml`

Berikut adalah contoh berkas konfigurasi `docker-compose.yml` yang akan digunakan untuk menjalankan kontainer Docker PostgreSQL:

```yaml
# docker-compose.yml

version: '3.8'
services:

  postgres:
    image: postgres:13.5
    restart: always
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'

volumes:
  postgres:
```

Beberapa hal yang perlu dipahami tentang konfigurasi ini:

- Opsi `image` mendefinisikan gambar Docker yang akan digunakan. Di sini, Anda menggunakan gambar postgres versi 13.5.

- Opsi `environment` menentukan variabel lingkungan yang akan disampaikan ke kontainer selama inisialisasi. Anda dapat menentukan opsi konfigurasi dan rahasia, seperti nama pengguna dan kata sandi, yang akan digunakan oleh kontainer di sini.

- Opsi `volumes` digunakan untuk menyimpan data dalam sistem berkas host agar persisten.

- Opsi `ports` memetakan port dari mesin host ke kontainer. Formatnya mengikuti konvensi 'host_port:container_port'. Dalam hal ini, Anda memetakan port 5432 mesin host ke port 5432 kontainer PostgreSQL. Port 5432 adalah port yang umumnya digunakan oleh PostgreSQL.

Pastikan tidak ada yang berjalan pada port 5432 mesin Anda. Untuk menjalankan kontainer PostgreSQL, buka jendela terminal baru dan jalankan perintah berikut di folder utama proyek Anda:

```bash
docker-compose up
```

Jika semuanya berjalan dengan benar, jendela terminal baru akan menampilkan log bahwa sistem basis data siap menerima koneksi. Anda akan melihat log serupa dengan yang berikut di jendela terminal:

```
...
postgres_1  | 2022-03-05 12:47:02.410 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres_1  | 2022-03-05 12:47:02.410 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres_1  | 2022-03-05 12:47:02.411 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres_1  | 2022-03-05 12:47:02.419 UTC [1] LOG:  database system is ready to accept connections
```

Selamat 🎉. Anda sekarang memiliki basis data PostgreSQL sendiri untuk dijelajahi! Dengan basis data ini, Anda siap untuk menghubungkannya dengan aplikasi NestJS Anda menggunakan Prisma. Jika Anda memiliki pertanyaan lebih lanjut atau memerlukan bantuan lebih lanjut, jangan ragu untuk bertanya. Selamat bersenang-senang dalam pengembangan!

## Menyiapkan Prisma
Sekarang bahwa basis data sudah siap, saatnya menyiapkan Prisma!

**Inisialisasi Prisma**
Untuk memulai, instal Prisma CLI sebagai dependensi pengembangan. Prisma CLI akan memungkinkan Anda menjalankan berbagai perintah dan berinteraksi dengan proyek Anda.

```bash
npm install -D prisma
```

Anda dapat menginisialisasi Prisma di dalam proyek Anda dengan menjalankan:

```bash
npx prisma init
```

Perintah ini akan membuat direktori `prisma` baru dengan berkas `schema.prisma`. Berkas `schema.prisma` ini adalah berkas konfigurasi utama yang berisi skema basis data Anda. Juga, perintah ini akan membuat berkas `.env` di dalam proyek Anda.

## Mengatur Variabel Lingkungan
Di dalam berkas `.env`, Anda akan melihat variabel lingkungan `DATABASE_URL` dengan string koneksi dummy. Gantilah string koneksi ini dengan yang sesuai dengan instansi PostgreSQL Anda:

```plaintext
// .env
DATABASE_URL="postgres://myuser:mypassword@localhost:5432/median-db"
```

## Memahami Skema Prisma
Jika Anda membuka berkas `prisma/schema.prisma`, Anda akan melihat skema default berikut:

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

Berkas ini ditulis dalam Bahasa Skema Prisma, yang merupakan bahasa yang digunakan oleh Prisma untuk mendefinisikan skema basis data Anda. Berkas `schema.prisma` memiliki tiga komponen utama:

1. **Data source**: Menentukan koneksi basis data Anda. Konfigurasi di atas berarti penyedia basis data Anda adalah PostgreSQL dan string koneksi basis data tersedia dalam variabel lingkungan `DATABASE_URL`.

2. **Generator**: Menunjukkan bahwa Anda ingin menghasilkan Prisma Client, yaitu pustaka pembangun kueri yang aman tipe untuk basis data Anda. Ini digunakan untuk mengirim kueri ke basis data Anda.

3. **Model data**: Mendefinisikan model basis data Anda. Setiap model akan dipetakan ke tabel dalam basis data yang mendasarinya. Saat ini belum ada model dalam skema Anda, Anda akan menjelajahi bagian ini pada bagian berikutnya.

## Memodelkan Data
Sekarang saatnya untuk mendefinisikan model data untuk aplikasi Anda. Untuk tutorial ini, Anda hanya akan memerlukan model `Article` untuk mewakili setiap artikel di blog.

Di dalam berkas `prisma/schema.prisma`, tambahkan model baru ke skema Anda dengan nama `Article`:

```prisma
// prisma/schema.prisma

model Article {
  id          Int      @id @default(autoincrement())
  title       String   @unique
  description String?
  body        String
  published   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

Di sini, Anda telah membuat model `Article` dengan beberapa field. Setiap field memiliki nama (id, title, dll.), tipe (Int, String, dll.), dan atribut opsional lainnya (@id, @unique, dll.). Field-field dapat diubah menjadi opsional dengan menambahkan tanda tanya (?) setelah tipe field.

Field `id` memiliki atribut khusus bernama @id. Atribut ini mengindikasikan bahwa field ini adalah kunci utama dari model. Atribut @default(autoincrement()) mengindikasikan bahwa field ini harus diinkrementasi secara otomatis dan diberikan ke setiap catatan yang baru dibuat.

Field `published` adalah tanda untuk mengindikasikan apakah sebuah artikel sudah dipublikasikan atau masih dalam mode draf. Atribut @default(false) mengindikasikan bahwa field ini harus diatur ke false secara default.

Dua field DateTime, yaitu `createdAt` dan `updatedAt`, akan melacak kapan sebuah artikel dibuat dan kapan terakhir kali diubah. Atribut @updatedAt akan secara otomatis memperbarui field dengan timestamp saat ini setiap kali artikel dimodifikasi.

## Migrasi Basis Data
Dengan skema Prisma yang telah didefinisikan, Anda akan menjalankan migrasi untuk membuat tabel sebenarnya dalam basis data. Untuk menghasilkan dan mengeksekusi migrasi pertama Anda, jalankan perintah berikut di terminal:

```bash
npx prisma migrate dev --name "init"
```

Perintah ini akan melakukan tiga hal:

1. Menyimpan migrasi: Prisma Migrate akan mengambil snapshot skema Anda dan menentukan perintah SQL yang diperlukan untuk menjalankan migrasi tersebut. Prisma akan menyimpan berkas migrasi yang berisi perintah SQL ke dalam direktori `prisma/migrations` yang baru saja dibuat.

2. Mengeksekusi migrasi: Prisma Migrate akan mengeksekusi SQL dalam berkas migrasi untuk membuat tabel-tabel yang mendasari dalam basis data Anda.

3. Menghasilkan Prisma Client: Prisma akan menghasilkan Prisma Client berdasarkan skema terbaru Anda. Karena Anda belum memiliki pustaka Client terpasang, CLI akan memasangnya untuk Anda juga. Anda akan melihat paket @prisma/client dalam dependensi dalam berkas package.json Anda. Prisma Client adalah pembangun kueri TypeScript yang dihasilkan otomatis dari skema Prisma Anda. Ini disesuaikan dengan skema Prisma Anda dan akan digunakan untuk mengirim kueri ke basis data.

Jika berhasil diselesaikan, Anda akan melihat pesan seperti ini:

```
The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20220528101323_init/
    └─ migration.sql

Your database is now in sync with your schema.
...
✔ Generated Prisma Client (3.14.0 | library) to ./node_modules/@prisma/client in 31ms
```

Basis data Anda sekarang telah disinkronkan dengan skema Anda. Anda telah berhasil membuat tabel-tabel yang sesuai dengan model `Article` yang telah Anda tentukan dalam skema Prisma Anda.

Juga, perintah ini akan menghasilkan Prisma Client yang sesuai dengan skema terbaru Anda. Anda dapat menggunakannya untuk mengirim kueri ke basis data dengan cara yang aman tipe. Prisma Client ini akan ditempatkan dalam direktori `node_modules/@prisma/client` dan siap digunakan dalam proyek Anda.

Jika berhasil diselesaikan, Anda akan melihat pesan seperti ini:

```
The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20220528101323_init/
    └─ migration.sql

Your database is now in sync with your schema.
...
✔ Generated Prisma Client (3.14.0 | library) to ./node_modules/@prisma/client in 31ms
```

Anda dapat memeriksa berkas migrasi yang dihasilkan untuk mendapatkan gambaran tentang apa yang Prisma Migrate lakukan di belakang layar. Berikut adalah isi dari berkas migrasi tersebut (dalam kasus ini, `migration.sql`):

Anda dapat memeriksa berkas migrasi yang dihasilkan untuk mendapatkan gambaran tentang apa yang Prisma Migrate lakukan di belakang layar. Berikut adalah isi dari berkas migrasi tersebut (dalam kasus ini, `migration.sql`):

```sql
-- prisma/migrations/20220528101323_init/migration.sql

-- CreateTable
CREATE TABLE "Article" (
    "id" SERIAL NOT NULL,
    "title" TEXT NOT NULL,
    "description" TEXT,
    "body" TEXT NOT NULL,
    "published" BOOLEAN NOT NULL DEFAULT false,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "Article_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "Article_title_key" ON "Article"("title");
```

Ini adalah perintah SQL yang dihasilkan oleh Prisma Migrate untuk membuat tabel `Article` di dalam basis data PostgreSQL Anda.

Menanam Basis Data
Saat ini, basis data Anda kosong. Jadi Anda akan membuat sebuah skrip penanaman yang akan mengisi basis data dengan beberapa data palsu.

Pertama, buat berkas penanaman (seed) yang disebut prisma/seed.ts. Berkas ini akan berisi data palsu dan kueri-kueri yang diperlukan untuk menanamkan basis data Anda.

```bash
touch prisma/seed.ts
```

Kemudian, di dalam berkas penanaman, tambahkan kode berikut:

```typescript
// prisma/seed.ts

import { PrismaClient } from '@prisma/client';

// Inisialisasi Prisma Client
const prisma = new PrismaClient();

async function main() {
  // Membuat dua artikel palsu
  const post1 = await prisma.article.upsert({
    where: { title: 'Prisma Adds Support for MongoDB' },
    update: {},
    create: {
      title: 'Prisma Adds Support for MongoDB',
      body: 'Support for MongoDB has been one of the most requested features since the initial release of...',
      description:
        "We are excited to share that today's Prisma ORM release adds stable support for MongoDB!",
      published: false,
    },
  });

  const post2 = await prisma.article.upsert({
    where: { title: "What's new in Prisma? (Q1/22)" },
    update: {},
    create: {
      title: "What's new in Prisma? (Q1/22)",
      body: 'Our engineers have been working hard, issuing new releases with many improvements...',
      description:
        'Learn about everything in the Prisma ecosystem and community from January to March 2022.',
      published: true,
    },
  });

  console.log({ post1, post2 });
}

// Eksekusi fungsi utama
main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    // Tutup Prisma Client pada akhirnya
    await prisma.$disconnect();
  });
```

Di dalam skrip ini, Anda pertama-tama menginisialisasi Prisma Client. Kemudian, Anda membuat dua artikel menggunakan fungsi prisma.upsert(). Fungsi upsert hanya akan membuat artikel baru jika tidak ada artikel yang sesuai dengan kondisi where. Anda menggunakan kueri upsert alih-alih kueri create karena upsert menghilangkan kesalahan yang terkait dengan mencoba secara tidak sengaja menyisipkan catatan yang sama dua kali.

Anda perlu memberi tahu Prisma skrip apa yang akan dieksekusi saat menjalankan perintah seeding. Anda dapat melakukannya dengan menambahkan kunci prisma.seed di akhir berkas package.json Anda:


```
// package.json

// ...
  "scripts": {
    // ...
  },
  "dependencies": {
    // ...
  },
  "devDependencies": {
    // ...
  },
  "jest": {
    // ...
  },
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
  ```

  Perintah penanaman data akan menjalankan skrip prisma/seed.ts yang telah Anda definisikan sebelumnya. Perintah ini seharusnya berfungsi secara otomatis karena ts-node sudah diinstal sebagai dependensi pengembangan (dev dependency) dalam package.json Anda. Ini memungkinkan Anda menjalankan skrip TypeScript tanpa perlu kompilasi manual. Pastikan Anda telah menjalankan perintah ini setelah mempersiapkan basis data dan mendefinisikan skrip penanaman data.

  Anda dapat mengeksekusi perintah penanaman data menggunakan perintah berikut:

```bash
npx prisma db seed
```

Anda akan melihat keluaran berikut:

```
Running seed command `ts-node prisma/seed.ts` ...
{
  post1: {
    id: 1,
    title: 'Prisma Adds Support for MongoDB',
    description: "We are excited to share that today's Prisma ORM release adds stable support for MongoDB!",
    body: 'Support for MongoDB has been one of the most requested features since the initial release of...',
    published: false,
    createdAt: 2022-04-24T14:20:27.674Z,
    updatedAt: 2022-04-24T14:20:27.674Z
  },
  post2: {
    id: 2,
    title: "What's new in Prisma? (Q1/22)",
    description: 'Learn about everything in the Prisma ecosystem and community from January to March 2022.',
    body: 'Our engineers have been working hard, issuing new releases with many improvements...',
    published: true,
    createdAt: 2022-04-24T14:20:27.705Z,
    updatedAt: 2022-04-24T14:20:27.705Z
  }
}

🌱  The seed command has been executed.
```

Perintah ini akan menjalankan skrip penanaman data yang telah Anda definisikan sebelumnya. Data awal akan ditambahkan ke basis data Anda sesuai dengan skrip yang Anda tulis. Pastikan Anda telah menjalankan perintah ini setelah mempersiapkan basis data dan mendefinisikan skrip penanaman data.

## Membuat Layanan Prisma

Dalam aplikasi NestJS Anda, adalah praktik yang baik untuk mengabstraksi API Prisma Client dari aplikasi Anda. Untuk melakukannya, Anda akan membuat layanan baru yang akan berisi Prisma Client. Layanan ini, disebut PrismaService, akan bertanggung jawab atas instansiasi PrismaClient dan menghubungkan ke basis data Anda.

Nest CLI memberikan cara mudah untuk menghasilkan modul dan layanan secara langsung melalui CLI. Jalankan perintah berikut di terminal Anda:

```bash
npx nest generate module prisma
npx nest generate service prisma
```

Langkah ini akan menciptakan modul Prisma dan layanan Prisma dalam aplikasi NestJS Anda. Selanjutnya, Anda dapat memasukkan logika Prisma dalam layanan Prisma ini untuk digunakan dalam aplikasi Anda. Pastikan untuk menambahkan modul Prisma ke modul utama Anda jika belum ditambahkan.

Ini akan menghasilkan subdirektori baru ./src/prisma dengan file prisma.module.ts dan prisma.service.ts. File layanan seharusnya berisi kode berikut:

```typescript
// src/prisma/prisma.service.ts

import { Injectable } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient {}
```

Modul Prisma akan bertanggung jawab atas menciptakan instansi singleton dari PrismaService dan memungkinkan berbagi layanan tersebut di seluruh aplikasi Anda. Untuk melakukannya, Anda akan menambahkan PrismaService ke dalam array exports di file prisma.module.ts:

```typescript
// src/prisma/prisma.module.ts

import { Module } from '@nestjs/common'; 
import { PrismaService } from './prisma.service';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

Sekarang, setiap modul yang mengimpor PrismaModule akan memiliki akses ke PrismaService dan dapat menyuntikkannya ke dalam komponen/layanan mereka sendiri. Ini adalah pola umum dalam aplikasi NestJS.

Dengan demikian, Anda telah menyelesaikan pengaturan Prisma! Sekarang Anda dapat mulai membangun REST API Anda. 

**Langkah-langkah Konfigurasi Swagger di Aplikasi NestJS**

Swagger adalah alat untuk mendokumentasikan API Anda menggunakan spesifikasi OpenAPI. Nest memiliki modul khusus untuk Swagger, yang akan Anda gunakan dalam waktu dekat.

**Langkah 1:** Install dependensi yang diperlukan dengan menjalankan perintah berikut di direktori proyek Anda:

```bash
npm install --save @nestjs/swagger swagger-ui-express
```

**Langkah 2:** Buka file `main.ts`, di mana Anda menginisialisasi aplikasi NestJS Anda.

**Langkah 3:** Impor modul yang diperlukan dan buat dokumentasi Swagger:

```javascript
// src/main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Membuat konfigurasi dokumen Swagger
  const config = new DocumentBuilder()
    .setTitle('Median')
    .setDescription('Deskripsi API Median')
    .setVersion('0.1')
    .build();

  // Membuat dokumen Swagger dengan menggunakan konfigurasi
  const document = SwaggerModule.createDocument(app, config);

  // Menyiapkan Swagger UI untuk melayani dokumentasi
  SwaggerModule.setup('api', app, document);

  // Memulai aplikasi NestJS Anda
  await app.listen(3000);
}

bootstrap();
```

Ketika aplikasi berjalan, buka peramban Anda dan akses http://localhost:3000/api. Anda akan melihat antarmuka Swagger (Swagger UI).

![Alt text](.images/image-1.png)

**Implementasi Operasi CRUD untuk Model Artikel**

Pada bagian ini, Anda akan mengimplementasikan operasi Create, Read, Update, dan Delete (CRUD) untuk model Artikel beserta logika bisnis yang mendampinginya.

**Langkah 1: Menghasilkan Sumber Daya REST**

Sebelum Anda dapat mengimplementasikan REST API, Anda perlu menghasilkan sumber daya REST untuk model Artikel. Anda dapat melakukannya dengan cepat menggunakan Nest CLI. Jalankan perintah berikut di terminal Anda:

```
npx nest generate resource
```

**Menghasilkan Sumber Daya REST dengan CLI**

Saat menjalankan perintah di atas, Anda akan diberikan beberapa pertanyaan oleh CLI. Berikut adalah cara menjawabnya:

- Apa nama yang ingin Anda gunakan untuk sumber daya ini (dalam bentuk jamak, misalnya, "artikel")? Jawaban: articles
- Lapisan transport apa yang Anda gunakan? REST API. Jawaban: REST API
- Apakah Anda ingin menghasilkan titik masuk CRUD? Jawaban: Ya

Setelah menjawab pertanyaan ini, Anda akan memiliki direktori baru "src/articles" yang berisi boilerplate untuk titik akhir REST. Di dalam file "src/articles/articles.controller.ts", Anda akan melihat definisi berbagai rute (route handlers). Logika bisnis untuk menangani setiap permintaan ada di dalam file "src/articles/articles.service.ts". Saat ini, file ini mungkin berisi implementasi contoh.

Jika Anda membuka kembali halaman API Swagger, Anda akan melihat sesuatu seperti ini: 
![Alt text](.images/image-2.png)

SwaggerModule mencari semua decorator @Body(), @Query(), dan @Param() pada handler rute untuk menghasilkan halaman API ini.

## Menambahkan PrismaClient ke Modul Artikel
Untuk mengakses PrismaClient di dalam modul Artikel, Anda harus menambahkan PrismaModule sebagai impor. Tambahkan impor berikut ke ArticlesModule:

```javascript
// src/articles/articles.module.ts

import { Module } from '@nestjs/common';
import { ArticlesService } from './articles.service';
import { ArticlesController } from './articles.controller';
import { PrismaModule } from 'src/prisma/prisma.module';

@Module({
  controllers: [ArticlesController],
  providers: [ArticlesService],
  imports: [PrismaModule],
})
export class ArticlesModule {}
```

Setelah menambahkan PrismaModule ke impor modul Artikel, Anda dapat menyuntikkan PrismaService ke dalam ArticlesService dan menggunakannya untuk mengakses database. Untuk melakukannya, tambahkan constructor ke articles.service.ts seperti berikut:

```javascript
// src/articles/articles.service.ts

import { Injectable } from '@nestjs/common';
import { CreateArticleDto } from './dto/create-article.dto';
import { UpdateArticleDto } from './dto/update-article.dto';
import { PrismaService } from 'src/prisma/prisma.service';

@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  // Operasi CRUD
}
```
# Mendefinisikan Endpoints GET /articles

Dalam langkah ini, Anda akan mendefinisikan endpoint GET `/articles` yang akan digunakan untuk mengambil semua artikel yang telah dipublikasikan di basis data. Controller yang menangani endpoint ini disebut `findAll`. Berikut adalah contoh kode untuk controller `findAll`:

```javascript
// src/articles/articles.controller.ts

@Get()
findAll() {
  return this.articlesService.findAll();
}
```

Namun, Anda perlu memperbarui metode `findAll` di dalam `ArticlesService` agar mengembalikan array semua artikel yang telah dipublikasikan dalam basis data. Berikut adalah contoh perbaruan kode untuk `articles.service.ts`:

```javascript
@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  create(createArticleDto: CreateArticleDto) {
    return 'This action adds a new article';
  }

  findAll() {
    return this.prisma.article.findMany({ where: { published: true } });
  }
}
```

Perintah `findMany` akan mengembalikan semua catatan artikel yang cocok dengan kondisi `where`.

Anda dapat menguji endpoint ini dengan membuka Swagger di alamat http://localhost:3000/api, kemudian pilih opsi GET/articles. Klik pada tombol "Try it out" dan lalu "Execute" untuk melihat hasilnya.

![Alt text](.images/image-3.png)


# Menyusun Endpoints GET `/articles/drafts`

Anda akan mendefinisikan sebuah rute baru untuk mengambil semua artikel yang belum dipublikasikan. NestJS tidak secara otomatis menghasilkan handler rute kontroler untuk endpoint ini, jadi Anda harus menulisnya sendiri.

```typescript
// src/articles/articles.controller.ts

@Controller('articles')
export class ArticlesController {
  constructor(private readonly articlesService: ArticlesService) {}

  @Post()
  create(@Body() createArticleDto: CreateArticleDto) {
    return this.articlesService.create(createArticleDto);
  }

  @Get('drafts')
  findDrafts() {
    return this.articlesService.findDrafts();
  }

  // ...

}
```

Editor Anda akan menunjukkan kesalahan bahwa tidak ada fungsi yang disebut `articlesService.findDrafts()`. Untuk memperbaikinya, Anda perlu mengimplementasikan metode `findDrafts` di dalam `ArticlesService`.

```typescript
// src/articles/articles.service.ts

@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  create(createArticleDto: CreateArticleDto) {
    return 'This action adds a new article';
  }

  findDrafts() {
    return this.prisma.article.findMany({ where: { published: false } });
  }

  // ...

}
```

Dengan mengimplementasikan metode `findDrafts`, Anda sekarang dapat mengakses dan mengambil artikel yang belum dipublikasikan melalui endpoint GET `/articles/drafts`. Endpoint ini akan tersedia di halaman Swagger API. Anda dapat mengujinya dengan membuka halaman Swagger, mengklik opsi GET/articles/drafts, lalu mengklik tombol "Try it out" dan "Execute" untuk melihat hasilnya.

# Mendefinisikan Endpoints GET `/articles/:id`

Handler rute kontroler untuk endpoint ini disebut `findOne` dan terlihat seperti ini:

```typescript
// src/articles/articles.controller.ts

@Get(':id')
findOne(@Param('id') id: string) {
  return this.articlesService.findOne(+id);
}
```

Rute ini menerima parameter dinamis `id`, yang diteruskan ke handler rute kontroler `findOne`. Karena model `Article` memiliki bidang `id` yang berupa bilangan bulat, parameter `id` perlu diubah menjadi angka dengan menggunakan operator +.

Sekarang, perbarui metode `findOne` di dalam `ArticlesService` untuk mengembalikan artikel dengan id yang diberikan:

```typescript
// src/articles/articles.service.ts
@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  create(createArticleDto: CreateArticleDto) {
    return 'This action adds a new article';
  }

  findAll() {
    return this.prisma.article.findMany({ where: { published: true } });
  }

  findOne(id: number) {
    return this.prisma.article.findUnique({ where: { id } });
  }
}
```

Sekali lagi, Anda dapat menguji endpoint ini dengan pergi ke http://localhost:3000/api. Klik pada menu dropdown GET /articles/{id}. Tekan tombol "Try it out", tambahkan nilai yang valid ke parameter id, dan tekan tombol "Execute" untuk melihat hasilnya.

![Alt text](.images/image-4.png)

Define POST /articles endpoint
Ini adalah endpoint untuk membuat artikel baru. Controller route handler untuk endpoint ini disebut create. Ini terlihat seperti ini:

```javascript
// src/articles/articles.controller.ts

@Post()
create(@Body() createArticleDto: CreateArticleDto) {
  return this.articlesService.create(createArticleDto);
}
```

Perhatikan bahwa itu mengharapkan argumen bertipe CreateArticleDto dalam body permintaan. DTO (Data Transfer Object) adalah objek yang mendefinisikan bagaimana data akan dikirimkan melalui jaringan. Saat ini, CreateArticleDto adalah kelas kosong. Anda akan menambahkan properti ke dalamnya untuk mendefinisikan bentuk body permintaan.

```javascript
// src/articles/dto/create-article.dto.ts

import { ApiProperty } from '@nestjs/swagger';

export class CreateArticleDto {
  @ApiProperty()
  title: string;

  @ApiProperty({ required: false })
  description?: string;

  @ApiProperty()
  body: string;

  @ApiProperty({ required: false, default: false })
  published?: boolean = false;
}
```

Dekorator @ApiProperty diperlukan untuk membuat properti kelas terlihat oleh SwaggerModule. Informasi lebih lanjut tentang ini tersedia di dokumen NestJS.

CreateArticleDto sekarang harus didefinisikan dalam halaman API Swagger di bawah Schemas. Bentuk UpdateArticleDto secara otomatis disimpulkan dari definisi CreateArticleDto. Jadi UpdateArticleDto juga didefinisikan di dalam Swagger.

![Alt text](.images/image-5.png)

Sekarang, perbarui metode create dalam ArticlesService untuk membuat artikel baru dalam database:

```typescript
// src/articles/articles.service.ts

@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  create(createArticleDto: CreateArticleDto) {
    return this.prisma.article.create({ data: createArticleDto });
  }

  // ...
}
```

Tentukan endpoint PATCH /articles/:id
Endpoint ini digunakan untuk memperbarui artikel yang ada. Route handler untuk endpoint ini disebut `update`. Ini terlihat seperti ini:

```typescript
// src/articles/articles.controller.ts

@Patch(':id')
update(@Param('id') id: string, @Body() updateArticleDto: UpdateArticleDto) {
  return this.articlesService.update(+id, updateArticleDto);
}
```

Definisi `updateArticleDto` didefinisikan sebagai `PartialType` dari `CreateArticleDto`. Jadi, itu dapat memiliki semua properti dari `CreateArticleDto`.

```typescript
// src/articles/dto/update-article.dto.ts

import { PartialType } from '@nestjs/swagger';
import { CreateArticleDto } from './create-article.dto';

export class UpdateArticleDto extends PartialType(CreateArticleDto) {}
```

Seperti sebelumnya, Anda harus memperbarui metode layanan yang sesuai untuk operasi ini:

```typescript
// src/articles/articles.service.ts

@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) {}

  // ...

  update(id: number, updateArticleDto: UpdateArticleDto) {
    return this.prisma.article.update({
      where: { id },
      data: updateArticleDto,
    });
  }

  // ...
}
```

Operasi article.update akan mencoba menemukan catatan Artikel dengan id yang diberikan dan memperbarui catatan tersebut dengan data dari updateArticleDto.

Jika tidak ada catatan Artikel yang sesuai dengan id tersebut di database, Prisma akan mengembalikan sebuah kesalahan. Dalam kasus seperti ini, API tidak akan mengembalikan pesan kesalahan yang ramah pengguna. Anda akan mempelajari tentang penanganan kesalahan dengan NestJS dalam tutorial mendatang.

## Define DELETE /articles/:id endpoint
Endpoint ini digunakan untuk menghapus artikel yang sudah ada. Route handler untuk endpoint ini dinamai remove dan memiliki penampakan seperti ini:

```javascript
// src/articles/articles.controller.ts

@Delete(':id')
remove(@Param('id') id: string) {
  return this.articlesService.remove(+id);
}
```

Sama seperti sebelumnya, buka file ArticlesService dan perbarui metode yang sesuai:

```javascript
// src/articles/articles.service.ts

@Injectable()
export class ArticlesService {
  constructor(private prisma: PrismaService) { }

  // ...

  remove(id: number) {
    return `Tindakan ini menghapus artikel #${id}`;
    return this.prisma.article.delete({ where: { id } });
  }
}
```

Itu adalah operasi terakhir untuk endpoint artikel. Selamat, API Anda hampir siap! 🎉

## Group endpoints together in Swagger
Tambahkan decorator @ApiTags ke kelas ArticlesController untuk mengelompokkan semua endpoint artikel bersama-sama dalam Swagger:

```javascript
// src/articles/articles.controller.ts

import { ApiTags } from '@nestjs/swagger';

@Controller('articles')
@ApiTags('articles')
export class ArticlesController {
  // ...
}
```

Halaman API sekarang memiliki semua endpoint artikel yang dikelompokkan bersama-sama.

![Alt text](.images/image-6.png)

Update Swagger response types
Jika Anda melihat tab Respons di bawah setiap endpoint di Swagger, Anda akan menemukan bahwa Deskripsi kosong. Ini terjadi karena Swagger tidak mengetahui jenis respons untuk setiap endpoint. Anda akan memperbaikinya dengan menggunakan beberapa decorator.

Pertama, Anda perlu mendefinisikan sebuah entitas yang dapat digunakan oleh Swagger untuk mengidentifikasi bentuk objek entitas yang dikembalikan. Untuk melakukannya, perbarui kelas ArticleEntity dalam berkas articles.entity.ts sebagai berikut:

```javascript
// src/articles/entities/article.entity.ts

import { Article } from '@prisma/client';
import { ApiProperty } from '@nestjs/swagger';

export class ArticleEntity implements Article {
  @ApiProperty()
  id: number;

  @ApiProperty()
  title: string;

  @ApiProperty({ required: false, nullable: true })
  description: string | null;

  @ApiProperty()
  body: string;

  @ApiProperty()
  published: boolean;

  @ApiProperty()
  createdAt: Date;

  @ApiProperty()
  updatedAt: Date;
}
```

Ini adalah implementasi tipe Article yang dihasilkan oleh Prisma Client, dengan decorator @ApiProperty ditambahkan ke setiap properti.

Sekarang, saatnya untuk memberi anotasi pada handler rute pengendali dengan jenis respons yang benar. NestJS memiliki serangkaian decorator untuk tujuan ini.

```javascript
// src/articles/articles.controller.ts

import { ApiCreatedResponse, ApiOkResponse, ApiTags } from '@nestjs/swagger';
import { ArticleEntity } from './entities/article.entity';

@Controller('articles')
@ApiTags('articles')
export class ArticlesController {
  constructor(private readonly articlesService: ArticlesService) {}

  @Post()
  @ApiCreatedResponse({ type: ArticleEntity })
  create(@Body() createArticleDto: CreateArticleDto) {
    return this.articlesService.create(createArticleDto);
  }

  @Get()
  @ApiOkResponse({ type: ArticleEntity, isArray: true })
  findAll() {
    return this.articlesService.findAll();
  }

  @Get('drafts')
  @ApiOkResponse({ type: ArticleEntity, isArray: true })
  findDrafts() {
    return this.articlesService.findDrafts();
  }

  @Get(':id')
  @ApiOkResponse({ type: ArticleEntity })
  findOne(@Param('id') id: string) {
    return this.articlesService.findOne(+id);
  }

  @Patch(':id')
  @ApiOkResponse({ type: ArticleEntity })
  update(@Param('id') id: string, @Body() updateArticleDto: UpdateArticleDto) {
    return this.articlesService.update(+id, updateArticleDto);
  }

  @Delete(':id')
  @ApiOkResponse({ type: ArticleEntity })
  remove(@Param('id') id: string) {
    return this.articlesService.remove(+id);
  }
}
```

Anda telah menambahkan @ApiOkResponse untuk endpoint GET, PATCH, dan DELETE dan @ApiCreatedResponse untuk endpoint POST. Properti type digunakan untuk menentukan tipe kembalian. Anda dapat menemukan semua decorator respons yang disediakan oleh NestJS di dokumentasi NestJS.

Sekarang, Swagger seharusnya sudah dengan benar menentukan tipe respons untuk semua endpoint di halaman API.

![Alt text](.images/image-7.png)

Selamat! Anda telah berhasil membangun REST API sederhana dengan menggunakan NestJS. Selama tutorial ini, Anda:

Membangun REST API dengan NestJS
Mengintegrasikan Prisma dengan lancar dalam proyek NestJS
Mendokumentasikan REST API Anda menggunakan Swagger dan OpenAPI
Salah satu hal utama yang dapat Anda pelajari dari tutorial ini adalah seberapa mudahnya membangun REST API dengan NestJS dan Prisma. Ini adalah tumpukan teknologi yang sangat produktif untuk membangun aplikasi backend yang terstruktur dengan baik, aman dalam hal tipe data, dan mudah untuk dipelihara.

Anda dapat menemukan kode sumber untuk proyek ini di GitHub. Jangan ragu untuk membuat isu di repositori tersebut atau mengirimkan permintaan tarik jika Anda menemui masalah. Anda juga dapat menghubungi saya langsung di Twitter.# nest-lab-crud
