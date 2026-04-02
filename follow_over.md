# Flow Tong Quat - AI Tu Van Mon An (posSpeak)

> Tai lieu nay mo ta flow tong quat cua chuc nang AI, khong gan voi bat ky kich ban cu the nao.
> Moi noi dung AI phan hoi (loi chao, goi y, giai thich) deu do Dify sinh ra dua tren Knowledge + Context.

---

## 1. TONG QUAN FLOW

```
┌──────────────────────────────────────────────────────────────────┐
│                       VONG DOI 1 SESSION                         │
│                                                                  │
│  ┌─────────┐   ┌───────────┐   ┌──────────┐   ┌─────────────┐  │
│  │  KHOI   │   │    AI     │   │  KHACH   │   │   KHACH     │  │
│  │  TAO    │──→│  CHAO +  │──→│  TUONG   │──→│   DAT MON   │  │
│  │ SESSION │   │  GOI Y   │   │  TAC     │   │   + THANH   │  │
│  │         │   │  CHU DONG │   │  VOI AI  │   │   TOAN      │  │
│  └─────────┘   └───────────┘   └──────────┘   └─────────────┘  │
│                                      ↑                           │
│                                      │                           │
│                               (lap lai nhieu lan)                │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. CHI TIET TUNG BUOC

### Buoc 1: Khoi Tao Session

```
Khach ngoi ban (quet QR / NFC / nhan vien gan)
       │
       v
Flutter tao session voi Ruby BE
       │
       ├── POST /api/sessions { table_id }
       │   → Nhan: session_id, dify_conversation_id (null)
       │
       ├── GET /api/menus?category=recommend
       │   → Nhan: danh sach mon mac dinh (menu cua nha hang)
       │
       └── GET /api/context
           → Nhan: { weather, time_of_day, season, popular_items_today }
```

**Ket qua:** Man hinh Menu hien thi voi menu mac dinh cua nha hang. AI chua can thiep.

---

### Buoc 2: AI Chao + Goi Y Chu Dong

```
Flutter gui context den Ruby BE
       │
       v
Ruby BE gui den Dify:
  POST /chat-messages (streaming)
  {
    inputs: { weather, time_of_day, season, language },
    query: "Khach moi ngoi ban. Hay chao va goi y mon phu hop.",
    response_mode: "streaming",
    user: session_id
  }
       │
       v
Dify xu ly:
  1. Doc Knowledge (menu, nguyen lieu, thoi tiet)
  2. Phan tich context (troi nong? troi lanh? bua trua? bua toi?)
  3. Sinh ra:
     - Loi chao phu hop
     - Nhan xet ve ngu canh (thoi tiet, thoi gian...)
     - Danh sach mon goi y kem ly do
       │
       v
Ruby BE:
  - Stream phan text ve Flutter (hien tung ky tu tren AI Banner)
  - Parse phan suggest (menu_ids + reasons)
  - Map menu_ids → menu items day du tu DB
  - Stream suggested_items ve Flutter
       │
       v
Flutter:
  1. AI Banner: hien loi chao streaming
  2. AI Banner: hien nhan xet ngu canh streaming
  3. Menu Grid: blur menu hien tai
  4. Hien Voice Guide (mic) de khach co the phan hoi
  5. Swap menu → suggested items
  6. Reveal tung card voi animation + AI reason badge
  7. Luu dify_conversation_id vao session
```

**Diem quan trong:**
- Toan bo noi dung (loi chao, nhan xet thoi tiet, mon goi y) do DIFY quyet dinh
- Ruby BE chi lam proxy + map menu_ids thanh data day du
- Flutter chi lo hien thi + animation
- Khong co bat ky text nao fix cung trong code

---

### Buoc 3: Khach Tuong Tac Voi AI

Khach co the tuong tac theo nhieu cach, tat ca deu di qua cung 1 flow:

```
┌──────────────────────────────────────────────────────────┐
│                   CAC CACH TUONG TAC                      │
│                                                          │
│  [A] Noi qua Mic ─── STT ───┐                           │
│  [B] Nhap text ──────────────┼──→ text message           │
│  [C] Tap quick chip ─────────┘       │                   │
│  [D] Chon mon (tap card) ────────────┼──→ menu_item_id   │
│                                      │                   │
│                                      v                   │
│                            POST /api/ai/chat             │
│                            {                             │
│                              session_id,                 │
│                              message,        ← text      │
│                              menu_item_id,   ← neu chon  │
│                              conversation_id ← tu session │
│                            }                             │
└──────────────────────────────┬───────────────────────────┘
                               │
                               v
                    Ruby BE → Dify (streaming)
                               │
                               v
              ┌────────────────┼────────────────┐
              │                │                │
              v                v                v
       [Text phan hoi]  [Suggest moi]   [Giai thich mon]
       Stream ve        Menu items moi  Chi tiet + ly do
       AI Banner        swap vao grid   hien tren detail
```

**Cac kich ban Dify co the tra ve:**

| Khach noi/lam | Dify tra ve | Flutter xu ly |
|---|---|---|
| "Toi muon an gi do mat" | Text + suggest 6 mon lanh | Stream text + swap menu |
| "Co mon nao khong co gluten?" | Text + suggest mon khong gluten | Stream text + swap menu |
| "Mon nay co gi?" (dang o detail) | Text giai thich chi tiet | Stream text tren banner |
| "Cho toi cay hon" | Text + suggest mon cay | Stream text + swap menu |
| "Mon nay bao nhieu calo?" | Text tra loi calo | Stream text tren banner |
| "Goi y ruou hop voi mon nay" | Text + suggest do uong | Stream text + swap menu |
| Chon 1 mon (tap card) | Text giai thich ly do chon | Stream text + hien detail |
| "OK cho toi mon nay" | Text xac nhan | Them vao cart |

---

### Buoc 4: Xem Chi Tiet Mon (AI Explain)

```
Khach tap 1 card mon
       │
       v
Flutter:
  1. GET /api/menus/:id → lay chi tiet mon
  2. POST /api/ai/explain { session_id, menu_item_id, conversation_id }
       │
       v
Ruby BE → Dify:
  query: "Khach dang xem mon [ten mon]. Hay giai thich tai sao phu hop."
  (Dify co conversation context → biet so thich khach da noi truoc do)
       │
       v
Dify tra ve:
  - Ly do tai sao mon nay phu hop voi khach
  - Goi y cac mon bo sung (candidates)
  - Canh bao di ung neu co
       │
       v
Flutter hien thi:
  - Anh lon + thong tin mon
  - AI Banner: stream text giai thich
  - AI Reason badge tren anh
  - Candidate navigator (prev/next) neu co nhieu candidates
  - Allergen warning tags
  - Voice Guide de khach hoi them
```

---

### Buoc 5: Hoi Tiep / Tinh Chinh (Vong Lap)

```
Khach dang o Detail hoac Menu
       │
       ├── Noi: "Cay hon" ──────────────────────────┐
       ├── Noi: "Re hon" ──────────────────────────┐ │
       ├── Tap chip: "Mon thit" ───────────────────┤ │
       ├── Nhap: "Tim mon cho nguoi an chay" ──────┤ │
       │                                           │ │
       v                                           v v
  POST /api/ai/chat ─── Ruby BE ─── Dify (co conversation context)
       │
       v
  Dify hieu context truoc do + yeu cau moi
  → Tra ve goi y moi phu hop hon
       │
       v
  Flutter:
  - Neu o Menu: swap suggested items moi
  - Neu o Detail: chuyen sang mon moi hoac cap nhat candidates
  - AI Banner luon stream text phan hoi moi
```

**Dify giu conversation context:**
- Lan 1: "Toi muon mon mat" → Dify goi y 6 mon lanh
- Lan 2: "Cay hon" → Dify hieu "mon mat + cay" → goi y mon lanh vi cay
- Lan 3: "Khong co hai san" → Dify hieu "mon mat + cay + khong hai san" → loc tiep

---

### Buoc 6: Them Vao Gio + Dat Mon

```
Khach tap "Them vao gio"
       │
       v
Flutter: POST /api/sessions/:id/cart/items { menu_id, quantity, note }
       │
       v
(Khach co the quay lai Menu de chon them)
(AI co the goi y: "Mon nay hop voi [mon da chon] day!")
       │
       v
Khach tap "Dat mon"
       │
       v
Flutter: POST /api/sessions/:id/orders
       │
       v
Ruby BE:
  - Tao order tu cart items
  - Gui thong bao den bep
  - (Optional) Goi Dify: "Khach da dat [danh sach mon]. Hay noi gi do."
       │
       v
AI Banner: "ありがとうございます！お料理は約15分でお届けします。"
(Noi dung do Dify sinh, khong fix cung)
```

---

## 3. VAI TRO CUA TUNG THANH PHAN

### Flutter App - Chi Lo Hien Thi + Thu Thap Input

```
KHONG lam:
  ✗ Khong chua bat ky text AI nao (loi chao, goi y, giai thich)
  ✗ Khong quyet dinh goi y mon nao
  ✗ Khong xu ly logic AI

CO lam:
  ✓ Hien thi menu tu Ruby BE
  ✓ Stream text tu SSE len AI Banner (tung ky tu)
  ✓ Thu am → chuyen text (STT)
  ✓ Gui text/menu_item_id den Ruby BE
  ✓ Nhan suggested_items → hien thi voi animation
  ✓ Quan ly cart, order (CRUD qua Ruby BE)
  ✓ Animation: blur, reveal, streaming cursor, mic pulse
  ✓ Da ngon ngu UI (ja/en/vi) — nhan language tu session
```

### Ruby BE - Proxy + Data + Business Logic

```
KHONG lam:
  ✗ Khong sinh noi dung AI
  ✗ Khong quyet dinh goi y mon nao

CO lam:
  ✓ CRUD: menus, categories, sessions, cart, orders
  ✓ Proxy Dify API: nhan request tu Flutter → gui Dify → stream ve
  ✓ Parse Dify response: tach text va <!--SUGGEST:--> tag
  ✓ Map menu_ids tu Dify → menu items day du (ten, gia, anh, allergens...)
  ✓ Cung cap context: goi Weather API, tinh time_of_day, season
  ✓ Dong bo data len Dify Knowledge (menu, nguyen lieu, thoi tiet)
  ✓ Luu AI log (lich su hoi thoai)
  ✓ Quan ly dify_conversation_id theo session
```

### Dify - Bo Nao AI

```
KHONG lam:
  ✗ Khong biet ve UI, animation, layout
  ✗ Khong truy cap truc tiep DB

CO lam:
  ✓ Nhan context (thoi tiet, thoi gian, mua, ngon ngu)
  ✓ Doc Knowledge (menu, nguyen lieu) de biet nha hang co gi
  ✓ Giu conversation context (nho so thich khach da noi)
  ✓ Sinh loi chao phu hop
  ✓ Phan tich ngu canh → chon mon goi y + ly do
  ✓ Tra loi cau hoi ve mon an (calo, nguyen lieu, di ung...)
  ✓ Tra ve structured data: text + <!--SUGGEST:{menu_ids, reasons}-->
  ✓ Ho tro da ngon ngu (nhan language tu inputs)
```

---

## 4. DIFY KNOWLEDGE ORGANIZATION

### Knowledge 1: Menu Items (dong bo tu Ruby)

```
Moi document = 1 mon an, format:

---
ID: 12
Ten: 冷やしラーメン
Category: 麺類
Gia: ¥950
Mo ta: 濃厚なゴマだれに冷たい麺。夏の定番の一品
Calories: 420 kcal
Thoi gian nau: 10 phut
Nguyen lieu: mi, vung, dau tuong, hanh, trung
Di ung: trung, dau tuong, lua mi
Tuy chinh: Co (do cay, topping)
Dac diem: mon lanh, phu hop mua he, nhe nhang
---
```

### Knowledge 2: Nguyen Lieu + Di Ung

```
Danh sach nguyen lieu va phan loai:
- Lua mi (grain) — ALLERGEN
- Trung (protein) — ALLERGEN
- Sua (dairy) — ALLERGEN
- Tom (seafood) — ALLERGEN
- Dau phong (nut) — ALLERGEN
- Ca hoi (fish)
- Thit bo (meat)
- Rau cai (vegetable)
...
```

### Knowledge 3: Context Dong (cap nhat dinh ky)

```
Cap nhat moi 30 phut boi Ruby cron job:

Thoi tiet hien tai: 35°C, nang, do am 65%
Buoi: bua trua (11:00-14:00)
Mua: he
Mon ban chay hom nay: [冷やしラーメン (32 phan), レモンソーダ (28 phan)]
Mon moi: [新メニュー: 抹茶パフェ]
Mon het: [海鮮天ぷら — het nguyen lieu]
```

---

## 5. DIFY SYSTEM PROMPT (TONG QUAT)

```
Ban la AI Staff cua nha hang posSpeak.
Ten: duoc cau hinh boi nha hang (vi du: Sakura, Hana, Yuki...)
Nhiem vu: tu van mon an cho khach.

NGUYEN TAC CHUNG:
1. Tra loi bang ngon ngu tuong ung voi {{language}}
2. Dua tren context (thoi tiet, thoi gian, mua) de goi y mon phu hop
3. Ghi nho so thich khach da noi trong conversation
4. Neu khach hoi ve di ung → CANH BAO RO RANG, uu tien an toan
5. Giong noi: than thien, lich su, chuyen nghiep
6. Khong ep khach, chi goi y va giai thich

KHI GOI Y MON:
- Toi da 6 mon moi lan
- Moi mon phai co ly do cu the tai sao phu hop
- Tra ve cuoi message:
  <!--SUGGEST:{"menu_ids":[...],"reasons":{"id":"emoji + ly do ngan"}}-->

KHI GIAI THICH MON:
- Noi ve huong vi, nguyen lieu chinh
- Lien he voi context (tai sao mon nay phu hop luc nay)
- Neu biet so thich khach → lien he voi so thich do
- Goi y mon bo sung neu phu hop

KHI KHACH TINH CHINH:
- Hieu yeu cau moi CONG THEM voi context cu
- Vi du: "cay hon" = giu cac tieu chi cu + them dieu kien cay
- Goi y lai voi tieu chi da cap nhat

CONTEXT (tu dong dien):
- Thoi tiet: {{weather}}
- Buoi: {{time_of_day}}
- Mua: {{season}}
- Ngon ngu: {{language}}
```

---

## 6. SSE STREAM PROTOCOL (Ruby ↔ Flutter)

### Events

```
# --- AI dang noi (tung ky tu hoac tung cum) ---
data: {"event":"message","answer":"あ"}
data: {"event":"message","answer":"り"}
data: {"event":"message","answer":"がとう"}

# --- AI goi y mon (sau khi parse <!--SUGGEST:-->) ---
data: {"event":"suggest","items":[
  {"id":1,"name":"冷やしラーメン","price":950,"image_url":"...","reason":"🌡️ ...","calories":420,"cooking_time":10},
  {"id":3,"name":"レモンソーダ","price":480,"image_url":"...","reason":"✨ ...","calories":80,"cooking_time":3}
]}

# --- Conversation ID (lan dau hoac khi thay doi) ---
data: {"event":"conversation","conversation_id":"abc-123"}

# --- Ket thuc ---
data: {"event":"done"}

# --- Loi ---
data: {"event":"error","message":"Dify timeout"}
```

### Flutter xu ly

```
Nhan "message"      → append text vao AI Banner (streaming effect)
Nhan "suggest"      → blur menu hien tai → swap items → reveal animation
Nhan "conversation" → luu conversation_id vao session provider
Nhan "done"         → tat streaming cursor, hien Voice Guide
Nhan "error"        → hien thong bao loi, fallback menu mac dinh
```

---

## 7. XU LY TRUONG HOP DAC BIET

### Dify Timeout / Loi

```
Flutter gui request → Ruby BE → Dify khong tra loi (timeout 15s)
       │
       v
Ruby BE tra ve: {"event":"error","message":"..."}
       │
       v
Flutter:
  - AI Banner: "申し訳ございません。少々お待ちください。" (text mac dinh)
  - Hien menu mac dinh (khong co AI suggest)
  - Voice Guide van hien de khach thu lai
  - Retry tu dong sau 10s (toi da 2 lan)
```

### Khach Khong Tuong Tac

```
AI da chao + goi y xong, khach khong noi gi (30s)
       │
       v
Flutter tu dong:
  - AI Banner: stream 1 message nhac nhe (lay tu Dify)
  - Vi du: "何かお手伝いできることはありますか？" (Dify sinh)
  - Neu 60s van khong tuong tac → AI im, chi hien menu binh thuong
```

### Khach Doi Ngon Ngu

```
Khach tap "言語" → chon English
       │
       v
Flutter:
  - Cap nhat session.language = "en"
  - PATCH /api/sessions/:id { language: "en" }
  - Goi POST /api/ai/chat { message: "Switch to English" }
       │
       v
Dify nhan language = "en" trong inputs
  → Tu dong chuyen sang tra loi tieng Anh
  → Goi y mon van giong nhau, chi doi ngon ngu
```

### Mon Het Nguyen Lieu

```
Dify doc Knowledge (context dong) → biet mon nao da het
  → Khong goi y mon da het
  → Neu khach hoi mon da het: "Xin loi, mon X hom nay da het. Thay vao do..."
```

---

## 8. MAPPING: DEMO HTML → PRODUCTION FLOW

| Demo (fix cung)                                  | Production (dong)                                    |
|--------------------------------------------------|------------------------------------------------------|
| msg1 = "ようこそいらっしゃいませ..."              | Dify sinh loi chao dua tren context                  |
| msg2 = "本日は暑い日が続いて..."                  | Dify phan tich thoi tiet → sinh nhan xet phu hop     |
| coolMenuItems = [...6 mon fix cung]              | Dify chon menu_ids tu Knowledge → Ruby map data      |
| reason: "🌡️ 暑い日に体を冷やす"                  | Dify sinh reason cho tung mon                        |
| msg3 = "ありがとうございます！では..."            | Dify sinh phan hoi sau khi khach tuong tac           |
| candidates = [...3 mon fix cung]                 | Dify chon top 3 candidates tu conversation context   |
| ticker messages = [...3 cau fix cung]            | Dify sinh giai thich dua tren mon + context          |
| quick chips = ["肉料理がいい",...]               | Ruby BE sinh tu categories hoac Dify goi y           |
| context tags = ["さっぱりしたもの",...]           | Dify trich xuat tu conversation history              |
