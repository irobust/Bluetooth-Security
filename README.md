# Hands-on Lab: Bluetooth Security Testing with BlueToolkit + Caeruleus + nRF Connect
## หลักสูตร 1 วัน — คู่มือผู้สอนและผู้เข้าอบรม

> **ขอบเขตและความปลอดภัย**
>
> คู่มือนี้ออกแบบสำหรับการทดสอบอุปกรณ์ที่ผู้เข้าอบรมเป็นเจ้าของ หรือได้รับอนุญาตเป็นลายลักษณ์อักษรเท่านั้น ใช้พื้นที่ทดสอบที่ควบคุมได้ ห้ามทดสอบรถยนต์ อุปกรณ์สาธารณะ อุปกรณ์ของบุคคลอื่น หรืออุปกรณ์ที่กำลังใช้งานจริงโดยไม่ได้รับอนุญาต
>
> Lab นี้เน้นการสำรวจ (reconnaissance) การตรวจสอบค่าความปลอดภัย การรวบรวมหลักฐาน และการจัดทำรายงาน ไม่ให้ขั้นตอนโจมตี DoS, RCE, การดักข้อมูล, MITM, การข้ามการยืนยันตัวตน หรือการรบกวนการเชื่อมต่อกับอุปกรณ์จริง
>
> **ชื่อเครื่องมือ:** เอกสารสาธารณะที่ตรงกับงานนี้คือ **BlueToolkit** (ไม่ใช่ยืนยันว่า “BluetoothKit” ที่ผู้ใช้หมายถึงเป็นโครงการเดียวกัน) — https://github.com/sgxgsx/BlueToolkit  
> ตรวจสอบ README และ Wiki ล่าสุดก่อนเริ่ม เพราะความสามารถ คำสั่ง และความต้องการฮาร์ดแวร์อาจเปลี่ยนแปลงได้

---

## 1. วัตถุประสงค์การเรียนรู้

เมื่อจบ Lab ผู้เรียนจะสามารถ:

1. แยกความแตกต่างระหว่าง Bluetooth Classic (BR/EDR) และ Bluetooth Low Energy (BLE)
2. เตรียมเครื่องทดสอบ Linux และตรวจสอบ Bluetooth adapter ได้
3. ระบุอุปกรณ์เป้าหมายที่ได้รับอนุญาตและบันทึกข้อมูลพื้นฐานได้
4. ใช้ BlueToolkit สำหรับตรวจสอบ reconnaissance และ connectivity ตามความสามารถที่ติดตั้งไว้
5. ใช้ Caeruleus และ nRF Connect for Mobile ตรวจสอบ BLE discovery, GATT services และ characteristics บน nRF52840 ที่เตรียมไว้
6. เข้าใจบทบาทของ nRF52840 ที่ทำงานเป็น BLE peripheral เทียบกับการแฟลช firmware ให้เป็น packet sniffer
7. อ่านผลลัพธ์และแยกข้อเท็จจริงออกจากข้อสันนิษฐาน
8. จัดทำหลักฐานและรายงาน finding พร้อมระดับความเสี่ยงและแนวทางแก้ไข

## 2. กำหนดการอบรม 1 วัน (ฉบับปรับปรุง)

| เวลา | หัวข้อ | รูปแบบ |
|---|---|---|
| 09:00–09:35 | Bluetooth Classic, BLE, GATT, pairing และ threat model | บรรยาย |
| 09:35–10:20 | Lab 0: Setup BlueZ, BlueToolkit และ Caeruleus | Hands-on |
| 10:20–10:35 | พัก | |
| 10:35–11:15 | Lab 1: BLE Discovery และ Target Inventory | Hands-on |
| 11:15–12:00 | Lab 2: BlueToolkit Reconnaissance | Hands-on |
| 12:00–13:00 | พักกลางวัน | |
| 13:00–13:35 | Lab 3: Caeruleus GATT Enumeration และ Read | Hands-on |
| 13:35–14:10 | Lab 4: Authentication และ Encryption Assessment | Hands-on |
| 14:10–14:25 | พัก | |
| 14:25–15:10 | Lab 5: เปรียบเทียบ Insecure / Secure ESP32 | Hands-on |
| 15:10–15:50 | Lab 6: nRF Connect for Mobile + nRF52840 | Hands-on |
| 15:50–16:30 | Lab 7: Evidence, JSON และ Reporting | Hands-on |
| 16:30–17:00 | สรุปผลและนำเสนอ Finding | Workshop |

## 3. ข้อกำหนดก่อนเข้าอบรม

### ฮาร์ดแวร์

- Laptop ที่รัน Ubuntu/Debian Linux ได้จริงหรือ VM ที่ส่งผ่าน Bluetooth USB adapter ได้
- Bluetooth USB adapter ที่รองรับ BlueZ และมองเห็นได้จาก Linux
- อุปกรณ์เป้าหมายสำหรับทดสอบโดยได้รับอนุญาต เช่น ESP32 DemoSensor และ/หรือ Nordic nRF52840 DK (แนะนำ PCA10056)
- Android/iOS smartphone ที่ติดตั้ง nRF Connect for Mobile
- (ตัวเลือก) nRF52840 DK/Dongle เพิ่มอีกหนึ่งตัวสำหรับ nRF Sniffer; อุปกรณ์ที่ถูกแฟลชเป็น sniffer จะไม่ทำหน้าที่เป็น BLE peripheral ในเวลาเดียวกัน
- แนะนำให้ใช้ target สำหรับ Lab ที่ไม่มีข้อมูลส่วนบุคคลและไม่มีผลกระทบต่อบริการจริง

### ซอฟต์แวร์

- Ubuntu/Debian รุ่นที่ BlueToolkit ระบุว่ารองรับ
- BlueZ utilities เช่น `bluetoothctl`, `btmgmt`, `bluetoothd`
- Git และเครื่องมือ build ที่จำเป็นตาม README ของ BlueToolkit
- BlueToolkit จาก repository ทางการ: https://github.com/sgxgsx/BlueToolkit
- Caeruleus จาก repository ทางการ: https://github.com/praetorian-inc/caeruleus
- Go toolchain ที่รองรับเวอร์ชันของ Caeruleus หากติดตั้งจาก source
- nRF Connect for Mobile: https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile
- nRF Connect for Desktop (Programmer) หากต้องแฟลช firmware ลง nRF52840: https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-desktop
- nRF Sniffer for Bluetooth LE (ตัวเลือกสำหรับ packet capture): https://www.nordicsemi.com/Products/Development-tools/nRF-Sniffer-for-Bluetooth-LE
- Wireshark เป็นตัวเลือกสำหรับการวิเคราะห์ capture ที่เก็บอย่างถูกต้องและได้รับอนุญาต

> ไม่ควรติดตั้งเครื่องมือโดยเดาคำสั่งจากบทความเก่า ให้ตรวจสอบ README, installation script และ dependencies ของ repository ปัจจุบันก่อนเสมอ

---

# Lab 0 — เตรียมสภาพแวดล้อม

## เป้าหมาย

ยืนยันว่า Linux มองเห็น Bluetooth adapter และผู้เรียนเข้าใจขอบเขตของ target ก่อนเริ่มทดสอบ

## ขั้นตอน

1. เปิดเครื่อง Linux และเข้าสู่ระบบด้วยบัญชีที่มีสิทธิ์ติดตั้ง package
2. ตรวจสอบระบบปฏิบัติการ:

   ```bash
   cat /etc/os-release
   uname -a
   ```

3. ตรวจสอบว่ามี Bluetooth controller:

   ```bash
   bluetoothctl list
   ```

4. ตรวจสอบสถานะ controller:

   ```bash
   bluetoothctl show
   ```

5. หากคำสั่งไม่พบ controller ให้ตรวจสอบการเชื่อมต่อ USB, USB passthrough ของ VM, service ของ Bluetooth และ driver ก่อน ห้ามดำเนินการสแกนต่อจนกว่าจะยืนยัน adapter ได้
6. ตรวจสอบสถานะ service:

   ```bash
   systemctl status bluetooth --no-pager
   ```

7. อ่าน README ของ BlueToolkit และทำตามขั้นตอนติดตั้งอย่างเป็นทางการ:

   ```bash
   git clone --recurse-submodules https://github.com/sgxgsx/BlueToolkit.git
   cd BlueToolkit
   ```

8. ตรวจสอบไฟล์และคำแนะนำใน repository:

   ```bash
   ls
   less README.md
   ```

9. หากจะเรียก installer ให้ตรวจสอบเนื้อหาสคริปต์ก่อน และรันเฉพาะใน VM/lab ที่ควบคุมได้ โดยใช้ขั้นตอนตาม README ปัจจุบันของโครงการ
10. บันทึกเวอร์ชันของระบบ เครื่องมือ และ adapter ลงใน lab notes

## ผลที่คาดหวัง

- Linux เห็น controller
- ผู้เรียนระบุได้ว่าใช้ adapter ใด
- มีบันทึกเวอร์ชันและขอบเขต target
- ไม่มีการสแกนหรือทดสอบอุปกรณ์ที่ไม่ได้รับอนุญาต

## Troubleshooting

- **ไม่พบ controller:** ตรวจสอบ USB passthrough, driver, `rfkill list` และสถานะ Bluetooth
- **Bluetooth ถูก block:** ตรวจสอบ `rfkill`; ปลด block เฉพาะเครื่องทดสอบที่ได้รับอนุญาต
- **VM ไม่เห็น adapter:** ตั้งค่า USB passthrough ให้ adapter ปรากฏใน guest OS
- **ติดตั้ง BlueToolkit ไม่สำเร็จ:** อ่าน error และ README/Issues ของ repository อย่ารันสคริปต์สุ่มจากแหล่งอื่น

---

# Lab 1 — สำรวจอุปกรณ์และสร้าง Target Inventory

## เป้าหมาย

สร้างรายการอุปกรณ์ที่อยู่ในขอบเขตการทดสอบ โดยบันทึกข้อมูลที่จำเป็นต่อการประเมินความเสี่ยง

## ขั้นตอน

1. ขออนุญาตและยืนยันชื่อ/รุ่นของอุปกรณ์เป้าหมายจากผู้สอน
2. บันทึกข้อมูลต่อไปนี้ก่อนทดสอบ:
   - Asset ID หรือชื่อสมมติ
   - ผู้เป็นเจ้าของและหลักฐานอนุญาต
   - รุ่นอุปกรณ์และ firmware หากทราบ
   - ประเภท Bluetooth: Classic, BLE หรือ dual-mode
   - Bluetooth address ที่จำเป็นสำหรับการทดสอบ
   - เวลาที่เริ่มทดสอบและผู้ปฏิบัติงาน
3. ตรวจสอบว่า controller พร้อมใช้งาน:

   ```bash
   bluetoothctl show
   ```

4. เปิด interactive shell:

   ```bash
   bluetoothctl
   ```

5. ภายใน `bluetoothctl` ใช้คำสั่งช่วยเหลือเพื่อดูคำสั่งที่รองรับ:

   ```text
   help
   ```

6. เริ่ม discovery เฉพาะในห้องแล็บและช่วงเวลาที่ได้รับอนุญาต:

   ```text
   scan on
   ```

7. สังเกตอุปกรณ์ที่แสดงขึ้นมา บันทึกเฉพาะ target ที่ผู้สอนยืนยันว่าอยู่ในขอบเขต อย่าพยายามเชื่อมต่อกับอุปกรณ์ที่ไม่รู้จัก
8. หยุด discovery:

   ```text
   scan off
   ```

9. ออกจาก shell:

   ```text
   quit
   ```

10. เติมตาราง inventory:

| Field | ค่าที่บันทึก |
|---|---|
| Lab Asset ID | |
| Device model | |
| Bluetooth address | |
| Bluetooth type | Classic / BLE / Dual-mode / Unknown |
| Advertised name | |
| Services/UUIDs ที่พบ | |
| Firmware/version | |
| Scope authorization | |
| Observation timestamp | |

## คำถามสำหรับผู้เรียน

- เหตุใดชื่ออุปกรณ์และ MAC address เพียงอย่างเดียวจึงไม่ยืนยันตัวตนอุปกรณ์ได้อย่างสมบูรณ์?
- การพบอุปกรณ์ใน discovery หมายความว่าอุปกรณ์มีช่องโหว่หรือไม่?
- ควรทำอย่างไรเมื่อพบอุปกรณ์ที่อยู่นอกขอบเขต?

## ผลที่คาดหวัง

มี inventory ที่ระบุ target และขอบเขตได้ชัดเจน พร้อมแยกข้อมูลที่สังเกตได้จริงจากข้อมูลที่ยังไม่ทราบ

---

# Lab 2 — BlueToolkit Recon และ Connectivity Check

## เป้าหมาย

เรียนรู้ workflow ของ BlueToolkit ในส่วนตรวจสอบข้อมูลพื้นฐานและการเชื่อมต่อ โดยไม่เรียกใช้ exploit

## ขั้นตอน

1. เปิดโฟลเดอร์ repository ที่ติดตั้งไว้:

   ```bash
   cd BlueToolkit
   ```

2. ตรวจสอบวิธีเรียกใช้งานและตัวเลือกของเครื่องมือ:

   ```bash
   bluekit -h
   ```

   หากคำสั่งไม่พบ ให้กลับไปตรวจสอบขั้นตอน installation และ PATH ตาม README ทางการ อย่าเดาชื่อ executable หรือแก้ไข environment โดยไม่ตรวจสอบ

3. แสดงรายการการทดสอบที่เครื่องมือรู้จัก:

   ```bash
   bluekit -l
   ```

4. อ่านรายการผลลัพธ์และระบุรายการที่เป็น reconnaissance, connectivity check และ reporting ห้ามเลือก exploit ใน Lab นี้
5. ตรวจสอบ setup ของโมดูลที่จำเป็นตามเอกสารทางการ หากคำสั่งนี้มีในเวอร์ชันที่ติดตั้ง:

   ```bash
   bluekit -c
   ```

6. กำหนด Bluetooth address ของ target จาก Lab 1 เท่านั้น ให้ผู้สอนตรวจสอบ address ก่อนเรียกคำสั่งทุกครั้ง
7. ทดสอบการเชื่อมต่อ/การมองเห็น target โดยใช้ตัวเลือก connectivity check ที่เอกสารของเวอร์ชันติดตั้งระบุไว้ ตัวอย่างรูปแบบ CLI ที่ README ของ BlueToolkit แสดงคือ:

   ```bash
   sudo bluekit -t <AUTHORIZED_TARGET_ADDRESS> -ct
   ```

   แทน `<AUTHORIZED_TARGET_ADDRESS>` ด้วย address ของอุปกรณ์แล็บที่ได้รับอนุญาต ห้ามใช้ address ที่สุ่มพบจากพื้นที่สาธารณะ

8. ทำ reconnaissance ตามเอกสารทางการ เฉพาะเมื่อผู้สอนยืนยันว่าการตรวจสอบดังกล่าวเหมาะสมกับ target:

   ```bash
   sudo bluekit -t <AUTHORIZED_TARGET_ADDRESS> -r
   ```

9. บันทึก output ลงในไฟล์หลักฐานภายใน lab workspace ตามแนวทางการเก็บหลักฐานของผู้สอน อย่าเผยแพร่ address หรือข้อมูลระบุตัวอุปกรณ์จริงในรายงานสาธารณะ
10. อ่านผลลัพธ์และแยกข้อมูลเป็น:
    - **Observed:** มีข้อมูลใน output ยืนยัน
    - **Not observed:** การทดสอบไม่พบข้อมูล
    - **Not tested:** ไม่ได้ทดสอบ
    - **Inconclusive:** ผลไม่เพียงพอหรือสภาพแวดล้อมไม่รองรับ

## ข้อควรระวัง

- `-r` คือ reconnaissance ตาม CLI ของ BlueToolkit; ตรวจสอบรายละเอียดของเวอร์ชันที่ติดตั้งก่อนรัน
- อย่าใช้ `-e`, `--exploits` หรือคำสั่งรัน exploit ใน Lab นี้
- การไม่พบปัญหาไม่ได้แปลว่าอุปกรณ์ปลอดภัย
- ผลลัพธ์จากการทดสอบขึ้นกับ adapter, firmware, OS, ระยะห่าง และสภาพแวดล้อม RF

## ผลที่คาดหวัง

ผู้เรียนสามารถอธิบายผล recon ได้อย่างระมัดระวัง พร้อมบันทึกคำสั่ง เวอร์ชัน เวลา และข้อจำกัดของการทดสอบ

---

# Lab 3 — ตรวจสอบ Security Posture แบบไม่รบกวนอุปกรณ์

## เป้าหมาย

ประเมินการตั้งค่าและพฤติกรรมด้านความปลอดภัยจากหลักฐานที่สังเกตได้ โดยไม่พยายามเจาะระบบ

## Checklist

### A. Pairing และ Bonding

1. ใช้เฉพาะอุปกรณ์แล็บที่ผู้สอนเตรียมไว้
2. ตรวจสอบเอกสารหรือหน้าตั้งค่าของอุปกรณ์ว่าใช้ pairing mode แบบใด
3. สังเกตว่าผู้ใช้ต้องยืนยันการจับคู่หรือไม่ และการยืนยันเกิดขึ้นที่อุปกรณ์ใดบ้าง
4. บันทึกว่าเป็น Just Works, Numeric Comparison, Passkey Entry หรือวิธีอื่น เฉพาะเมื่อมีหลักฐานรองรับ
5. อย่าลองจับคู่กับอุปกรณ์ของบุคคลอื่น และอย่าพยายามข้ามการยืนยัน

### B. การเปิดเผยบริการ

1. ใช้ข้อมูลบริการ/UUID ที่ได้จากอุปกรณ์แล็บหรือเอกสารผู้ผลิต
2. ตรวจสอบว่าแต่ละบริการจำเป็นต่อการทำงานหรือไม่
3. บันทึกบริการที่เปิดเผยและเหตุผลทางธุรกิจที่จำเป็น
4. อย่าสรุปว่า service UUID ที่มองเห็นได้เป็นช่องโหว่ด้วยตัวมันเอง

### C. การเข้าถึงข้อมูล

1. ให้ผู้สอนระบุ characteristic หรือข้อมูลตัวอย่างที่ตั้งใจเปิดให้ Lab เข้าถึง
2. ตรวจสอบ permission จากเอกสารของอุปกรณ์หรือการแสดงผลจากเครื่องมือที่ได้รับอนุญาต
3. หากต้องอ่านค่า ให้ใช้เฉพาะค่าทดสอบที่ไม่ใช่ข้อมูลส่วนบุคคล และทำตามเอกสารทางการของอุปกรณ์
4. ห้ามเขียนค่าไปยัง characteristic ที่ควบคุม actuator, lock, motor, alarm, safety function หรือระบบจริง
5. บันทึกว่า read/write/notify ต้องการ authentication หรือ encryption หรือไม่ เฉพาะเมื่อพิสูจน์ได้จากหลักฐาน

### D. Privacy และการเปิดเผยตัวตน

1. สังเกตว่าอุปกรณ์ประกาศชื่อหรือข้อมูลผู้ผลิตหรือไม่
2. ตรวจสอบเอกสารว่าอุปกรณ์ใช้ address privacy/randomization หรือไม่ หากเครื่องมือและอุปกรณ์รองรับการตรวจสอบนี้
3. บันทึกเฉพาะข้อมูลที่จำเป็น หลีกเลี่ยงการเก็บข้อมูลของอุปกรณ์ที่อยู่นอกขอบเขต
4. อย่าพยายามติดตามอุปกรณ์หรือเชื่อมโยงตัวตนของผู้ใช้จริง

## แบบบันทึกผล

| Test ID | สิ่งที่ตรวจ | หลักฐาน | ผล | หมายเหตุ |
|---|---|---|---|---|
| SP-01 | Pairing interaction | | Pass / Finding / Inconclusive / N/A | |
| SP-02 | บริการที่เปิดเผย | | Pass / Finding / Inconclusive / N/A | |
| SP-03 | Permission ของข้อมูลทดสอบ | | Pass / Finding / Inconclusive / N/A | |
| SP-04 | การเปิดเผยชื่อ/ข้อมูลผู้ผลิต | | Pass / Finding / Inconclusive / N/A | |
| SP-05 | Address privacy ตามหลักฐานที่มี | | Pass / Finding / Inconclusive / N/A | |

> “Pass” หมายถึงผ่านเฉพาะเงื่อนไขที่ตรวจ ไม่ได้เป็นการรับรองความปลอดภัยทั้งหมด

---

# Lab 4 — วิเคราะห์ผลและจัดการ False Positives

## เป้าหมาย

เรียนรู้การตรวจสอบความน่าเชื่อถือของผลลัพธ์ก่อนประกาศว่าเป็นช่องโหว่

## ขั้นตอน

1. เลือกผลลัพธ์หนึ่งรายการจาก Lab 2 หรือ Lab 3
2. บันทึกเวอร์ชัน BlueToolkit, OS, adapter, target model/firmware, เวลา และเงื่อนไขการทดสอบ
3. ระบุข้อความ output ที่สนับสนุนข้อสรุป หลีกเลี่ยงการคัดลอกข้อมูลส่วนบุคคล
4. ตรวจสอบว่าเงื่อนไขการทดสอบครบถ้วนหรือไม่ เช่น target เปิดอยู่, adapter รองรับ, ระยะห่างเหมาะสม และไม่มี error
5. ทำซ้ำเฉพาะการตรวจสอบที่ปลอดภัยและได้รับอนุญาต เพื่อดูว่าผลสอดคล้องกันหรือไม่
6. หากผลลัพธ์คลุมเครือ ให้จัดเป็น **Inconclusive** แทนการสรุปว่า vulnerable
7. แยกประเด็นดังต่อไปนี้:
   - **Observation:** ข้อเท็จจริงที่เห็น
   - **Potential weakness:** จุดที่อาจมีความเสี่ยง
   - **Confirmed vulnerability:** มีหลักฐานเพียงพอและตรวจสอบตามเกณฑ์ที่กำหนดแล้ว
   - **Limitation:** สิ่งที่ยังทดสอบไม่ได้

## ตัวอย่าง

**Observation:** อุปกรณ์ปรากฏในรายการ discovery และแสดงชื่อผลิตภัณฑ์

**สรุปที่ไม่ถูกต้อง:** อุปกรณ์มีช่องโหว่ Bluetooth เพราะ discoverable

**สรุปที่เหมาะสม:** อุปกรณ์เปิดเผยข้อมูลระบุตัวผลิตภัณฑ์ระหว่าง discovery ควรตรวจสอบว่าการเปิดเผยดังกล่าวจำเป็นต่อการใช้งานหรือไม่ และมีผลกระทบต่อ privacy ตาม threat model ขององค์กรหรือไม่

---

# Lab 5 — สร้างรายงานผลการทดสอบ

## เป้าหมาย

ส่งมอบรายงานที่ทีมพัฒนาหรือทีมดูแลผลิตภัณฑ์นำไปแก้ไขได้จริง

## ขั้นตอน

1. สรุป scope และรายการอุปกรณ์ที่ได้รับอนุญาต
2. บันทึกวิธีทดสอบ เวอร์ชันเครื่องมือ และข้อจำกัด
3. สร้าง finding เฉพาะประเด็นที่มีหลักฐาน
4. ให้คะแนนความเสี่ยงโดยพิจารณา:
   - ผลกระทบต่อ Confidentiality, Integrity และ Availability
   - ต้องอยู่ใกล้อุปกรณ์หรือไม่
   - ต้องจับคู่หรือผ่าน authentication ก่อนหรือไม่
   - ต้องอาศัย interaction จากผู้ใช้หรือไม่
   - ความเป็นไปได้ในการทำซ้ำและขอบเขตผลกระทบ
5. เสนอ remediation ที่สอดคล้องกับสาเหตุ
6. แยก finding ที่ยืนยันแล้วออกจากข้อเสนอแนะและรายการที่ยังทดสอบไม่ได้
7. ให้ผู้สอนตรวจรายงานก่อนส่งมอบ

## Template: Finding

### [F-XX] ชื่อประเด็น

- **Severity:** Critical / High / Medium / Low / Informational
- **Affected asset:** Lab Asset ID
- **Status:** Confirmed / Potential / Inconclusive
- **Description:** อธิบายสิ่งที่ตรวจพบ
- **Evidence:** Output หรือภาพหน้าจอที่ตัดข้อมูลอ่อนไหวออกแล้ว
- **Impact:** ผลกระทบที่มีเหตุผลรองรับ
- **Reproduction summary:** ขั้นตอนที่ปลอดภัยและอยู่ในขอบเขตที่ใช้ยืนยัน
- **Limitations:** ข้อจำกัดของการทดสอบ
- **Recommendation:** วิธีแก้ไข
- **Retest criteria:** เงื่อนไขที่ใช้ตรวจซ้ำหลังแก้ไข

## ตัวอย่างแนวทางแก้ไขทั่วไป

- จำกัดบริการ Bluetooth ให้เหลือเท่าที่จำเป็น
- กำหนดให้ข้อมูลหรือคำสั่งที่มีความอ่อนไหวต้องผ่าน authentication และ encryption ตามความสามารถของแพลตฟอร์ม
- หลีกเลี่ยงการใช้ pairing mode ที่ไม่มีการยืนยันตัวตน หาก threat model ต้องการ MITM protection
- ลดการเปิดเผยข้อมูลระบุตัวผลิตภัณฑ์ที่ไม่จำเป็น
- อัปเดต firmware และ Bluetooth stack จากผู้ผลิต
- ทดสอบ regression หลังแก้ไข โดยใช้ target และเกณฑ์เดียวกับการทดสอบครั้งแรก

---

## 4. เกณฑ์ประเมินผู้เรียน

| รายการ | คะแนน |
|---|---:|
| เตรียมสภาพแวดล้อมและตรวจสอบ adapter | 10 |
| จัดทำ inventory และยืนยัน scope | 15 |
| ใช้ BlueToolkit สำหรับ recon/check อย่างถูกต้อง | 25 |
| วิเคราะห์หลักฐานและอธิบายข้อจำกัด | 20 |
| จัดทำรายงาน finding และ remediation | 20 |
| ปฏิบัติตามขอบเขตและกติกาความปลอดภัย | 10 |
| **รวม** | **100** |

### เกณฑ์ผ่านที่แนะนำ

- คะแนนรวมอย่างน้อย 70/100
- ต้องไม่ทดสอบอุปกรณ์นอกขอบเขต
- ต้องไม่รัน exploit หรือทดสอบที่อาจทำให้บริการหยุดชะงักใน Lab นี้
- ต้องระบุข้อจำกัดและไม่กล่าวอ้างเกินหลักฐาน

## 5. สิ่งที่ผู้เรียนต้องส่ง

- `target-inventory.md` — รายการอุปกรณ์ใน scope
- `lab-notes.md` — คำสั่งที่ใช้ เวอร์ชัน และบันทึกผล
- `evidence/` — หลักฐานที่ตัดข้อมูลอ่อนไหวแล้ว
- `assessment-report.md` — สรุปผลและ remediation
- `ret test-plan.md` — เกณฑ์ทดสอบซ้ำหลังแก้ไข

## 6. เอกสารอ้างอิง

- BlueToolkit repository: https://github.com/sgxgsx/BlueToolkit
- BlueToolkit Wiki: https://github.com/sgxgsx/BlueToolkit/wiki
- BlueToolkit research paper (USENIX WOOT 2025): https://www.usenix.org/system/files/woot25-zubkov.pdf

**คำเตือน:** ความสามารถบางส่วนของ BlueToolkit ต้องใช้ hardware เฉพาะ และการทดสอบ exploit อาจรบกวนหรือทำให้อุปกรณ์เป้าหมายหยุดทำงานได้ คู่มือนี้จึงจำกัดการปฏิบัติไว้ที่ recon, connectivity check, security posture review และ reporting แบบไม่รบกวนอุปกรณ์


---
# ส่วนเพิ่มเติม: Caeruleus + ESP32 BLE Security Labs (ปรับปรุง 2026-10-29)

> **ขอบเขต:** ใช้เฉพาะ ESP32 DemoSensor หรือ BLE development board ของผู้สอนที่ได้รับอนุญาต หลีกเลี่ยงการสแกนต่อเนื่องในพื้นที่สาธารณะ การจับคู่กับอุปกรณ์อื่น การส่งคำสั่งที่เปลี่ยนสถานะอุปกรณ์จริง การ fuzzing และการทดสอบ denial-of-service
>
> **การรองรับ:** Caeruleus ใช้ Linux + BlueZ และทำงานเป็น BLE Central เท่านั้น ไม่ใช่เครื่องมือ sniffing หรือ Bluetooth Classic scanner และไม่ได้ทำ active MITM; ใช้ BlueToolkit สำหรับส่วน Bluetooth Classic / Recon ตามความเหมาะสม

## Lab 0 (เพิ่มเติม) — ติดตั้ง Caeruleus

### อุปกรณ์
- Linux (Ubuntu/Debian หรือ Kali) พร้อม USB BLE adapter ที่ BlueZ มองเห็น
- ESP32 development board ที่แฟลช firmware ทดสอบไว้แล้ว (ตัวอย่างชื่อ `DemoSensor`)
- USB cable และ Android + nRF Connect (ทางเลือก)

### ขั้นตอน

1. ตรวจสอบ Bluetooth service และ adapter:

   ```bash
   bluetoothctl list
   bluetoothctl show
   systemctl status bluetooth --no-pager
   ```

2. ตรวจสอบ Go ที่ติดตั้งไว้ และเทียบกับ `go.mod` ของ Caeruleus:

   ```bash
   go version
   ```

3. ติดตั้งตาม README ของโครงการ (ต้องมี Go toolchain ที่รองรับ):

   ```bash
   go install github.com/praetorian-inc/caeruleus/cmd/caeruleus@latest
   ```

4. หาก shell หา binary ไม่พบ ให้เรียกจากตำแหน่ง Go bin โดยตรง (หรือเพิ่ม PATH):

   ```bash
   "$(go env GOPATH)/bin/caeruleus" --help
   export PATH="$(go env GOPATH)/bin:$PATH"
   ```

5. ตรวจสอบความพร้อม:

   ```bash
   caeruleus doctor
   caeruleus recipes
   ```

**ผลที่คาดหวัง:** `doctor` แสดงสถานะ Bluetooth service และ adapter พร้อมข้อความ OK/WARN/FAIL; หาก FAIL ให้แก้ไขสภาพแวดล้อมก่อนทดสอบต่อ อย่าสรุปว่า target มีช่องโหว่จาก error ของ adapter

## Lab 1 (เพิ่มเติม) — BLE Discovery ด้วย Caeruleus

1. เปิด ESP32 DemoSensor ที่ผู้สอนเตรียมไว้ และยืนยันชื่อหรือรหัสอุปกรณ์กับผู้สอน
2. สำรวจอุปกรณ์ในบริเวณ Lab:

   ```bash
   caeruleus scan
   ```

3. ระบุเฉพาะอุปกรณ์ DemoSensor ใน scope จากชื่อและรายละเอียดที่ผู้สอนยืนยัน ห้ามเชื่อมต่อกับอุปกรณ์ที่ไม่รู้จัก
4. บันทึก Address, Address Type, RSSI, ชื่ออุปกรณ์, เวลา และรุ่น firmware หากทราบ
5. สร้างไฟล์ `target-inventory.md` และกำหนดตัวแปรสำหรับ Lab ต่อไป:

   ```bash
   export TARGET='AA:BB:CC:DD:EE:FF'
   ```

   แทนค่าด้วย address ของ ESP32 ที่ได้รับอนุญาตจริง

**ผลที่คาดหวัง:** พบ DemoSensor และมี inventory พร้อมหลักฐานว่า target อยู่ในขอบเขต หากไม่พบ ให้ตรวจสอบการเปิด advertising, ระยะห่าง, adapter และความสามารถ BLE

## Lab 3 — GATT Enumeration และการอ่านข้อมูลทดสอบ

### Scenario

ผู้สอนเตรียม ESP32 เป็น BLE peripheral ที่มี GATT service และ characteristic สองชนิด:

- `PublicTemperature`: ข้อมูลอุณหภูมิจำลอง อนุญาตให้อ่านได้โดยไม่ต้อง pair
- `ProtectedConfig`: ข้อความจำลองที่ควรต้องผ่าน authentication/encryption ก่อนอ่าน เช่น `LAB-CONFIG-ONLY` **ห้ามใช้รหัสผ่านหรือ token จริง**

### ขั้นตอน

1. ตรวจสอบคำสั่งที่ติดตั้ง:

   ```bash
   caeruleus enumerate --help
   caeruleus read --help
   ```

2. สำรวจ GATT database ของ ESP32 โดยยังไม่อ่านค่าทุก characteristic:

   ```bash
   caeruleus enumerate -b "$TARGET"
   ```

3. จด service UUID, characteristic UUID, properties และ handle ที่รายงานได้ (handle เปลี่ยนได้ตาม firmware และ BlueZ)
4. ให้ผู้สอนยืนยัน handle ของ `PublicTemperature` ก่อนทดสอบอ่าน:

   ```bash
   export PUBLIC_HANDLE='0x0025'
   caeruleus read -b "$TARGET" -a "$PUBLIC_HANDLE"
   ```

   `0x0025` เป็น **ตัวอย่างเท่านั้น** ต้องแทนด้วย handle ที่ค้นพบจริงจากอุปกรณ์ Lab

5. หากผู้สอนยืนยันว่า characteristic ทั้งหมดใน target มีเฉพาะข้อมูลจำลอง สามารถอ่านพร้อมกันได้:

   ```bash
   caeruleus enumerate -b "$TARGET" --values
   ```

6. ส่งออกโครงสร้างเป็น JSON:

   ```bash
   mkdir -p evidence
   caeruleus enumerate -b "$TARGET" -o json > evidence/gatt-enumeration.json
   ```

### วิเคราะห์ผล

- การพบ service/UUID ไม่เท่ากับพบช่องโหว่
- การอ่าน `PublicTemperature` ได้โดยไม่ pair เป็นพฤติกรรมที่ยอมรับได้ หากออกแบบให้เป็นข้อมูลสาธารณะ
- ถ้าข้อมูลที่กำหนดว่า protected ถูกอ่านได้โดยไม่ pair อาจเป็น finding ด้าน access control ต้องตรวจสอบ requirement และทำซ้ำกับ firmware ที่ควบคุมได้
- Error เช่น connection failed, handle not found หรือ insufficient authentication ต้องแยกสาเหตุก่อนสรุป

## Lab 4 — Authentication และ Encryption Assessment

### Scenario

ใช้ ESP32 ที่มี characteristic จำลอง `ProtectedConfig` และกำหนด policy ไว้ชัดเจนว่าไม่ควรเปิดอ่านโดยไม่มีการยืนยันตัวตน/การเข้ารหัส

### ขั้นตอน

1. เริ่มจาก ESP32 ที่ยังไม่ได้ pair กับ Linux adapter ของผู้เรียน หากเคย pair มาก่อน ให้ผู้สอนรีเซ็ตสถานะ bonding ของ **อุปกรณ์ Lab เท่านั้น** ด้วยวิธีของ firmware ที่จัดเตรียมไว้
2. ดูตัวเลือก assessment ของ Caeruleus เวอร์ชันที่ติดตั้ง:

   ```bash
   caeruleus assess check-auth --help
   caeruleus assess encryption --help
   ```

3. ตรวจสอบ unauthenticated access แบบ read-only (อย่าเปิดตัวเลือก `--probe-write`):

   ```bash
   caeruleus assess check-auth -b "$TARGET" -o json > evidence/auth-assessment.json
   ```

4. ตรวจสอบข้อกำหนด encryption ตามที่เครื่องมือรองรับ:

   ```bash
   caeruleus assess encryption -b "$TARGET" -o json > evidence/encryption-assessment.json
   ```

5. อ่านไฟล์ JSON และตรวจสอบ `summary`, `findings`, `severity`, `handle`, `uuid`, `evidence` ตาม schema ที่เครื่องมือสร้างจริง ไม่สมมติว่าต้องมีทุก field
6. เทียบกับ policy ของ firmware: characteristic ที่กำหนดให้ protected ควรปฏิเสธการเข้าถึงก่อนผ่านเงื่อนไขที่กำหนด
7. ห้ามรัน `assess wwr`, `fuzz write`, `raw-write` หรือการทดสอบที่ส่งข้อมูลจำนวนมากในห้องเรียนทั่วไป

### ตัวอย่างตารางสรุป (ผลลัพธ์สมมติ ไม่ใช่ผลจากเครื่องมือจริง)

| Firmware | ProtectedConfig ก่อน pair | ผลวิเคราะห์ |
|---|---|---|
| Insecure | Read succeeded | Potential unauthorized data exposure |
| Secure | Read denied / authentication required | Control enforced for this test |

**หมายเหตุ:** การ pair ไม่ได้ยืนยันโดยอัตโนมัติว่ามี MITM protection; ต้องตรวจสอบ security mode, association model และ policy ของอุปกรณ์เพิ่มเติมก่อนสรุป

## Lab 5 — เปรียบเทียบ Insecure / Secure ESP32

### เตรียม firmware โดยผู้สอน

สร้าง firmware สอง profile ซึ่งมี service/characteristic และข้อมูลจำลองชุดเดียวกัน:

**Profile A — Insecure (สำหรับห้อง Lab เท่านั้น)**
- Advertising ชื่อ `DemoSensor-LAB-A`
- `PublicTemperature` อ่านได้โดยไม่ pair
- `ProtectedConfig` อ่านได้โดยไม่ pair ทั้งที่ requirement กำหนดให้ต้องป้องกัน

**Profile B — Secure**
- Advertising ชื่อ `DemoSensor-LAB-B`
- `PublicTemperature` อ่านได้โดยไม่ pair
- `ProtectedConfig` ต้องผ่าน security requirement ที่กำหนด เช่น authenticated pairing และ encryption (เลือกตาม SDK และความสามารถของบอร์ด)

> แนะนำให้ผู้สอนแฟลช firmware และทดสอบล่วงหน้า การตั้งค่า security ของ ESP-IDF แตกต่างกันตาม Bluetooth stack และเวอร์ชัน SDK อย่าคัดลอก API โดยไม่ตรวจสอบ documentation ของรุ่นที่ใช้

### ขั้นตอนผู้เรียน

1. ทดสอบ Profile A โดยทำ Lab 3 และ Lab 4 กับ target ที่ผู้สอนกำหนด
2. บันทึกผลอ่าน `ProtectedConfig` และ JSON assessment
3. ให้ผู้สอนเปลี่ยนเป็น Profile B แล้วตรวจสอบ target address ใหม่ (อาจเปลี่ยนได้)
4. ทดสอบคำสั่งเดิมอีกครั้งกับ Profile B
5. เปรียบเทียบผล: การอ่านที่ไม่ควรอนุญาตต้องถูกปฏิเสธใน Profile B ก่อนยืนยันตัวตน
6. สรุปว่า policy ใดได้รับการบังคับใช้ และส่วนใดยังไม่ได้ตรวจสอบ เช่น MITM resistance หรือ key management

### คำถาม

- เพราะเหตุใด characteristic ที่มี `read` property จึงยังอาจถูกป้องกันด้วย authentication/encryption ได้?
- หาก Profile B ปฏิเสธการอ่าน ถือว่าปลอดภัยทุกกรณีหรือไม่?
- ทำไมต้องควบคุม firmware, adapter และ target state ให้ใกล้เคียงกันเมื่อทำ retest?

## Lab 7 — JSON Evidence และ Reporting

1. สร้าง directory แยกหลักฐาน Profile A/B:

   ```bash
   mkdir -p evidence/insecure evidence/secure
   ```

2. รันคำสั่ง assessment แบบ read-only ที่ผ่านการทดสอบแล้ว โดยเปลี่ยน target ตาม firmware profile:

   ```bash
   caeruleus assess check-auth -b "$TARGET" -o json > evidence/insecure/check-auth.json
   ```

   เมื่อติดตั้ง Profile B แล้ว ให้เปลี่ยน output เป็น `evidence/secure/check-auth.json`

3. ใช้ `python3 -m json.tool` ตรวจสอบว่าไฟล์ JSON อ่านได้:

   ```bash
   python3 -m json.tool evidence/insecure/check-auth.json > /dev/null
   ```

4. เปิดผลลัพธ์และบันทึกเฉพาะข้อค้นพบที่มีหลักฐาน:
   - Affected asset และ firmware profile
   - UUID / handle ที่เกี่ยวข้อง
   - ข้อกำหนดด้านการเข้าถึงที่คาดหวัง
   - ผลทดสอบจริงและข้อความ error/success
   - Severity พร้อมเหตุผล
   - Remediation และ retest criteria
5. เทียบผลกับ BlueToolkit reconnaissance จาก Lab 2 โดยแยกข้อจำกัดของเครื่องมือ (BlueToolkit รองรับ Classic/BLE; Caeruleus เน้น BLE GATT)
6. ตรวจสอบรายงานร่วมกับผู้สอนก่อนส่ง และปกปิดข้อมูลระบุตัวอุปกรณ์ที่ไม่จำเป็น

### Template สรุป Finding

```markdown
## F-01: Protected BLE characteristic accessible without authentication
- Asset: DemoSensor-LAB-A
- Scope: Authorized classroom ESP32
- Requirement: ProtectedConfig requires authenticated encrypted access
- Observation: [paste actual read result]
- Evidence: evidence/insecure/check-auth.json
- Impact: Unauthorized exposure of simulated configuration data
- Confidence: [Confirmed / Potential / Inconclusive]
- Remediation: Enforce GATT permission and security policy on ProtectedConfig
- Retest: Before authentication, read must be rejected; authorized session must follow intended policy
```



## Lab 6 — nRF Connect for Mobile + nRF52840

### เป้าหมาย

ใช้ nRF Connect for Mobile เป็น BLE client เพื่อค้นหาและเชื่อมต่อกับบอร์ด nRF52840 ที่ผู้สอนเตรียมไว้ จากนั้นตรวจสอบ GATT services/characteristics และเปรียบเทียบกับผลจาก Caeruleus บน Linux

### อุปกรณ์และการเตรียมก่อน Lab

- Nordic nRF52840 DK (แนะนำ PCA10056) พร้อมสาย USB หรือ nRF52840 board ที่ผู้สอนยืนยันว่าใช้งาน BLE peripheral ได้
- โทรศัพท์ Android หรือ iOS ที่ติดตั้ง nRF Connect for Mobile
- Firmware BLE peripheral ที่เตรียมไว้ล่วงหน้า เช่น demo `DemoSensor-nRF52840` หรือ Nordic Bluetooth peripheral sample ที่เหมาะสม
- หากใช้ custom DemoSensor firmware ให้กำหนดข้อมูลทดสอบเท่านั้น เช่น `PublicTemperature` และ `ProtectedConfig`; ห้ามใช้รหัสผ่าน, token หรือข้อมูลส่วนบุคคลจริง

> **สำคัญ:** nRF Connect for Mobile เป็นแอปบนโทรศัพท์สำหรับ BLE scanning/connection/GATT exploration ไม่ใช่ตัว packet sniffer โดยตรง บอร์ด nRF52840 ต้องรัน firmware ที่ทำหน้าที่เป็น BLE peripheral ก่อนจึงจะปรากฏให้เชื่อมต่อได้

### A. เตรียมบอร์ด nRF52840

1. ตรวจสอบรุ่นบอร์ดและ revision ที่พิมพ์บนบอร์ดหรือกล่อง และเชื่อมต่อ USB
2. ใช้ firmware ที่ผู้สอนเตรียมและทดสอบไว้แล้ว หากต้องแฟลชด้วยตนเอง ให้ใช้ nRF Connect for Desktop Programmer และ firmware ที่ตรงกับบอร์ด/SDK รุ่นนั้น
3. เปิดบอร์ดและยืนยันว่า firmware เริ่ม BLE advertising โดยชื่ออุปกรณ์ควรระบุชัดเจน เช่น `DemoSensor-nRF52840-LAB`
4. หากใช้ sample firmware มาตรฐานแทน custom DemoSensor ให้จดชื่อ service/characteristics ที่ sample นั้นให้บริการไว้ก่อนเริ่ม และปรับขั้นตอนอ่านค่าให้ตรงกับ sample
5. บันทึก firmware version, board model และชื่อ advertising ลงใน `target-inventory.md`

### B. ติดตั้งและตั้งค่า nRF Connect for Mobile

1. ติดตั้งแอปจากหน้าอย่างเป็นทางการ: https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile
2. เปิด Bluetooth ของโทรศัพท์
3. บน Android ให้ให้สิทธิ์ที่แอปขอสำหรับ BLE scanning และเปิด Location Services หากระบบปฏิบัติการ/รุ่นโทรศัพท์ต้องการสำหรับการสแกน
4. เปิดแอปและเลือกแท็บ **Scanner**
5. กด **Scan** แล้วรอให้พบ `DemoSensor-nRF52840-LAB` (หรือชื่อที่ผู้สอนกำหนด)
6. เลือกเฉพาะอุปกรณ์ใน scope ที่ผู้สอนยืนยัน ห้ามเชื่อมต่อกับอุปกรณ์ใกล้เคียงที่ไม่รู้จัก
7. บันทึกชื่อ, address ที่แสดง, RSSI และ advertising data ที่จำเป็น โดยหลีกเลี่ยงการเก็บข้อมูลของอุปกรณ์อื่น

### C. เชื่อมต่อและสำรวจ GATT

1. กด **Connect** ที่ target ของ Lab
2. รอให้แอปทำ service discovery และแสดงรายการ services/characteristics
3. บันทึก UUID, properties (เช่น Read, Write, Notify) และ descriptor ที่แอปแสดง
4. หากใช้ custom DemoSensor firmware ให้หา characteristic `PublicTemperature` จาก UUID ที่ผู้สอนแจกไว้ แล้วกด **Read**
5. ตรวจสอบว่าค่าที่อ่านได้เป็นข้อมูลจำลองตามที่คาดไว้ เช่น อุณหภูมิห้องทดลอง
6. หา `ProtectedConfig` และตรวจสอบ properties/คำอธิบายที่ผู้สอนให้ไว้ **อย่าทดลองเขียนค่า** และอย่าเปิดเผยข้อมูลที่ไม่ใช่ข้อมูล Lab
7. หาก target ใช้ sample firmware มาตรฐานที่ไม่มี `PublicTemperature`/`ProtectedConfig` ให้ตรวจสอบ characteristic ที่มีอยู่ตาม worksheet ของผู้สอน และอย่าสรุปว่าการไม่มี characteristic เหล่านั้นเป็นช่องโหว่
8. เมื่อจบให้กด **Disconnect**

### D. เปรียบเทียบผลกับ Caeruleus

1. กลับไปยัง Linux ที่ติดตั้ง Caeruleus แล้วสแกน target ที่ได้รับอนุญาต:

   ```bash
   caeruleus scan
   ```

2. กำหนด address ที่ตรวจสอบแล้วของบอร์ด:

   ```bash
   export TARGET='AA:BB:CC:DD:EE:FF'
   ```

   แทนค่าตัวอย่างด้วย address ที่แสดงจากอุปกรณ์ Lab และตรวจสอบอีกครั้งก่อนดำเนินการ

3. สำรวจ GATT ด้วย Caeruleus:

   ```bash
   caeruleus enumerate -b "$TARGET"
   ```

4. เปรียบเทียบ service UUID, characteristic UUID และ properties กับข้อมูลที่ nRF Connect for Mobile แสดง
5. หากข้อมูลไม่ตรงกัน ให้ตรวจสอบว่าเชื่อมต่อกับ target เดียวกันหรือไม่, firmware ตรงกันหรือไม่, และแอป/BlueZ cache หรือ address privacy มีผลหรือไม่
6. เก็บผลลัพธ์ที่เกี่ยวข้องโดยปกปิดข้อมูลที่ไม่จำเป็น และไม่บันทึกข้อมูลของอุปกรณ์ที่อยู่นอก scope

### Expected Results

- nRF Connect for Mobile พบอุปกรณ์ nRF52840 ที่กำหนดไว้
- สามารถเชื่อมต่อและแสดง GATT database ได้ตาม firmware ที่แฟลช
- อ่านได้เฉพาะข้อมูลทดสอบที่ firmware อนุญาต
- ข้อมูล GATT หลักสอดคล้องกับ Caeruleus เมื่อใช้ target และ firmware เดียวกัน
- ผู้เรียนอธิบายได้ว่า **การมองเห็น service หรือ characteristic ไม่ได้พิสูจน์ว่าเป็นช่องโหว่**; ต้องเทียบกับ requirement และ access-control policy

### Troubleshooting

| ปัญหา | วิธีตรวจสอบ |
|---|---|
| ไม่พบอุปกรณ์ | ตรวจสอบว่า firmware ทำ BLE advertising, เปิด Bluetooth, ให้สิทธิ์ scanning และอยู่ในระยะใกล้ |
| พบอุปกรณ์แต่ Connect ไม่ได้ | ตรวจสอบว่าเป็น connectable advertising, ไม่มี client อื่นจับการเชื่อมต่อค้าง และรีสตาร์ตเฉพาะอุปกรณ์ Lab หากจำเป็น |
| GATT ไม่แสดงตามคาด | ตรวจสอบ firmware build, UUID และ cache; disconnect/reconnect แล้วลองใหม่ |
| Read ถูกปฏิเสธ | อาจเป็นผลที่ถูกต้องสำหรับ characteristic ที่ต้องใช้ security; บันทึก error และตรวจ policy ก่อนสรุป |
| Caeruleus เห็นข้อมูลไม่ตรงกับแอป | ยืนยัน address/target/firmware, ตรวจ cache และตรวจว่ามี address rotation หรือไม่ |

### Worksheet

| รายการ | ผลที่พบ |
|---|---|
| Board model / revision | |
| Firmware / sample | |
| Advertising name | |
| Address / address type | |
| Service UUIDs | |
| Characteristic UUIDs และ properties | |
| ผล Read ของข้อมูลทดสอบ | |
| เปรียบเทียบกับ Caeruleus | |
| ข้อจำกัด / สิ่งที่ยังไม่ได้ทดสอบ | |

### ตัวเลือกเสริม — ใช้ nRF52840 เป็น BLE Packet Sniffer

หากต้องการสาธิต packet capture เพิ่ม ให้ใช้ **nRF52840 DK/Dongle อีกตัวหนึ่ง** และแฟลช firmware ของ nRF Sniffer for Bluetooth LE ตามคู่มือ Nordic จากนั้นเชื่อมต่อ Wireshark และจับ traffic ระหว่างโทรศัพท์กับบอร์ด peripheral ในห้อง Lab เท่านั้น

1. อ่านคู่มือทางการก่อน: https://docs.nordicsemi.com/bundle/nrfutil/page/nrfutil-ble-sniffer/ และ https://www.nordicsemi.com/Products/Development-tools/nRF-Sniffer-for-Bluetooth-LE
2. ใช้ nRF Connect for Desktop Programmer หรือขั้นตอนที่เอกสารเวอร์ชันปัจจุบันระบุ เพื่อแฟลช firmware sniffer ที่ตรงกับรุ่นบอร์ด
3. ทำตามคู่มือ Nordic สำหรับ `nrfutil ble-sniffer bootstrap` และการตั้งค่า Wireshark extcap ตาม OS ที่ใช้
4. เริ่ม capture แล้วเชื่อมต่อโทรศัพท์กับ peripheral ที่กำหนดไว้ จากนั้นหยุด capture และบันทึกไฟล์ใน `evidence/`
5. ตรวจสอบ metadata และ packet flow ตามวัตถุประสงค์ Lab เท่านั้น อย่าพยายามถอดรหัสหรือสกัดข้อมูลจากอุปกรณ์/การเชื่อมต่อที่ไม่ได้รับอนุญาต

> บอร์ดที่แฟลชด้วย firmware sniffer จะทำหน้าที่เป็น sniffer แทนการทำงานเป็น peripheral application จึงต้องใช้บอร์ดอีกตัวเป็น target (หรือ reflash กลับเป็น peripheral firmware หลัง Lab) ดูรายการอุปกรณ์ที่รองรับและข้อจำกัดรุ่นล่าสุดในเอกสาร Nordic ก่อนจัดเตรียมอุปกรณ์


## Troubleshooting เฉพาะ Caeruleus

| ปัญหา | แนวทางตรวจสอบ |
|---|---|
| `caeruleus: command not found` | ตรวจสอบ `$(go env GOPATH)/bin` และ PATH |
| `doctor` พบ adapter error | ตรวจสอบ BlueZ service, rfkill, USB passthrough และสิทธิ์ |
| Scan ไม่พบ DemoSensor | ตรวจสอบ advertising, ระยะห่าง และว่า adapter ใช้ BLE ได้ |
| Enumerate แล้วไม่พบ characteristic | ตรวจสอบ target, firmware, BlueZ cache และ connection state; อาจต้องใช้วิธี raw enumeration ที่ต้องมีสิทธิ์เพิ่มตาม README |
| Read แล้ว `Insufficient Authentication` | อาจเป็นผลที่คาดหวังสำหรับ protected characteristic; ตรวจสอบ policy |
| ผลลัพธ์ไม่ตรงกันระหว่างสองรอบ | ตรวจสอบ bonding state, firmware, address rotation, GATT cache และ adapter |

## อ้างอิงคำสั่งและข้อจำกัด

- Caeruleus README: https://github.com/praetorian-inc/caeruleus/blob/main/README.md
- Caeruleus Wiki: https://github.com/praetorian-inc/caeruleus/wiki
- BlueToolkit README: https://github.com/sgxgsx/BlueToolkit/blob/main/README.md
- Espressif BLE overview: https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/ble/ble-introduction.html
- nRF Connect for Mobile: https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-mobile
- nRF Sniffer for Bluetooth LE: https://www.nordicsemi.com/Products/Development-tools/nRF-Sniffer-for-Bluetooth-LE
- Nordic nRF Sniffer setup guide: https://academy.nordicsemi.com/topic/test/

> คำสั่ง Caeruleus ข้างต้นอ้างอิงจาก README ของโครงการ ตรวจสอบ `--help` ของเวอร์ชันที่ติดตั้งก่อนเริ่มสอนจริง และทดลองบน ESP32 ของผู้สอนล่วงหน้า เนื่องจากผลการทดสอบขึ้นกับ firmware, pairing state, BlueZ และ BLE adapter
