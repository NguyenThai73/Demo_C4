# Follow AI – Tổng hợp các Case Tư vấn của AI Concierge

Tài liệu này tổng hợp toàn bộ các kịch bản tư vấn mà AI (SAKURA) thực hiện trong hệ thống posSpeak Demo C4.

---

## AI Concierge Profile

| Thuộc tính | Giá trị |
|---|---|
| Tên | SAKURA (さくら) |
| Vai trò | AI Concierge |
| Ngôn ngữ | Tiếng Nhật (mặc định), Tiếng Anh (menu_ai_en.html) |
| Giao diện | chat_ai.html, chat_ai_project1.html, chat_ai_project2.html, menu_ai_chat.html |

---

## Case 1 – Chào mừng & Gợi ý ban đầu

**File:** `chat_ai.html`, `chat_ai_project1.html`, `chat_ai_project2.html`

**Trigger:** Khi khách hàng mở màn hình chat lần đầu

**AI nói:**
> こんにちは！さくらです。🌸
> 本日の気分やご予算に合わせて、最適なメニューをご提案します。
> 今日は何か特別な日ですか？🎉

**Sản phẩm gợi ý ngay:**

| Món | Giá | Badge | Lý do AI |
|---|---|---|---|
| 豊後牛ハンバーグ | ¥1,850 | イチオシ | 🌡️ 暑い日に体を冷やす |
| 旬の海鮮にぎり 8貫 | ¥2,400 | 本日入荷 | ✨ 本日入荷の鮮魚 |

**Chip gợi ý nhanh:**
- `今日のおすすめは？` (primary)
- `人気メニューを見せて`
- `予算で選びたい`

---

## Case 2 – Tư vấn theo "Phổ biến nhất"

**File:** `chat_ai.html`

**Trigger:** Người dùng nhập hoặc chọn chip chứa từ `人気`

**AI trả lời:**
> 当店で特に人気のメニューをご紹介します！皆様から大変好評をいただいております！✨

**Sản phẩm gợi ý:**

| Món | Giá | Badge | Lý do AI |
|---|---|---|---|
| 刺身盛り合わせ | ¥2,800 | 人気No.1 | 📈 今月最も注文されています |
| 特選和牛ステーキ | ¥3,800 | リピート多 | ⭐ 満足度98%の人気メニュー |

---

## Case 3 – Tư vấn theo Ngân sách

**File:** `chat_ai.html`

**Trigger:** Người dùng nhập hoặc chọn chip chứa từ `予算`

**AI trả lời:**
> ご予算に合わせて、満足度の高いお得なセットをご提案いたします！💰

**Sản phẩm gợi ý:**

| Món | Giá | Badge | Lý do AI |
|---|---|---|---|
| 日替わり御膳 | ¥1,200 | コスパ良 | 💰 お手頃価格で大満足 |
| 彩り野菜の天丼 | ¥980 | 得。 | 🥗 ヘルシー＆リーズナブル |

---

## Case 4 – Tư vấn "Hôm nay nên ăn gì" (Recommend mặc định)

**File:** `chat_ai.html`

**Trigger:** Người dùng hỏi về おすすめ hoặc các câu hỏi khác không thuộc case 2 & 3

**AI trả lời:**
> 本日の仕入れに合わせた、シェフ自慢の「本日のおすすめ」をご案内いたします！🍃

**Sản phẩm gợi ý:**

| Món | Giá | Badge | Lý do AI |
|---|---|---|---|
| 豊後牛ハンバーグ | ¥1,850 | イチオシ | 🌡️ 暑い日に体を冷やす |
| 旬の海鮮にぎり | ¥2,400 | 本日入荷 | ✨ 本日入荷の鮮魚 |

---

## Case 5 – Tư vấn theo Thời tiết / Ngữ cảnh (Weather-based)

**File:** `menu_ai_chat.html`

**Trigger:** AI tự động phân tích bối cảnh (ngày hôm nay nóng) và chủ động đề xuất

**AI nói (streaming text):**
1. > ようこそいらっしゃいませ！ 当店へのご来店ありがとうございます。ごゆっくりお楽しみください。
2. > 本日は暑い日が続いておりますね 🌞  体を冷やす爽やかなお料理はいかがでしょうか？ 当店おすすめの涼しいメニューをご紹介いたします！

**Section label:** `✦ AIがご提案 — 本日の暑さにぴったりな涼しいメニュー 6 選`

**Sản phẩm gợi ý (6 món "mát lạnh"):**

| Món | Giá | Lý do AI |
|---|---|---|
| 冷やしラーメン | ¥950 | 🌡️ 暑い日に体を冷やす |
| レモンソーダ | ¥480 | ✨ 今日のNo.1人気ドリンク |
| 冷製トマトパスタ | ¥1,080 | 🍅 さっぱり軽めのランチに |
| 蒸し鶏サラダ | ¥780 | 💪 低カロリーで栄養満点 |
| グリーンスムージー | ¥620 | Kiwi ビタミン補給に最適 |
| 冷やし中華 | ¥950 | 🔥 辛さで汗を流してスッキリ |

**Hiệu ứng UI:** Các card hiện tại bị làm mờ (blur), sau đó AI reveal từng card mới với animation.

---

## Case 6 – Tư vấn qua Voice Input (Giọng nói)

**File:** `chat_ai.html`

**Trigger:** Người dùng nhấn nút microphone, giả lập nhận diện giọng nói

**Kết quả mô phỏng (sau 2.5 giây):**
> お勧めののデザートはありますか？

Sau đó AI xử lý như một câu hỏi thông thường và trả lời với sản phẩm gợi ý.

**Trạng thái nút:** `recording` → pulse animation đỏ khi đang nghe

---

## Case 7 – Thêm vào giỏ hàng qua AI Chat

**File:** `chat_ai.html`, `chat_ai_project1.html`, `chat_ai_project2.html`

**Trigger:** Người dùng nhấn nút "追加" trên card sản phẩm trong chat

**AI xác nhận:**
> カートに追加しました。

**Logic giá:**
| Từ khóa | Giá thêm vào |
|---|---|
| ハンバーグ | +¥1,850 |
| 寿司 | +¥2,400 |
| Khác | +¥1,000 |

Cart total được cập nhật real-time trên mini cart góc phải màn hình.

---

## Case 8 – Tư vấn thêm thông tin nhà hàng

**File:** `chat_ai.html`

**Trigger:** Khi AI chào hỏi ban đầu (trước khi người dùng hỏi)

**AI cung cấp thêm thông tin:**
> 記念日のディナーにぴったりなコースもご用意しております。
> 地元の新鮮な食材をふんだんに使った料理をぜひお楽しみください。

---

## Case 9 – Tư vấn Đồ uống

**File:** `chat_ai.html`

**Trigger:** Người dùng hỏi: `おすすめのドリンクはございますか？`

**AI trả lời:**
> はい、当店自慢の地酒と、季節のフルーツジュースがおすすめです。🍶🍹

Kèm gợi ý sản phẩm (豊後牛ハンバーグ và 旬の海鮮にぎり trong demo).

---

## Case 10 – Chuyển đổi từ Chat sang Voice Order

**File:** `menu_ai_chat.html`

**Trigger:** Người dùng nhấn nút "入力オーダー" hoặc link "AIチャットに切替"

**Hành động:**
- `入力オーダー` → chuyển tới `menu_ai.html` (voice order mode)
- `AIチャットに切替` → chuyển tới `chat_ai.html`

---

## Luồng điều hướng giữa các màn hình AI

```
login.html
    └──> menu_ai.html (AI Menu – tiếng Nhật)
            ├──> chat_ai.html (AI Chat)
            ├──> menu_ai_detail.html (Chi tiết món)
            ├──> menu_ai_detail_wine.html (Chi tiết rượu)
            ├──> cart.html (Giỏ hàng)
            └──> order_status.html (Lịch sử đơn)

menu_ai_en.html (AI Menu – tiếng Anh)
    └──> menu_ai_detail_wine_en.html

menu_ai_chat.html (Chat Order)
    ├──> menu_ai.html
    └──> chat_ai.html
```

---

## Tổng quan các kịch bản tư vấn

| # | Case | Trigger | File |
|---|---|---|---|
| 1 | Chào hỏi & gợi ý ban đầu | Mở màn hình chat | chat_ai.html |
| 2 | Tư vấn theo độ phổ biến | Từ khóa "人気" | chat_ai.html |
| 3 | Tư vấn theo ngân sách | Từ khóa "予算" | chat_ai.html |
| 4 | Gợi ý hôm nay (mặc định) | Câu hỏi chung | chat_ai.html |
| 5 | Tư vấn theo thời tiết | AI tự động (nóng) | menu_ai_chat.html |
| 6 | Nhập bằng giọng nói | Nút microphone | chat_ai.html |
| 7 | Thêm giỏ hàng qua chat | Nút "追加" trên card | chat_ai.html |
| 8 | Thông tin nhà hàng | Tin nhắn chào ban đầu | chat_ai.html |
| 9 | Tư vấn đồ uống | Câu hỏi về ドリンク | chat_ai.html |
| 10 | Chuyển Voice/Chat | Nút chuyển đổi mode | menu_ai_chat.html |
