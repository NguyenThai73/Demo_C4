# Flow AI Tu Van Mon An - posSpeak

## Tong Quan

He thong posSpeak su dung AI Staff ten **Sakura** (さくら) de tu van mon an tren iPad.  
AI chu dong nhan biet ngu canh (thoi tiet, so thich) va de xuat mon phu hop qua giong noi streaming.

---

## So Do Flow Tong The

```
Khach vao cua hang
  |
  v
[menu_ai.html] AI chao khach (Phase 1)
  |
  v
AI nhan biet thoi tiet nong → Goi y mon mat (Phase 2)
  |
  v
Menu hien tai bi blur → Nut Microphone xuat hien
  |
  v
Khach tap Microphone → AI nghe (2 giay)
  |
  v
AI swap 6 mon mat + reveal tung card (Phase 3)
  |
  v
Microphone hien lai → Khach co the noi tiep
  |
  v
Khach chon 1 mon (click card)
  |
  v
[menu_ai_detail.html] Trang chi tiet mon
  |
  v
AI giai thich ly do chon mon (streaming text)
  |
  v
Khach duyet 3 candidates (prev/next)
  |
  v
Voice input → AI Chat overlay → Them vao gio hang
```

---

## Man Hinh 1: menu_ai.html (Menu Grid + AI Suggest)

### Cau Truc Giao Dien

| Thanh phan         | Mo ta                                                        |
|--------------------|--------------------------------------------------------------|
| AI Banner (top)    | Speaker icon + Avatar Sakura + Streaming text                |
| Category Tabs      | おすすめ, 前菜, 麺類, ドリンク, サラダ                         |
| Section Label      | Badge "AIがご提案" + mo ta ngu canh                           |
| Menu Grid          | 3 cot, 6 card mon an (anh + ten + gia + nut them)            |
| Bottom Nav         | メニュー, カート, 注文履歴, 会話, 言語                         |
| Voice Guide (float)| Nut Microphone lon + text huong dan                          |
| Replay Button      | Nut "デモ再生" de chay lai demo                               |

### Demo Sequence (3 Phase)

#### Phase 1 - Chao Khach
- AI Banner stream text: "ようこそいらっしゃいませ！当店へのご来店ありがとうございます。ごゆっくりお楽しみください。"
- Hien thi 6 mon regular (menu mac dinh)
- Toc do stream: 45ms/ky tu

#### Phase 2 - Goi Y Theo Ngu Canh
- AI nhan biet thoi tiet nong
- Stream text: "本日は暑い日が続いておりますね 🌞 体を冷やす爽やかなお料理はいかがでしょうか？当店おすすめの涼しいメニューをご紹介いたします！"
- Sau 3 giay → chuyen sang transition

#### Transition - Blur + Voice Guide
- Tat ca 6 card menu bi blur (opacity 0.18, blur 3px)
- Sau 700ms → swap data sang coolMenuItems
- Hien thi nut Microphone voi text "タップしてご希望を話してください。"
- Cho khach tap Microphone

#### Phase 3 - AI De Xuat Mon Mat
- Khach tap Mic → Mic chuyen xanh (#22c55e) + "聞き取り中..." (2 giay)
- Mic an di → AI stream: "ありがとうございます！では、本日の暑さにぴったりな涼やかメニュー 6 品をご紹介します 🍃"
- Section label hien (sau 700ms): "✦ AIがご提案 — 本日の暑さにぴったりな涼しいメニュー 6 選"
- 6 card reveal tuan tu (moi card cach 420ms) voi animation pop
- Sau khi xong → Mic hien lai de khach tiep tuc tuong tac

### 6 Mon Regular (Menu Mac Dinh)

| #  | Ten Mon                  | Gia    | Mo Ta                                          |
|----|--------------------------|--------|-------------------------------------------------|
| 1  | 牛カルビ焼き              | ¥1,680 | Thit bo kalbi nuong than                        |
| 2  | 海鮮天ぷら盛り合わせ       | ¥1,380 | Tempura hai san tong hop                        |
| 3  | 特製ラーメン              | ¥980   | Ramen dac biet, nuoc dung tonkotsu 12 gio       |
| 4  | 鶏の唐揚げ定食            | ¥1,080 | Set ga ran kieu Nhat                            |
| 5  | 季節の刺身盛り            | ¥1,580 | Sashimi theo mua                                |
| 6  | マルゲリータピザ           | ¥1,280 | Pizza Margherita lo da                          |

### 6 Mon AI De Xuat (Cool Menu - Khi Troi Nong)

| #  | Ten Mon             | Gia    | AI Reason                    |
|----|---------------------|--------|------------------------------|
| 1  | 冷やしラーメン       | ¥950   | 🌡️ 暑い日に体を冷やす (Lam mat co the ngay nong)  |
| 2  | レモンソーダ         | ¥480   | ✨ 今日のNo.1人気ドリンク (Do uong hot nhat hom nay) |
| 3  | 冷製トマトパスタ     | ¥1,080 | 🍅 さっぱり軽めのランチに (Bua trua nhe nhang)      |
| 4  | 蒸し鶏サラダ         | ¥780   | 💪 低カロリーで栄養満点 (It calo, nhieu dinh duong)  |
| 5  | グリーンスムージー    | ¥620   | 🥝 ビタミン補給に最適 (Bo sung vitamin)             |
| 6  | 冷やし中華           | ¥950   | 🔥 辛さで汗を流してスッキリ (Cay de giai nhiet)      |

---

## Man Hinh 2: menu_ai_detail.html (Chi Tiet Mon + AI Chat)

### Cau Truc Giao Dien

| Thanh phan              | Mo ta                                                      |
|-------------------------|------------------------------------------------------------|
| AI Banner (top)         | Speaker + Avatar Sakura + Streaming text + Nut "戻る"       |
| Image Section (trai)    | Anh lon 600x600 + badge "AI推奨写真" + AI Reason overlay    |
| Thumbnail Gallery       | 4 anh nho de chuyen doi                                    |
| Info Panel (phai)       | Ten mon, mo ta, gia, calories, thoi gian nau               |
| Critical Tags           | Canh bao di ung + Tuy chinh duoc                           |
| Candidate Nav (duoi)    | Prev/Next de duyet 3 candidates                            |
| Bottom Nav              | Giong menu_ai.html                                         |
| Voice Guide (float)     | Nut Microphone, an khi AI dang noi                         |
| AI Chat Overlay         | Bottom sheet voi context, quick chips, text input           |

### Thong Tin Hien Thi Cho Moi Mon

- **Ten mon** (font 32px, bold 900)
- **Mo ta** chi tiet
- **Gia** (font 36px, bold 900)
- **Calories** (vi du: 320 kcal)
- **Thoi gian nau** (vi du: ~15 phut)
- **Canh bao di ung**: 乳製品・小麦を含む (Chua sua va lua mi)
- **Tuy chinh**: カスタマイズ可能 (Co the tuy chinh)
- **AI Reason Badge**: Overlay tren anh, giai thich ly do AI chon

### 3 Candidates (Mon De Xuat Chi Tiet)

| #  | Ten Mon                           | Gia  | AI Reason                     |
|----|-----------------------------------|------|-------------------------------|
| 1  | 白身魚のグリル 〜香草ソース〜       | ¥980 | 🌡️ 暑い日に体を冷やす          |
| 2  | 完熟トマトのカプレーゼ              | ¥720 | 🍅 さっぱり軽めのランチに       |
| 3  | 旬野菜のバーニャカウダ              | ¥880 | 🌿 野菜たっぷりでヘルシー       |

### AI Ticker (Streaming Text)

- Text hien tung ky tu (30ms/ky tu) voi cursor nhap nhay
- Lap lai moi 15 giay voi 3 messages ngau nhien:
  1. "ご希望にぴったりの一品です。白身魚の軽やかな味わいが白ワインと相性抜群。"
  2. "この料理は暑い日に体を冷やす効果があります。"
  3. "白ワインとの相性が抜群です。"
- Khi AI dang noi → an Microphone
- Khi AI noi xong → hien Microphone (sau 500ms)

### Voice Input Flow (Trang Detail)

```
Khach tap Mic
  → Mic chuyen xanh + "聞き取り中..."
  → An voice guide (hidden)
  → Sau 2 giay: reset Mic
  → Hien lai voice guide (sau 500ms)
  → Mo AI Chat Overlay
  → Tu dong dien text: "もっと辛いものが食べたい"
```

### AI Chat Overlay

- **Context tags**: さっぱりしたもの, 白ワインに合う, 軽めのメイン
- **Quick chips** (tra loi nhanh):
  - 肉料理がいい (Muon mon thit)
  - もう少し軽いもの (Nhe hon chut)
  - 辛いものが食べたい (Muon an cay)
- **Text input**: Placeholder "例：お肉料理でもっと探して"
- **Mic button**: Trong khung nhap lieu

---

## Dac Diem Ky Thuat

### Streaming Text Engine
- Toc do: 45ms/ky tu (menu_ai.html), 30ms/ky tu (detail)
- Hieu ung fade out (420ms) truoc khi stream text moi
- Cursor nhap nhay (▋) khi dang stream
- Callback chain: msg1 → pause 3s → msg2 → pause 3s → transition → msg3

### Animation
- **Card blur**: opacity 0.18, filter blur(3px), transition 600ms
- **Card reveal**: translateY(10px) scale(0.97) → translateY(0) scale(1), cubic-bezier
- **Card hover**: translateY(-4px), box-shadow tang
- **Mic pulse**: scale 1 → 1.4, opacity 0.7 → 0, lap 2s
- **Avatar dot blink**: opacity 1 → 0.35, lap 1.4s
- **Speaker pulse**: scale 1 → 1.6, opacity 0.7 → 0, lap 2.2s

### Navigation Giua Cac Trang
- menu_ai.html → menu_ai_detail.html: Click bat ky card nao
- menu_ai_detail.html → menu_ai.html: Nut "戻る" hoac nav "メニュー"
- menu_ai_detail.html → order_status.html: Nav "注文履歴"
- menu_ai_detail.html → chat_ai.html: Nav "会話"

---

## Diem Noi Bat Cua AI

1. **Proactive**: AI tu phat hien ngu canh (thoi tiet) va de xuat, khong doi user hoi
2. **Voice-first**: Microphone la tuong tac chinh, luon hien khi AI khong noi
3. **Streaming UX**: Text hien tung ky tu tao cam giac "dang noi that"
4. **AI Reason**: Moi mon co ly do cu the tai sao AI chon, hien overlay tren anh
5. **Smooth Transition**: Blur → swap → reveal tao cam giac "AI dang suy nghi"
6. **Context-aware Chat**: Overlay giu context (so thich da biet) de hoi tiep
7. **Multi-candidate**: Hien 3 candidates de khach so sanh, khong ep chon 1 mon
