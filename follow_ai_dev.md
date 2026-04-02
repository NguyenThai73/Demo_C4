# Follow Dev - posSpeak AI Tu Van Mon An

## Kien Truc Tong The

```
┌─────────────────────────────────────────────────────────────────────┐
│                        FLUTTER APP (iPad)                           │
│                                                                     │
│  ┌──────────┐  ┌──────────────┐  ┌────────────┐  ┌──────────────┐  │
│  │ Menu     │  │ AI Suggest   │  │ Item       │  │ AI Chat      │  │
│  │ Screen   │→ │ Screen       │→ │ Detail     │  │ Overlay      │  │
│  └──────────┘  └──────────────┘  └────────────┘  └──────────────┘  │
│       │              │                 │                │           │
│       └──────────────┴─────────────────┴────────────────┘           │
│                              │                                      │
│                     Flutter Service Layer                           │
│                  (API Service + AI Service)                          │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                    HTTP REST API + SSE Stream
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          v                    v                    v
   ┌─────────────┐    ┌──────────────┐    ┌──────────────────┐
   │  RUBY BE    │    │  DIFY API    │    │  EXTERNAL API    │
   │             │    │              │    │                  │
   │ - Menu CRUD │    │ - Knowledge  │    │ - Weather API    │
   │ - Order     │    │   (menu,     │    │   (OpenWeather)  │
   │ - Cart      │    │    nguyen    │    │                  │
   │ - Session   │    │    lieu,     │    │ - Speech-to-Text │
   │ - User      │    │    thoi      │    │   (Google STT)   │
   │ - AI Log    │    │    tiet)     │    │                  │
   │             │    │ - Chat API   │    │ - Text-to-Speech │
   │  PostgreSQL │    │ - Streaming  │    │   (Google TTS)   │
   └─────────────┘    └──────────────┘    └──────────────────┘
```

---

## Luong Du Lieu Chi Tiet

```
KHACH VAO CUA HANG
       │
       v
┌─ Flutter ──────────────────────────────────────────────────────┐
│ 1. App detect table (QR code / NFC)                            │
│ 2. Goi Ruby BE: POST /api/sessions (tao session moi)          │
│ 3. Goi Ruby BE: GET /api/menus (lay menu mac dinh)            │
│ 4. Goi Ruby BE: GET /api/context (lay thoi tiet, gio, mua)    │
│ 5. Hien thi Menu Grid (6 mon regular)                          │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             v
┌─ AI Welcome (tu dong) ────────────────────────────────────────┐
│ 6. Flutter goi Ruby BE: POST /api/ai/welcome                  │
│    Body: { session_id, table_id, context: {weather, time} }    │
│                                                                │
│ 7. Ruby BE goi Dify API: POST /chat-messages (streaming)       │
│    - Dify tra ve loi chao + goi y dua tren knowledge           │
│    - Ruby BE stream response ve Flutter qua SSE                │
│                                                                │
│ 8. Flutter nhan SSE stream → hien thi tung ky tu tren banner   │
│    Phase 1: "Xin chao..." → Phase 2: "Hom nay troi nong..."   │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             v
┌─ AI Suggest (sau khi chao) ───────────────────────────────────┐
│ 9. Dify tra ve danh sach menu_item_ids de xuat                 │
│    Response: { message: "...", suggested_ids: [1,3,7,12,15,9] }│
│                                                                │
│ 10. Ruby BE map IDs → menu items day du (ten, gia, anh, ...)   │
│     Response ve Flutter: { ai_message, suggested_items[] }      │
│                                                                │
│ 11. Flutter:                                                    │
│     - Blur menu hien tai                                        │
│     - Hien Voice Guide (mic)                                    │
│     - Cho user tap mic hoac tu dong sau 3s                      │
│     - Swap menu → suggested items                               │
│     - Reveal tung card voi animation                            │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             v
┌─ Voice Input ─────────────────────────────────────────────────┐
│ 12. User tap Mic → Flutter bat dau ghi am                      │
│ 13. Gui audio → Google Speech-to-Text → nhan text              │
│ 14. Gui text ve Ruby BE: POST /api/ai/chat                     │
│     Body: { session_id, message: "cay hon", conversation_id }  │
│                                                                │
│ 15. Ruby BE goi Dify: POST /chat-messages                      │
│     - Dify dua tren knowledge → tra ve mon moi hoac giai thich │
│     - Stream ve Flutter                                         │
│                                                                │
│ 16. Flutter cap nhat UI theo response                           │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             v
┌─ Item Detail ─────────────────────────────────────────────────┐
│ 17. User chon 1 mon → Flutter: GET /api/menus/:id             │
│ 18. Dong thoi goi Ruby BE: POST /api/ai/explain               │
│     Body: { session_id, menu_item_id, conversation_id }        │
│                                                                │
│ 19. Ruby BE goi Dify → Dify giai thich ly do chon mon          │
│     (dua tren context: thoi tiet, so thich da biet)            │
│     Stream ve Flutter → hien tren AI banner                    │
│                                                                │
│ 20. Candidate nav: Ruby BE tra ve top 3 candidates tu Dify     │
│ 21. User them vao gio: POST /api/cart/items                    │
└───────────────────────────────────────────────────────────────┘
```

---

## RUBY BACKEND

### Database Schema

```
tables
  - id
  - table_number
  - status (available, occupied, reserved)

sessions
  - id
  - table_id
  - started_at
  - ended_at
  - language (ja, en, vi)
  - dify_conversation_id    ← luu conversation_id cua Dify de giu context

menus
  - id
  - name                    ← ten mon (Japanese)
  - name_en                 ← ten English
  - name_vi                 ← ten Vietnamese
  - description
  - price
  - image_url
  - category_id
  - calories
  - cooking_time_minutes
  - is_available
  - allergens[]             ← mang cac chat gay di ung
  - is_customizable
  - sort_order

categories
  - id
  - name (おすすめ, 前菜, 麺類, ドリンク, サラダ)
  - sort_order

menu_ingredients
  - id
  - menu_id
  - ingredient_id
  - quantity

ingredients
  - id
  - name
  - type (meat, fish, vegetable, dairy, grain...)
  - is_allergen

cart_items
  - id
  - session_id
  - menu_id
  - quantity
  - note
  - added_at

orders
  - id
  - session_id
  - status (pending, preparing, served, paid)
  - total_price
  - ordered_at

order_items
  - id
  - order_id
  - menu_id
  - quantity
  - price

ai_logs
  - id
  - session_id
  - role (system, user, assistant)
  - message
  - suggested_menu_ids[]
  - dify_conversation_id
  - created_at
```

### API Endpoints

```
# --- Session ---
POST   /api/sessions                  # Tao session khi khach ngoi ban
GET    /api/sessions/:id              # Lay session info

# --- Menu ---
GET    /api/menus                     # Lay tat ca menu (filter by category)
GET    /api/menus/:id                 # Chi tiet 1 mon
GET    /api/categories                # Lay danh sach category

# --- Cart ---
GET    /api/sessions/:id/cart         # Xem gio hang
POST   /api/sessions/:id/cart/items   # Them mon vao gio
DELETE /api/sessions/:id/cart/items/:item_id

# --- Order ---
POST   /api/sessions/:id/orders       # Dat mon (tu cart)
GET    /api/sessions/:id/orders       # Lich su order

# --- AI (proxy to Dify) ---
POST   /api/ai/welcome                # AI chao khach (Phase 1+2), tra ve SSE stream
POST   /api/ai/chat                   # User noi/nhap → AI tra loi, tra ve SSE stream
POST   /api/ai/explain                # AI giai thich mon, tra ve SSE stream

# --- Context ---
GET    /api/context                   # Lay thoi tiet, gio, mua hien tai
```

### Ruby BE → Dify Integration

```ruby
# app/services/dify_service.rb

class DifyService
  DIFY_BASE_URL = ENV['DIFY_BASE_URL']  # "https://api.dify.ai/v1"
  DIFY_API_KEY  = ENV['DIFY_API_KEY']

  # Goi Dify chat API voi streaming
  def chat(conversation_id:, message:, context: {})
    body = {
      inputs: {
        weather:     context[:weather],      # "35°C, sunny"
        time_of_day: context[:time_of_day],  # "lunch"
        season:      context[:season],       # "summer"
        language:    context[:language]       # "ja"
      },
      query:           message,
      response_mode:   "streaming",
      conversation_id: conversation_id,      # nil cho lan dau, Dify tra ve id moi
      user:            context[:session_id]
    }

    # POST to Dify, yield tung chunk SSE
    stream_post("/chat-messages", body) do |chunk|
      yield chunk  # { event: "message", answer: "あ" }
    end
  end

  private

  def stream_post(path, body, &block)
    uri = URI("#{DIFY_BASE_URL}#{path}")
    # ... HTTP streaming implementation
  end
end
```

```ruby
# app/controllers/api/ai_controller.rb

class Api::AiController < ApplicationController
  def welcome
    session = Session.find(params[:session_id])
    context = build_context(session)

    # System prompt cho Dify da duoc cau hinh trong Dify app
    # O day chi gui context
    message = "Khach moi vao ban #{session.table.table_number}. " \
              "Thoi tiet: #{context[:weather]}. " \
              "Hay chao khach va goi y mon phu hop."

    response.headers['Content-Type'] = 'text/event-stream'

    DifyService.new.chat(
      conversation_id: session.dify_conversation_id,
      message:         message,
      context:         context
    ) do |chunk|
      # Stream SSE ve Flutter
      response.stream.write("data: #{chunk.to_json}\n\n")

      # Neu Dify tra ve conversation_id moi → luu lai
      if chunk[:conversation_id] && session.dify_conversation_id.nil?
        session.update!(dify_conversation_id: chunk[:conversation_id])
      end

      # Neu Dify tra ve suggested_ids → luu log + tra menu items
      if chunk[:suggested_ids].present?
        items = Menu.where(id: chunk[:suggested_ids]).order_by_ids(chunk[:suggested_ids])
        response.stream.write("data: #{items.to_json}\n\n")
        AiLog.create!(session: session, role: 'assistant',
                       message: chunk[:answer], suggested_menu_ids: chunk[:suggested_ids])
      end
    end
  ensure
    response.stream.close
  end

  def chat
    session = Session.find(params[:session_id])
    context = build_context(session)

    response.headers['Content-Type'] = 'text/event-stream'

    DifyService.new.chat(
      conversation_id: session.dify_conversation_id,
      message:         params[:message],
      context:         context
    ) do |chunk|
      response.stream.write("data: #{chunk.to_json}\n\n")
    end
  ensure
    response.stream.close
  end

  private

  def build_context(session)
    weather = WeatherService.current  # Goi OpenWeather API (cache 30 min)
    {
      session_id:  session.id,
      weather:     "#{weather[:temp]}°C, #{weather[:condition]}",
      time_of_day: time_slot(Time.current.hour),
      season:      current_season,
      language:    session.language
    }
  end
end
```

### Dong Bo Data Len Dify Knowledge

```ruby
# app/services/dify_knowledge_sync.rb
# Chay dinh ky (cron job) hoac khi menu thay doi

class DifyKnowledgeSync
  DIFY_DATASET_API = "#{ENV['DIFY_BASE_URL']}/datasets"

  # Dong bo menu len Dify knowledge
  def sync_menus
    menus = Menu.includes(:category, :ingredients).where(is_available: true)

    document_content = menus.map do |m|
      <<~TEXT
        Mon: #{m.name}
        Category: #{m.category.name}
        Gia: ¥#{m.price}
        Mo ta: #{m.description}
        Calories: #{m.calories} kcal
        Thoi gian nau: #{m.cooking_time_minutes} phut
        Nguyen lieu: #{m.ingredients.map(&:name).join(', ')}
        Di ung: #{m.allergens.join(', ')}
        Tuy chinh: #{m.is_customizable ? 'Co' : 'Khong'}
        ID: #{m.id}
      TEXT
    end.join("\n---\n")

    update_dify_document("menu_items", document_content)
  end

  # Dong bo nguyen lieu
  def sync_ingredients
    ingredients = Ingredient.all
    content = ingredients.map { |i| "#{i.name} (#{i.type})" }.join("\n")
    update_dify_document("ingredients", content)
  end

  # Dong bo thoi tiet (chay moi 30 phut)
  def sync_weather
    weather = WeatherService.current
    content = <<~TEXT
      Thoi tiet hien tai:
      Nhiet do: #{weather[:temp]}°C
      Trang thai: #{weather[:condition]}
      Do am: #{weather[:humidity]}%
      Mua: #{current_season}
      Thoi gian: #{Time.current.strftime('%H:%M')}
      Buoi: #{time_slot(Time.current.hour)}
    TEXT
    update_dify_document("weather_context", content)
  end

  private

  def update_dify_document(doc_name, content)
    # Dify Dataset API: update or create document
    # POST /datasets/:dataset_id/documents
  end
end
```

---

## DIFY CONFIGURATION

### App Setup

```
App Type:        Chat Assistant
Model:           GPT-4o / Claude Sonnet (tuy chon)
Response Mode:   Streaming

Knowledge Base:
  1. menu_items       ← dong bo tu Ruby BE
  2. ingredients      ← dong bo tu Ruby BE
  3. weather_context  ← dong bo moi 30 phut
```

### System Prompt (cau hinh trong Dify)

```
Ban la "Sakura" (さくら), AI Staff cua nha hang posSpeak.
Nhiem vu: tu van mon an cho khach dua tren ngu canh.

QUY TAC:
1. Luon tra loi bang tieng Nhat (tru khi khach dung ngon ngu khac)
2. Dua tren thoi tiet, thoi gian, mua de goi y mon phu hop
3. Khi goi y mon, TRA VE JSON cuoi message voi format:
   <!--SUGGEST:{"menu_ids":[1,3,7],"reasons":{"1":"🌡️ lam mat","3":"✨ hot nhat","7":"🍅 nhe nhang"}}-->
4. Moi lan goi y toi da 6 mon
5. Ghi nho so thich khach da noi trong conversation
6. Neu khach hoi ve di ung → canh bao ro rang
7. Giu giong than thien, lich su, nhu nhan vien that

VI DU GOI Y:
- Troi nong (>30°C): mon lanh, do uong mat, salad
- Troi lanh (<15°C): ramen nong, lau, do uong am
- Bua trua: set menu, mon nhanh
- Bua toi: mon chinh, ruou, combo

CONTEXT (tu dong dien boi he thong):
- Thoi tiet: {{weather}}
- Buoi: {{time_of_day}}
- Mua: {{season}}
- Ngon ngu: {{language}}
```

### Dify Response Format (quy uoc)

```
# Response binh thuong (streaming text):
"ようこそいらっしゃいませ！当店へのご来店ありがとうございます。"

# Response co goi y mon (cuoi message):
"本日は暑い日ですね。涼しいメニューをご紹介します！"
<!--SUGGEST:{"menu_ids":[1,3,7,12,15,9],"reasons":{"1":"🌡️ 暑い日に体を冷やす","3":"✨ 今日のNo.1人気"}}-->

# Ruby BE parse tag <!--SUGGEST:...--> de lay menu_ids
# Chi stream phan text cho Flutter, xu ly suggest rieng
```

---

## FLUTTER APP

### Project Structure

```
lib/
├── main.dart
├── config/
│   └── app_config.dart              # API URLs, keys
├── models/
│   ├── menu_item.dart
│   ├── category.dart
│   ├── cart_item.dart
│   ├── session.dart
│   └── ai_response.dart
├── services/
│   ├── api_service.dart             # HTTP calls to Ruby BE
│   ├── ai_service.dart              # SSE stream handler
│   ├── speech_service.dart          # Google STT/TTS
│   └── session_service.dart         # Session management
├── providers/
│   ├── menu_provider.dart           # State: menu list
│   ├── cart_provider.dart           # State: cart
│   ├── ai_provider.dart             # State: AI messages, suggestions
│   └── session_provider.dart        # State: current session
├── screens/
│   ├── menu_screen.dart             # Menu grid (tuong ung menu_ai.html)
│   ├── menu_detail_screen.dart      # Chi tiet mon (tuong ung menu_ai_detail.html)
│   ├── cart_screen.dart
│   ├── order_history_screen.dart
│   └── chat_screen.dart
├── widgets/
│   ├── ai_banner.dart               # Banner streaming text + avatar
│   ├── streaming_text.dart          # Widget hien text tung ky tu
│   ├── menu_card.dart               # Card mon an voi AI badge
│   ├── voice_button.dart            # Nut microphone + animation
│   ├── candidate_navigator.dart     # Prev/Next candidates
│   ├── ai_chat_overlay.dart         # Bottom sheet chat
│   └── category_tabs.dart           # Tab category
```

### AI Service (SSE Stream)

```dart
// lib/services/ai_service.dart

class AiService {
  final String baseUrl;

  AiService({required this.baseUrl});

  /// Goi AI welcome, tra ve Stream<AiChunk>
  Stream<AiChunk> welcome({
    required String sessionId,
    required String tableId,
  }) async* {
    final url = Uri.parse('$baseUrl/api/ai/welcome');
    final request = http.Request('POST', url)
      ..headers['Content-Type'] = 'application/json'
      ..body = jsonEncode({
        'session_id': sessionId,
        'table_id': tableId,
      });

    final response = await http.Client().send(request);

    // Doc SSE stream
    await for (final chunk in response.stream
        .transform(utf8.decoder)
        .transform(const LineSplitter())) {

      if (chunk.startsWith('data: ')) {
        final data = jsonDecode(chunk.substring(6));
        yield AiChunk.fromJson(data);
      }
    }
  }

  /// Goi AI chat (user message)
  Stream<AiChunk> chat({
    required String sessionId,
    required String message,
  }) async* {
    final url = Uri.parse('$baseUrl/api/ai/chat');
    final request = http.Request('POST', url)
      ..headers['Content-Type'] = 'application/json'
      ..body = jsonEncode({
        'session_id': sessionId,
        'message': message,
      });

    final response = await http.Client().send(request);

    await for (final chunk in response.stream
        .transform(utf8.decoder)
        .transform(const LineSplitter())) {

      if (chunk.startsWith('data: ')) {
        final data = jsonDecode(chunk.substring(6));
        yield AiChunk.fromJson(data);
      }
    }
  }
}

/// Model cho moi chunk SSE
class AiChunk {
  final String? text;              // Phan text streaming
  final List<MenuItem>? suggested; // Danh sach mon de xuat (neu co)
  final String? conversationId;

  AiChunk({this.text, this.suggested, this.conversationId});

  factory AiChunk.fromJson(Map<String, dynamic> json) {
    return AiChunk(
      text: json['answer'],
      suggested: json['suggested_items'] != null
          ? (json['suggested_items'] as List)
              .map((e) => MenuItem.fromJson(e))
              .toList()
          : null,
      conversationId: json['conversation_id'],
    );
  }
}
```

### Streaming Text Widget

```dart
// lib/widgets/streaming_text.dart

class StreamingText extends StatefulWidget {
  final Stream<String> textStream;
  final TextStyle style;

  const StreamingText({required this.textStream, required this.style});

  @override
  State<StreamingText> createState() => _StreamingTextState();
}

class _StreamingTextState extends State<StreamingText> {
  String _displayText = '';
  bool _isStreaming = false;
  StreamSubscription? _sub;

  @override
  void initState() {
    super.initState();
    _sub = widget.textStream.listen(
      (char) => setState(() {
        _displayText += char;
        _isStreaming = true;
      }),
      onDone: () => setState(() => _isStreaming = false),
    );
  }

  @override
  void dispose() {
    _sub?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Flexible(child: Text(_displayText, style: widget.style)),
        if (_isStreaming)
          BlinkingCursor(),  // ▋ nhap nhay
      ],
    );
  }
}
```

### AI Provider (State Management)

```dart
// lib/providers/ai_provider.dart

class AiProvider extends ChangeNotifier {
  final AiService _aiService;

  String bannerText = '';
  bool isStreaming = false;
  List<MenuItem> suggestedItems = [];
  bool showVoiceGuide = false;
  AiPhase currentPhase = AiPhase.idle;

  AiProvider(this._aiService);

  /// Chay flow welcome (Phase 1 → 2 → cho mic)
  Future<void> startWelcome(String sessionId, String tableId) async {
    currentPhase = AiPhase.welcome;
    isStreaming = true;
    bannerText = '';
    notifyListeners();

    await for (final chunk in _aiService.welcome(
      sessionId: sessionId,
      tableId: tableId,
    )) {
      // Text streaming
      if (chunk.text != null) {
        bannerText += chunk.text!;
        notifyListeners();
      }

      // Dify tra ve suggested items
      if (chunk.suggested != null) {
        suggestedItems = chunk.suggested!;
        currentPhase = AiPhase.suggesting;
        showVoiceGuide = true;
        notifyListeners();
      }
    }

    isStreaming = false;
    notifyListeners();
  }

  /// Xu ly voice input
  Future<void> handleVoiceMessage(String sessionId, String message) async {
    showVoiceGuide = false;
    isStreaming = true;
    bannerText = '';
    notifyListeners();

    await for (final chunk in _aiService.chat(
      sessionId: sessionId,
      message: message,
    )) {
      if (chunk.text != null) {
        bannerText += chunk.text!;
        notifyListeners();
      }
      if (chunk.suggested != null) {
        suggestedItems = chunk.suggested!;
        notifyListeners();
      }
    }

    isStreaming = false;
    showVoiceGuide = true;
    notifyListeners();
  }
}

enum AiPhase { idle, welcome, suggesting, explaining, chatting }
```

---

## THU TU TRIEN KHAI

### Giai Doan 1: Backend + Dify (1-2 tuan)

```
[ ] 1.1  Setup Ruby project (Rails API mode)
[ ] 1.2  Tao database schema (migrations)
[ ] 1.3  Seed data menu, categories, ingredients
[ ] 1.4  API CRUD: menus, categories
[ ] 1.5  API: sessions, cart, orders
[ ] 1.6  Setup Dify app + knowledge base
[ ] 1.7  Upload menu data len Dify knowledge
[ ] 1.8  Cau hinh System Prompt tren Dify
[ ] 1.9  Test Dify chat qua Dify UI
[ ] 1.10 Tao DifyService trong Ruby (streaming)
[ ] 1.11 API: /api/ai/welcome (SSE stream)
[ ] 1.12 API: /api/ai/chat (SSE stream)
[ ] 1.13 API: /api/ai/explain (SSE stream)
[ ] 1.14 Parse <!--SUGGEST:--> tag tu Dify response
[ ] 1.15 Tich hop Weather API (OpenWeather)
[ ] 1.16 Cron job dong bo weather len Dify knowledge
```

### Giai Doan 2: Flutter Core (1-2 tuan)

```
[ ] 2.1  Setup Flutter project (iPad layout)
[ ] 2.2  Models: MenuItem, Category, CartItem, Session
[ ] 2.3  ApiService: goi Ruby BE (REST)
[ ] 2.4  AiService: SSE stream handler
[ ] 2.5  Providers: MenuProvider, CartProvider, SessionProvider
[ ] 2.6  Menu Screen: grid layout 3 cot
[ ] 2.7  MenuCard widget: anh + ten + gia + nut them
[ ] 2.8  CategoryTabs widget
[ ] 2.9  Bottom navigation bar
[ ] 2.10 Menu Detail Screen: anh lon + info panel
[ ] 2.11 Cart Screen
[ ] 2.12 Order History Screen
```

### Giai Doan 3: AI Integration (1-2 tuan)

```
[ ] 3.1  AiBanner widget (avatar + streaming text)
[ ] 3.2  StreamingText widget (hien tung ky tu)
[ ] 3.3  AiProvider: state management cho AI flow
[ ] 3.4  Ket noi AiService → AiBanner (SSE → UI)
[ ] 3.5  Welcome flow: Phase 1 → 2 → blur → suggest
[ ] 3.6  Card blur + reveal animation
[ ] 3.7  AI badge tren card (reason)
[ ] 3.8  VoiceButton widget (mic + pulse animation)
[ ] 3.9  Tich hop Google Speech-to-Text
[ ] 3.10 Voice → text → goi /api/ai/chat → stream response
[ ] 3.11 Menu Detail: AI explain (streaming banner)
[ ] 3.12 CandidateNavigator widget (prev/next)
[ ] 3.13 AiChatOverlay (bottom sheet + chips + input)
[ ] 3.14 Tich hop Google Text-to-Speech (AI doc text)
```

### Giai Doan 4: Hoan Thien (1 tuan)

```
[ ] 4.1  Da ngon ngu (ja/en/vi)
[ ] 4.2  Dong bo menu tu Ruby → Dify khi menu thay doi
[ ] 4.3  AI Log luu lai lich su tu van
[ ] 4.4  Error handling: mat ket noi, Dify timeout
[ ] 4.5  Offline fallback: hien menu mac dinh khi mat AI
[ ] 4.6  Performance: cache menu, lazy load anh
[ ] 4.7  Test toan bo flow tren iPad that
[ ] 4.8  Fix UI/UX issues
```

---

## LUU Y QUAN TRONG

### Dify Knowledge - Cach To Chuc

```
Dataset 1: "Menu Items"
  - Moi mon la 1 document segment
  - Bao gom: ten, gia, mo ta, nguyen lieu, calories, allergens, ID
  - Cap nhat khi menu thay doi (webhook tu Ruby)

Dataset 2: "Ingredients & Allergens"
  - Danh sach nguyen lieu va phan loai
  - Thong tin di ung

Dataset 3: "Context"
  - Thoi tiet hien tai (cap nhat moi 30 phut)
  - Thong tin mua, buoi trong ngay
  - Cac mon hot trong ngay (tu order data)
```

### SSE Stream Format (Ruby → Flutter)

```
# Text chunk:
data: {"event":"message","answer":"あ","conversation_id":"abc-123"}

# Suggested items (sau khi parse <!--SUGGEST:-->):
data: {"event":"suggest","suggested_items":[{"id":1,"name":"冷やしラーメン","price":950,"image_url":"...","reason":"🌡️ 暑い日に体を冷やす"}]}

# Done:
data: {"event":"done"}
```

### Xu Ly <!--SUGGEST:--> Tag

```ruby
# Ruby BE parse Dify response truoc khi stream cho Flutter

def parse_and_stream(dify_chunk, response_stream)
  text = dify_chunk[:answer]

  # Check co tag suggest khong
  if text.include?('<!--SUGGEST:')
    # Tach phan text va phan suggest
    clean_text = text.gsub(/<!--SUGGEST:.*?-->/, '')
    suggest_json = text.match(/<!--SUGGEST:(.*?)-->/)[1]
    suggest_data = JSON.parse(suggest_json)

    # Stream text (khong co tag)
    stream_text(clean_text, response_stream)

    # Stream suggested items (tra ve menu items day du)
    items = Menu.where(id: suggest_data['menu_ids'])
    items_with_reasons = items.map do |item|
      item.as_json.merge(reason: suggest_data['reasons'][item.id.to_s])
    end
    stream_suggest(items_with_reasons, response_stream)
  else
    stream_text(text, response_stream)
  end
end
```
