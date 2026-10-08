# คู่มือแก้ปัญหา Project Hub

## Error: `connect ECONNREFUSED 127.0.0.1:3306`

ข้อผิดพลาดนี้หมายความว่า Node.js ติดต่อ MySQL/MariaDB ที่ `127.0.0.1:3306` ไม่ได้ โดยทั่วไปเกิดจาก **บริการฐานข้อมูลยังไม่เปิด**, ติดตั้งไว้คนละพอร์ต หรือค่าในไฟล์ `.env` ไม่ตรงกับฐานข้อมูลจริง

### วิธีแก้บน Windows

#### วิธีที่ 1: ถ้าใช้ MySQL Installer

1. กด `Win + R` → พิมพ์ `services.msc` → Enter
2. ค้นหาบริการชื่อ **MySQL80** (ชื่ออาจเป็น MySQL เวอร์ชันอื่น)
3. คลิกขวา → **Start** หรือ **Restart**
4. เปิด PowerShell ใหม่ แล้วตรวจสอบพอร์ต:

   ```powershell
   Test-NetConnection 127.0.0.1 -Port 3306
   ```

   ถ้าแสดง `TcpTestSucceeded : True` แปลว่าฐานข้อมูลเปิดแล้ว

#### วิธีที่ 2: ถ้าใช้ XAMPP

1. เปิด **XAMPP Control Panel**
2. กด **Start** ที่แถว **MySQL** (ไม่ใช่ Apache อย่างเดียว)
3. ตรวจสอบว่าพอร์ตแสดงเป็น `3306`
4. หาก XAMPP ใช้พอร์ตอื่น ให้แก้ `DB_PORT` ในไฟล์ `.env` ให้ตรงกัน

#### วิธีที่ 3: ถ้าใช้ MariaDB

1. เปิด `services.msc`
2. ค้นหา **MariaDB**
3. คลิกขวา → **Start**

หรือเปิด PowerShell แบบ Administrator:

```powershell
Get-Service *mysql*,*maria* | Format-Table Name,Status,DisplayName
Start-Service MariaDB       # เปลี่ยนเป็นชื่อบริการจริงหากแตกต่าง
```

### ตรวจสอบไฟล์ `.env`

ไปที่โฟลเดอร์เดียวกับ `package.json` แล้วตรวจสอบค่าต่อไปนี้:

```dotenv
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=รหัสผ่าน MySQL ของคุณ
DB_NAME=project_hub
```

หากใช้ XAMPP ที่ไม่มีรหัสผ่าน root ให้ใช้ `DB_PASSWORD=` ว่างได้ แต่ไม่ควรใช้การตั้งค่านี้ใน Production

> ต้องแก้ไฟล์ `.env` ไม่ใช่ `.env.example` และห้ามใส่ช่องว่างรอบเครื่องหมาย `=`

### สร้างฐานข้อมูลและตาราง

หลังเปิดบริการ MySQL/MariaDB แล้ว ให้เปิด PowerShell ในโฟลเดอร์โปรเจกต์และรัน:

```powershell
npm install
npm run check
npm run migrate
npm run create-admin
npm run seed
npm start
```

หาก `npm run check` ยังขึ้น `ECONNREFUSED` ให้ตรวจพอร์ตด้วย `Test-NetConnection` อีกครั้ง

### ถ้าฐานข้อมูลใช้พอร์ตอื่น

ตัวอย่างเช่น MySQL ใช้พอร์ต `3307` ให้แก้ `.env`:

```dotenv
DB_PORT=3307
```

จากนั้นปิดแล้วเปิดเซิร์ฟเวอร์ใหม่ด้วย `Ctrl+C` และ `npm start`

### ถ้าขึ้น `ER_ACCESS_DENIED_ERROR`

แปลว่าติดต่อเซิร์ฟเวอร์ได้แล้ว แต่ชื่อผู้ใช้หรือรหัสผ่านผิด ให้ตรวจ `DB_USER` และ `DB_PASSWORD` ใน `.env`

### ถ้าขึ้น `ER_BAD_DB_ERROR`

แปลว่ายังไม่มีฐานข้อมูล `project_hub` ให้สร้างก่อน:

```sql
CREATE DATABASE project_hub CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

จากนั้นรัน `npm run migrate`

### ถ้าขึ้น `ER_NO_SUCH_TABLE`

ฐานข้อมูลมีอยู่แล้ว แต่ยังไม่มีตาราง ให้รัน:

```powershell
npm run migrate
```

### ลำดับตรวจสอบแบบเร็ว

```powershell
# 1) ตรวจว่ามีพอร์ตเปิดหรือไม่
Test-NetConnection 127.0.0.1 -Port 3306

# 2) ตรวจค่าที่แอปอ่านจาก .env (ไม่แสดงรหัสผ่าน)
npm run check

# 3) สร้างตาราง
npm run migrate

# 4) เปิดเว็บ
npm start
```

เว็บจะเปิดที่ `http://localhost:3000/` และระบบหลังบ้านอยู่ที่ `http://localhost:3000/admin`

## ข้อควรระวัง

- ต้องเปิดบริการฐานข้อมูล **ก่อน** รัน `npm start`
- หากใช้ไฟล์ ZIP ให้แตกไฟล์ไว้ในโฟลเดอร์ที่เขียนไฟล์ได้ และรันคำสั่งจากโฟลเดอร์ที่มี `package.json`
- ระบบใช้ session ในตาราง `sessions` จึงต้องสร้างตารางก่อนจึงจะเข้าสู่ระบบได้
- เปลี่ยน `SESSION_SECRET` และรหัสผ่านผู้ดูแลก่อนใช้งานจริง
