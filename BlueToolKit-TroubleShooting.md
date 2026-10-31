# BlueToolkit บน Kali Linux: ขั้นตอนตรวจสอบและแก้ไขปัญหา Python Modules

เอกสารนี้สรุปวิธีตรวจสอบและแก้ไขปัญหาการติดตั้ง BlueToolkit บน Kali Linux
โดยเน้นกรณีที่พบข้อความ `ModuleNotFoundError` เช่น `tqdm`, `yaml` และ `psutil`

> **สาเหตุที่พบบ่อย:** BlueToolkit ถูกเรียกใช้งานผ่าน Python virtual environment
> แต่ติดตั้งไลบรารีไว้ใน Python ของระบบ (system Python) แทนที่จะติดตั้งใน virtual
> environment ของ BlueToolkit

------------------------------------------------------------------------

## 1. ตรวจสอบตำแหน่งติดตั้งและ Python environment

จากการตั้งค่าที่พบ ตัวเรียกใช้งาน BlueToolkit อยู่ที่:

``` text
/usr/local/bin/bluekit
```

และใช้ virtual environment ที่:

``` text
/usr/share/BlueToolkit/.venv
```

ตรวจสอบไฟล์เรียกใช้งาน:

``` bash
cat /usr/local/bin/bluekit
```

ควรพบการกำหนด `VENV_PATH` และการเรียกใช้งาน executable จาก `.venv` ลักษณะนี้:

``` bash
VENV_PATH="/usr/share/BlueToolkit/.venv"
source "$VENV_PATH/bin/activate"
exec "$VENV_PATH/bin/bluekit" "$@"
```

ตรวจสอบ Python และ pip ใน environment ดังกล่าว:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python --version
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip --version
```

ตรวจสอบว่า virtual environment มีอยู่จริง:

``` bash
ls -la /usr/share/BlueToolkit/.venv
```

**ข้อควรจำ:** ใช้คำสั่ง `.../.venv/bin/python -m pip` ในการติดตั้งแพ็กเกจ
เพื่อให้แน่ใจว่าติดตั้งลง environment ที่ BlueToolkit ใช้งาน ไม่ใช่ system Python

------------------------------------------------------------------------

## 2. แก้ไข ModuleNotFoundError ทีละรายการ

ตัวอย่างข้อผิดพลาดที่พบและแพ็กเกจที่ต้องติดตั้ง:

  ข้อความผิดพลาด                 ชื่อแพ็กเกจที่ติดตั้ง
  ---------------------------- ---------------
  `No module named 'tqdm'`     `tqdm`
  `No module named 'yaml'`     `PyYAML`
  `No module named 'psutil'`   `psutil`

ติดตั้งด้วยคำสั่งต่อไปนี้:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install tqdm
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install PyYAML
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install psutil
```

ชื่อแพ็กเกจที่ใช้ติดตั้งอาจต่างจากชื่อที่ใช้ import เช่น ติดตั้ง `PyYAML` แต่ใน Python ใช้
`import yaml`

ตรวจสอบว่าแต่ละโมดูล import ได้:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -c "import tqdm; print('tqdm OK')"
sudo /usr/share/BlueToolkit/.venv/bin/python -c "import yaml; print('yaml OK')"
sudo /usr/share/BlueToolkit/.venv/bin/python -c "import psutil; print('psutil OK')"
```

------------------------------------------------------------------------

## 3. ตรวจสอบไฟล์ Dependencies ของโปรเจกต์

การติดตั้งทีละโมดูลอาจทำให้พบข้อผิดพลาดของโมดูลอื่นต่อเนื่องกัน
ควรตรวจสอบก่อนว่าโปรเจกต์มีไฟล์กำหนด dependencies หรือไม่

``` bash
cd /usr/share/BlueToolkit

find . -maxdepth 2 -type f \
  \( -name "requirements*.txt" -o -name "pyproject.toml" -o -name "setup.py" -o -name "setup.cfg" \)
```

### กรณีพบ `requirements.txt`

ตรวจสอบไฟล์ก่อน:

``` bash
cat requirements.txt
```

หากเป็นไฟล์ requirements ของโปรเจกต์นี้ ให้ติดตั้งด้วย Python ของ virtual
environment:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install \
  -r /usr/share/BlueToolkit/requirements.txt
```

หากไฟล์อยู่ในไดเรกทอรีอื่น ให้เปลี่ยน path ให้ตรงกับตำแหน่งจริง

### กรณีไม่พบ `requirements.txt`

ตรวจสอบไฟล์และเอกสารติดตั้งของ repository เช่น `README.md`, `pyproject.toml`,
`setup.py` หรือสคริปต์ `install.sh` ก่อนตัดสินใจติดตั้งแพ็กเกจเพิ่มเติม อย่าสมมติรายชื่อ
dependencies จากข้อความ error เพียงอย่างเดียว

------------------------------------------------------------------------

## 4. ตรวจสอบความสมบูรณ์ของ Dependencies

หลังติดตั้งแพ็กเกจแล้ว ตรวจสอบ dependency conflicts:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip check
```

ตรวจสอบรายการแพ็กเกจที่ติดตั้ง:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip list
```

หาก `pip check` รายงาน conflict ให้อ่านชื่อแพ็กเกจและเวอร์ชันที่ขัดแย้งกัน
แล้วแก้ตามข้อกำหนดของ BlueToolkit อย่าอัปเกรดทุกแพ็กเกจแบบไม่ตรวจสอบ
เพราะอาจทำให้ dependency ที่ต้องการเวอร์ชันเฉพาะใช้งานไม่ได้

------------------------------------------------------------------------

## 5. ทดสอบการเรียกใช้งาน BlueToolkit

ทดลองเรียกผ่าน wrapper:

``` bash
sudo /usr/local/bin/bluekit -h
```

หากยังพบข้อผิดพลาด ให้ทดสอบ entry point ภายใน virtual environment โดยตรง:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/bluekit -h
```

การทดสอบทั้งสองแบบช่วยแยกปัญหาได้ว่าเกิดจาก wrapper
หรือจากตัวโปรแกรม/Dependencies

------------------------------------------------------------------------

## 6. ปัญหา pip แจ้งว่าไม่ได้ระบุ package

หากพบข้อความ:

``` text
ERROR: You must give at least one requirement to install
```

แปลว่า `pip install` ไม่ได้รับชื่อแพ็กเกจหรือไฟล์ requirements ที่จะติดตั้ง
อาจเกิดจากการคัดลอกคำสั่งหลายบรรทัดไม่ครบ หรือใช้เครื่องหมาย `\` ผิดตำแหน่ง

ตัวอย่างคำสั่งแบบบรรทัดเดียว:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install tqdm PyYAML psutil
```

หรือระบุไฟล์ requirements ที่มีอยู่จริง:

``` bash
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip install -r /usr/share/BlueToolkit/requirements.txt
```

อย่ารันคำสั่ง `-r` หากยังไม่ได้ตรวจสอบว่าไฟล์นั้นมีอยู่จริง

------------------------------------------------------------------------

## 7. ลำดับการ Troubleshooting ที่แนะนำ

ใช้ลำดับนี้เมื่อพบปัญหา:

1.  อ่าน error ให้ชัดเจน และจดชื่อโมดูลที่ขาดหายไป
2.  ตรวจสอบว่า BlueToolkit ใช้ Python จาก `/usr/share/BlueToolkit/.venv`
3.  ตรวจสอบไฟล์ dependencies ของ repository
4.  ติดตั้ง dependencies ลงใน virtual environment ที่ถูกต้อง
5.  รัน `pip check` เพื่อตรวจสอบ dependency conflicts
6.  ทดสอบ import โมดูลที่เคยมีปัญหา
7.  ทดสอบ `sudo /usr/local/bin/bluekit -h` อีกครั้ง
8.  หากยังไม่สำเร็จ ให้เก็บ error เต็มชุดเพื่อวิเคราะห์ต่อ

### คำสั่งตรวจสอบแบบรวบรัด

``` bash
# ตรวจสอบ Python และ pip
sudo /usr/share/BlueToolkit/.venv/bin/python --version
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip --version

# ตรวจสอบ dependencies
sudo /usr/share/BlueToolkit/.venv/bin/python -m pip check

# ทดสอบ modules ที่เคยพบปัญหา
sudo /usr/share/BlueToolkit/.venv/bin/python -c "import tqdm, yaml, psutil; print('All modules OK')"

# ทดสอบ BlueToolkit
sudo /usr/local/bin/bluekit -h
```

------------------------------------------------------------------------

## หมายเหตุด้านความปลอดภัย

-   ใช้ BlueToolkit ทดสอบเฉพาะอุปกรณ์และระบบ Bluetooth/BLE ที่ตนเองเป็นเจ้าของ
    หรือได้รับอนุญาตให้ทดสอบ
-   หลีกเลี่ยงการติดตั้งแพ็กเกจด้วย `sudo pip` ลง system Python โดยตรง ใช้ Python
    ของ virtual environment ตามตัวอย่าง
-   ก่อนแก้ไข dependency versions ให้ตรวจสอบข้อกำหนดของ repository
    และบันทึกสถานะเดิมไว้
-   การติดตั้ง dependencies สำเร็จไม่ได้ยืนยันว่าเครื่องมือทำงานได้ครบทุกฟังก์ชัน
    ควรทดสอบตามขั้นตอนของ lab ในสภาพแวดล้อมที่ได้รับอนุญาต
