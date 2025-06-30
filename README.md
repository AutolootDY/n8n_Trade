-----
<p align="center">
  <img src="test/test.gif"/>
</p>

# 📈 บอทเทรดคริปโต AI อัตโนมัติด้วย n8n บน Binance TH 🤖

โปรเจกต์นี้อธิบายถึงระบบเทรดคริปโตเคอร์เรนซีอัตโนมัติที่แข็งแกร่ง ซึ่งใช้พลังของ **n8n** สำหรับการจัดการ Workflow, **FastAPI** สำหรับการดึงข้อมูลประสิทธิภาพสูง, **Flask** สำหรับการส่งคำสั่งอย่างปลอดภัยบน Binance TH และ **AI Agent** (ผ่าน n8n) สำหรับการวิเคราะห์ทางเทคนิคที่ซับซ้อน ระบบยังรวมถึงการแจ้งเตือนสัญญาณการเทรดแบบเรียลไทม์ผ่าน **Telegram** ด้วย.

## ✨ คุณสมบัติหลัก

  * **การดึงข้อมูลอัตโนมัติ:** แบ็กเอนด์ FastAPI จะดึงข้อมูล OHLC (ราคาเปิด, ราคาสูงสุด, ราคาต่ำสุด, ราคาปิด) แบบเรียลไทม์ และตัวชี้วัดทางเทคนิคที่คำนวณไว้ (EMA, RSI, MACD) สำหรับแท่งเทียนย้อนหลัง 100 แท่งล่าสุด
  * **การวิเคราะห์ทางเทคนิคที่ขับเคลื่อนด้วย AI:** AI Agent ที่รวมอยู่ใน n8n จะวิเคราะห์แนวโน้มตลาด, สัญญาณตัวชี้วัด และรูปแบบ Price Action ตาม Prompt ที่กำหนดอย่างละเอียด
  * **การสร้างสัญญาณการเทรดอัจฉริยะ:** AI จะสร้างคำแนะนำการเทรดที่ชัดเจน (LONG, SELL หรือ HOLD) พร้อมเหตุผลประกอบ
  * **การรวมเข้ากับ Binance TH:** แบ็กเอนด์ Flask จะส่งคำสั่งเทรด (ซื้อ/ขาย) อย่างปลอดภัยไปยัง API ของ Binance TH
  * **การแจ้งเตือน Telegram แบบเรียลไทม์:** รับการอัปเดตทันทีเกี่ยวกับผลการวิเคราะห์ AI และสัญญาณการเทรดโดยตรงไปยังแชท Telegram ของคุณ รวมถึงรูปภาพกราฟด้วย
  * **สถาปัตยกรรมแบบโมดูลาร์:** ส่วนประกอบต่างๆ (FastAPI, Flask, n8n) ถูกแยกออกจากกันเพื่อความยืดหยุ่นและการปรับขนาด
  * **ระบบอัตโนมัติแบบ Low-Code (n8n):** จัดการ Workflow ที่ซับซ้อนและการรวมระบบด้วยภาพ และใช้โค้ดน้อยที่สุด

## 🚀 ระบบทำงานอย่างไร (สถาปัตยกรรมระบบ)

ระบบจะทำงานตามลำดับขั้นตอนดังนี้:

1.  **การนำเข้าข้อมูล (FastAPI ไปยัง n8n):**
      * **FastAPI Backend:** ดึงข้อมูล OHLC 1 นาทีล่าสุด 100 แท่ง สำหรับคู่สัญลักษณ์ที่ระบุจาก Binance
      * **การคำนวณตัวชี้วัด:** คำนวณตัวชี้วัด EMA (9, 21), RSI (14) และ MACD โดยใช้ไลบรารี `ta`
      * **การส่งข้อมูล:** **Post** ข้อมูล OHLC และตัวชี้วัดที่ครอบคลุมทั้งหมดไปยัง **Webhook เฉพาะของ n8n**
2.  **ระบบวิเคราะห์ AI (n8n):**
      * **n8n Webhook Node:** รอรับข้อมูลที่เข้ามาจาก FastAPI
      * **Set Clean JSON Node:** ประมวลผลและจัดระเบียบข้อมูล JSON ที่ได้รับ
      * **AI Analysis System (AI Agent Node):** ส่งข้อมูลที่มีโครงสร้างไปยัง AI Language Model (เช่น GPT-4o-mini) ด้วย Prompt พิเศษเพื่อทำการวิเคราะห์ทางเทคนิคเชิงลึก Prompt จะสั่งให้ AI วิเคราะห์แนวโน้ม EMA, ระดับ RSI, สัญญาณ MACD และรูปแบบ Price Action จากนั้นแนะนำสัญญาณ (LONG/SELL/HOLD) พร้อมเหตุผลประกอบ
      * **Code Node:** ดึงสัญญาณ `LONG`, `SELL` หรือ `HOLD` จากข้อความเอาต์พุตของ AI เพื่อนำไปประมวลผลต่อไป
3.  **การดำเนินการสัญญาณการเทรด (n8n ไปยัง Flask):**
      * **HTTP Request to Binance TH (n8n Node):** ตามสัญญาณจาก Code node, n8n จะส่งคำขอ POST ไปยัง Flask backend ที่ Endpoint `/order`
      * **Flask Backend:** รับสัญญาณการเทรด (เช่น `ACTION: "OPEN LONG"`, `SYMBOL: "BTCUSDT"`) และดำเนินการคำสั่งซื้อ/ขายที่เกี่ยวข้องบน Binance TH โดยใช้ไลบรารี `binance-connector`
4.  **ระบบแจ้งเตือน (n8n ไปยัง Telegram):**
      * **Set Clean Output to JSON Node:** เตรียมเอาต์พุตการวิเคราะห์ของ AI สำหรับ Telegram
      * **Set Interval and Symbol Node:** รวบรวมข้อมูลที่จำเป็นสำหรับการสร้างกราฟ
      * **HTTP Request to chart-img.com (n8n Node):** สร้างรูปภาพกราฟการเทรดแบบเรียลไทม์ (พร้อมตัวชี้วัด RSI, MACD, EMA 9) โดยใช้ API ของ `chart-img.com`
      * **Send Message to Telegram (n8n Node):** ส่งสรุปการวิเคราะห์ AI โดยละเอียด (ข้อความ) ไปยังแชท Telegram ที่กำหนดค่าไว้
      * **Send Image to Telegram (n8n Node):** ส่งรูปภาพกราฟการเทรดที่สร้างขึ้นไปยังแชท Telegram

## ⚙️ การตั้งค่าและการติดตั้ง

### สิ่งที่ต้องมี

  * **Python 3.8+**
  * **n8n Instance:** โฮสต์เองหรือบนคลาวด์
  * **บัญชี Binance TH:** พร้อม API Key และ Secret ที่เปิดใช้งานสำหรับการเทรด
  * **Telegram Bot:** สร้างผ่าน BotFather พร้อม Bot Token และ Chat ID ของคุณ
  * **chart-img.com API Key:** (ไม่บังคับ สำหรับรูปภาพกราฟ)

### 1\. การตั้งค่าแบ็กเอนด์ (FastAPI & Flask)

**a. Clone Repository:**

```bash
git clone <your-repo-url>
cd <your-repo-directory>
```

**b. สร้าง Virtual Environment และติดตั้ง Dependencies:**

```bash
# สำหรับ FastAPI
cd fastapi_backend # สมมติว่าโค้ด FastAPI ของคุณอยู่ในโฟลเดอร์นี้
python -m venv venv_fastapi
source venv_fastapi/bin/activate # บน Windows: .\venv_fastapi\Scripts\activate
pip install fastapi uvicorn httpx pandas ta

# สำหรับ Flask
cd ../flask_backend # สมมติว่าโค้ด Flask ของคุณอยู่ในโฟลเดอร์นี้
python -m venv venv_flask
source venv_flask/bin/activate # บน Windows: .\venv_flask\Scripts\activate
pip install Flask python-binance
```

**c. กำหนดค่า Environment Variables:**

สำหรับแบ็กเอนด์ Flask ให้สร้างไฟล์ `.env` หรือตั้งค่า Environment Variables โดยตรง:

```env
# ใน flask_backend/.env
BINANCE_TH_API_KEY="YOUR_BINANCE_TH_API_KEY"
BINANCE_TH_API_SECRET="YOUR_BINANCE_TH_API_SECRET"
```

อย่าลืมแทนที่ `"YOUR_BINANCE_TH_API_KEY"` และ `"YOUR_BINANCE_TH_API_SECRET"` ด้วย API Credentials ของ Binance TH ของคุณ

**d. รันแบ็กเอนด์:**

```bash
# รัน FastAPI (เช่น บนพอร์ต 8000)
# ในไดเรกทอรี fastapi_backend
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# รัน Flask (เช่น บนพอร์ต 5000)
# ในไดเรกทอรี flask_backend
flask run --host 0.0.0.0 --port 5000
```

ตรวจสอบให้แน่ใจว่าแบ็กเอนด์เหล่านี้สามารถเข้าถึงได้โดย n8n instance ของคุณ (เช่น ผ่าน `http://127.0.0.1:8000` และ `http://127.0.0.1:5000` หากรันบนเครื่องเดียวกัน หรือ IP สาธารณะ/DNS หากอยู่บนเซิร์ฟเวอร์ที่แตกต่างกัน)

### 2\. การตั้งค่า n8n Workflow

**a. นำเข้า Workflow:**

1.  เปิด n8n instance ของคุณ
2.  ไปที่ "Workflows" และคลิก "New"
3.  คลิกจุดสามจุด `...` ถัดจาก "New workflow" และเลือก "Import from JSON"
4.  อัปโหลดไฟล์ `n8n_Template.json` ที่ให้มาใน repository นี้

**b. กำหนดค่า n8n Nodes:**

  * **Webhook Node (`f813a8e1-59d5-442b-9afa-b50b39dc1fbf`):**
      * หลังจากนำเข้า n8n จะสร้าง Webhook URL ที่ไม่ซ้ำกันสำหรับ Node นี้ (path `/ohlc`) คัดลอก URL นี้
  * **HTTP Request to chart-img.com Node (`73ebc1db-250e-4562-bac3-eaa9adc7f7c9`):**
      * ใน `Header Parameters` ให้เพิ่ม `x-api-key` ของคุณจาก `chart-img.com`
  * **HTTP Request to Binance TH Node (`4f835ecc-b504-4203-9dc7-735bce0b0177`):**
      * อัปเดตฟิลด์ `URL` ให้ตรงกับ Endpoint คำสั่งของแบ็กเอนด์ Flask ของคุณ (เช่น `http://YOUR_FLASK_IP:5000/order`)
      * **สำคัญ:** ตรวจสอบให้แน่ใจว่าแบ็กเอนด์ Flask ของคุณกำลังทำงานและสามารถเข้าถึงได้จากที่ n8n โฮสต์อยู่
  * **Send Message to Telegram (`e58a5dac-b0c1-405d-b9ce-7eb3e42a252b`) & Send image to Telegram (`eec2ea5e-279e-4c6b-8e8c-eb174d88182c`):**
      * **Credentials:** คลิก "Credentials" และกำหนดค่า "Telegram API" credential ใหม่ คุณจะต้องใช้ Telegram Bot Token จาก BotFather
      * **Chat ID:** ในทั้งสอง Telegram Node ให้กรอก `Chat ID` ของผู้ใช้ Telegram หรือกลุ่มที่คุณต้องการส่งข้อความไป
      * ฟิลด์ `Text` สำหรับ `Send Message to Telegram` ดึงข้อมูลโดยตรงจากเอาต์พุตการวิเคราะห์ AI: `={{ $json.data }}`
      * ฟิลด์ `File` สำหรับ `Send image to Telegram` ดึง URL รูปภาพจากการตอบกลับของ `chart-img.com`: `={{ $json.url }}`
  * **AI Analysis System Node (`e4c58931-c9c4-429c-9426-8e56e6186ddf`):**
      * ตรวจสอบให้แน่ใจว่า "Language Model" ได้รับการกำหนดค่าอย่างถูกต้องและสามารถเข้าถึง API key ของ OpenAI (หรือที่คล้ายกัน) เทมเพลตใช้ `GPT-4o-mini`

**c. อัปเดต FastAPI ด้วย n8n Webhook URL:**

  * ในโค้ดของแบ็กเอนด์ FastAPI ของคุณ (`main.py` หรือที่คล้ายกัน) ให้อัปเดตตัวแปร `N8N_WEBHOOK_URL` ให้เป็น URL ที่แน่นอนของ n8n Webhook Node ของคุณ (อันที่คุณคัดลอกในขั้นตอน 2.b)

### 3\. เริ่มต้น Workflow

  * เปิดใช้งาน n8n workflow โดยสลับสวิตช์ "Active" ที่มุมขวาบนของ n8n editor

## 🏃 การรันระบบ

เมื่อส่วนประกอบทั้งหมดได้รับการกำหนดค่าและทำงานอยู่:

1.  **Trigger Data Push:** เข้าถึง Endpoint ของ FastAPI ในเบราว์เซอร์ของคุณหรือผ่านเครื่องมือเช่น Postman เพื่อเริ่มการรวบรวมข้อมูลและส่งไปยัง n8n
    ตัวอย่าง: `http://YOUR_FASTAPI_IP:8000/get_and_send_ohlc/BTCUSDT`
2.  **ตรวจสอบ n8n:** สังเกตการทำงานของ n8n workflow คุณควรเห็นข้อมูลไหลผ่าน Node ต่างๆ
3.  **ตรวจสอบ Telegram:** หาก AI สร้างสัญญาณ คุณควรได้รับการแจ้งเตือนข้อความและรูปภาพกราฟในแชท Telegram ที่คุณกำหนดค่าไว้
4.  **ยืนยันคำสั่ง:** หากมีการส่งสัญญาณการเทรด (LONG/SELL) ให้ตรวจสอบบัญชี Binance TH ของคุณสำหรับคำสั่งที่ดำเนินการแล้ว

## ⚠️ ข้อควรพิจารณาและข้อจำกัดที่สำคัญ

  * **ความปลอดภัยของ API:** **ห้ามฮาร์ดโค้ด API key หรือ secret ลงในโค้ดของคุณโดยตรงเด็ดขาด** ควรใช้ environment variables หรือระบบจัดการ secret ที่ปลอดภัยเสมอ
  * **ความเสี่ยงในการเทรด:** การเทรดอัตโนมัติมีความเสี่ยงทางการเงินอย่างมาก ควรทดสอบกลยุทธ์ของคุณอย่างละเอียดในสภาพแวดล้อมจำลอง (paper trading) ก่อนนำไปใช้กับเงินจริง
  * **ข้อจำกัดของ AI:** โมเดล AI อาจเกิดข้อผิดพลาดได้ การวิเคราะห์ของ AI ควรถูกมองว่าเป็นเครื่องมือช่วย ไม่ใช่สิ่งทดแทนการตัดสินใจของมนุษย์
  * **Rate Limits:** ระมัดระวัง Rate Limits ของ API จาก Binance และ `chart-img.com` เพื่อหลีกเลี่ยงการถูกบล็อก
  * **การจัดการข้อผิดพลาด:** ใช้การจัดการข้อผิดพลาดที่แข็งแกร่งทั้งใน FastAPI และ Flask เพื่อจัดการกับปัญหาที่ไม่คาดคิด (เช่น API ล้มเหลว, ปัญหาเครือข่าย)
  * **Backtesting:** ก่อนการเทรดจริง ให้ทำการ Backtest กลยุทธ์การเทรดของคุณอย่างละเอียดโดยใช้ข้อมูลย้อนหลังเพื่อประเมินประสิทธิภาพ
  * **การปรับแต่ง:** โค้ดและ n8n workflow ที่ให้มาเป็นเพียงจุดเริ่มต้น ปรับแต่ง Prompt ของ AI, Logic การเทรด, ตัวชี้วัด และประเภทคำสั่ง (`MARKET` vs. `LIMIT`) เพื่อให้เหมาะกับกลยุทธ์การเทรดเฉพาะของคุณ

-----

คุณสามารถร่วมพัฒนาโปรเจกต์นี้ หรือสร้าง Issue หากคุณพบปัญหาใดๆ ได้เลย\!
