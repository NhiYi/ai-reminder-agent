# 🤖 AI Schedule Notification Agent

Hệ thống **AI Schedule Notification Agent** là một workflow được xây dựng bằng **n8n + Google Gemini + Telegram**, có khả năng tiếp nhận yêu cầu lịch hẹn từ người dùng, phân tích nội dung bằng AI và gửi thông báo nhắc lịch qua Telegram.

---

## 📌 1. Giới thiệu dự án

Dự án xây dựng một hệ thống thông báo lịch tự động sử dụng **2 AI Agents**.

Người dùng chỉ cần gửi tin nhắn tự nhiên thông qua Telegram, ví dụ:

> Ngày mai lúc 9 giờ sáng họp nhóm AI, nhắc tôi trước 30 phút.

Hệ thống sẽ:

1. Nhận tin nhắn từ Telegram.
2. Agent 1 phân tích thông tin lịch.
3. Xác định ngày, giờ, nội dung và thời gian nhắc.
4. Agent 2 kiểm tra và tạo nội dung thông báo.
5. Chờ theo khoảng thời gian được cấu hình.
6. Gửi thông báo trở lại Telegram.

---

## 🎯 2. Mục tiêu dự án

### Mục tiêu chính

- Xây dựng workflow Automation bằng n8n.
- Sử dụng tối thiểu **2 AI Agents**.
- Cho phép người dùng nhập lịch bằng ngôn ngữ tự nhiên.
- Sử dụng AI để phân tích thông tin lịch.
- Tạo thông báo nhắc lịch tự động.
- Gửi thông báo thông qua Telegram.

### Yêu cầu

- Có ít nhất 2 AI Agents.
- Hai Agent cùng tham gia xử lý Task A.
- Có Telegram Trigger.
- Có AI Model.
- Có Wait Node.
- Có Telegram Send Message.
- Workflow có thể chạy end-to-end.

---

# 🏗️ 3. Kiến trúc hệ thống

```text
┌──────────────────────┐
│       Telegram       │
│   User sends text    │
└──────────┬───────────┘
           │
           ▼
┌─────────────────────────────┐
│      Telegram Trigger       │
│  Nhận message từ Telegram   │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Agent 1 - Schedule Parser   │
│                             │
│ Phân tích:                  │
│ - Ngày                      │
│ - Giờ                       │
│ - Nội dung                  │
│ - Thời gian nhắc            │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Agent 2 - Reminder Assistant│
│                             │
│ Kiểm tra thông tin và       │
│ tạo nội dung nhắc lịch      │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│            Wait             │
│   Chờ theo thời gian cấu hình│
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│    Telegram Send Message    │
│       Gửi thông báo         │
└─────────────┬───────────────┘
              │
              ▼
        📱 Telegram User
```

---

# 🛠️ 4. Công nghệ sử dụng

| Công nghệ | Mục đích |
|---|---|
| n8n | Xây dựng Automation Workflow |
| Google Gemini | AI Model cho các AI Agent |
| Telegram Bot | Nhận và gửi tin nhắn |
| BotFather | Tạo Telegram Bot |
| GitHub | Quản lý source code và tài liệu |
| Markdown | Viết README |
| JSON | Lưu workflow n8n |

---

# 🤖 5. Agent 1 - Schedule Parser

## Vai trò

Agent 1 có nhiệm vụ phân tích tin nhắn của người dùng và trích xuất thông tin lịch.

### Input

Ví dụ:

```text
Ngày mai lúc 9 giờ sáng họp nhóm AI, nhắc tôi trước 30 phút.
```

### Output

Agent 1 phân tích thành:

```text
Ngày: 01/10/2026
Giờ: 09:00
Nội dung: Họp nhóm AI
Nhắc trước: 30 phút
```

> Ngày thực tế được tính dựa trên ngày hiện tại của workflow.

---

## System Message của Agent 1

```text
Bạn là Schedule Agent – AI Agent chuyên phân tích lịch hẹn.

Nhiệm vụ:
Phân tích tin nhắn của người dùng và trích xuất thông tin lịch.

Bạn cần xác định:
1. Ngày thực hiện
2. Giờ thực hiện
3. Nội dung lịch
4. Số phút cần nhắc trước

Quy tắc:
- Hiểu tiếng Việt tự nhiên.
- Hiểu các cách nói như hôm nay, ngày mai, thứ hai, 9 giờ sáng, 2 giờ chiều.
- Nếu người dùng không nói thời gian nhắc, mặc định nhắc trước 15 phút.
- Nếu thiếu ngày hoặc giờ thì phải thông báo rằng thông tin lịch chưa đầy đủ.
- Không tự ý thêm nội dung mà người dùng không cung cấp.
- Múi giờ sử dụng là Việt Nam GMT+7.

QUY TẮC NGÀY THÁNG RẤT QUAN TRỌNG:

- Luôn sử dụng "Ngày hiện tại" được cung cấp trong Prompt làm mốc tính ngày.
- "Hôm nay" = ngày hiện tại.
- "Ngày mai" = ngày hiện tại + 1 ngày.
- "Ngày kia" = ngày hiện tại + 2 ngày.
- Nếu người dùng không nói năm, sử dụng năm của ngày được tính.
- Tuyệt đối không tự sử dụng năm 2024 hoặc một ngày mặc định khác.
- Múi giờ là Asia/Ho_Chi_Minh (GMT+7).

Sau khi phân tích, trả về theo mẫu:

Ngày: [ngày]
Giờ: [giờ]
Nội dung: [nội dung]
Nhắc trước: [số phút] phút
```

---

# 📅 6. Xử lý ngày hiện tại

Để tránh AI hiểu sai các từ như:

- hôm nay
- ngày mai
- ngày kia

Prompt của Agent 1 sử dụng ngày hiện tại của n8n:

```text
Ngày hiện tại: {{ $now.setZone('Asia/Ho_Chi_Minh').toFormat('dd/MM/yyyy') }}
Múi giờ: Asia/Ho_Chi_Minh (GMT+7)

Tin nhắn người dùng:
{{ $json.message.text }}
```

Điều này giúp AI có mốc thời gian chính xác khi phân tích lịch.

---

# 🤖 7. Agent 2 - Reminder Assistant

## Vai trò

Agent 2 nhận kết quả từ Agent 1 và tạo nội dung thông báo lịch.

### Nhiệm vụ

1. Kiểm tra thông tin lịch.
2. Kiểm tra ngày.
3. Kiểm tra giờ.
4. Kiểm tra nội dung.
5. Kiểm tra thời gian nhắc.
6. Tạo thông báo dễ đọc bằng tiếng Việt.

---

## System Message của Agent 2

```text
Bạn là Reminder Agent.

Bạn nhận thông tin lịch đã được Schedule Agent phân tích.

Nhiệm vụ:
1. Kiểm tra thông tin lịch có đủ ngày, giờ, nội dung và thời gian nhắc.
2. Tạo một thông báo lịch ngắn gọn bằng tiếng Việt.
3. Thông báo phải có ngày, giờ và nội dung lịch.
4. Không tự ý thêm thông tin không có trong dữ liệu.
5. Giọng văn thân thiện, dễ đọc.

Mẫu:

🔔 NHẮC LỊCH

📅 Ngày: ...
⏰ Thời gian: ...
📝 Nội dung: ...

Còn ... phút nữa đến giờ.
```

---

# ⏱️ 8. Wait Node

Workflow sử dụng **Wait Node** để tạm dừng workflow trước khi gửi thông báo.

Trong quá trình kiểm thử, Wait Node được cấu hình:

```text
After Time Interval
10 Seconds
```

Mục đích của cấu hình 10 giây là giúp kiểm tra nhanh workflow end-to-end.

### Phiên bản hoàn thiện

Có thể nâng cấp Wait Node để:

```text
Thời gian thực hiện
        ↓
Trừ thời gian nhắc
        ↓
Thời điểm cần gửi thông báo
        ↓
Wait
        ↓
Telegram
```

Ví dụ:

```text
Lịch: 10:00
Nhắc trước: 30 phút

→ Thời điểm gửi thông báo: 09:30
```

---

# 📱 9. Telegram Send Message

Sau Wait Node, hệ thống sử dụng Telegram Send Message để gửi thông báo cho người dùng.

### Chat ID

Sử dụng Expression:

```text
{{ $('Telegram Trigger').item.json.message.chat.id }}
```

### Message

Sử dụng:

```text
{{ $json.output }}
```

---

# 🔄 10. Workflow hoàn chỉnh

Workflow hiện tại:

```text
Telegram Trigger
       ↓
Agent 1 - Schedule Parser
       ↓
Agent 2 - Reminder Assistant
       ↓
Wait
       ↓
Telegram Send Message
```

---

# 🧪 11. Kiểm thử

## Test Case 01 - Nhập lịch cơ bản

### Input

```text
Hôm nay lúc 9 giờ sáng họp nhóm AI.
```

### Expected Result

AI xác định:

```text
Ngày: ngày hiện tại
Giờ: 09:00
Nội dung: Họp nhóm AI
Nhắc trước: 15 phút
```

---

## Test Case 02 - Ngày mai

### Input

```text
Ngày mai lúc 2 giờ chiều học tiếng Trung.
```

### Expected Result

```text
Ngày: ngày mai
Giờ: 14:00
Nội dung: Học tiếng Trung
Nhắc trước: 15 phút
```

---

## Test Case 03 - Có thời gian nhắc

### Input

```text
Ngày mai 9 giờ sáng họp nhóm AI, nhắc trước 30 phút.
```

### Expected Result

```text
Ngày: ngày mai
Giờ: 09:00
Nội dung: Họp nhóm AI
Nhắc trước: 30 phút
```

---

## Test Case 04 - Thiếu giờ

### Input

```text
Ngày mai họp nhóm AI.
```

### Expected Result

Hệ thống thông báo rằng thông tin lịch chưa đầy đủ vì chưa có giờ.

---

## Test Case 05 - Thiếu ngày

### Input

```text
9 giờ sáng họp nhóm AI.
```

### Expected Result

Hệ thống yêu cầu người dùng cung cấp ngày hoặc xác định ngày theo quy tắc được cấu hình.

---

# 📊 12. Kết quả kiểm thử

| Test Case | Nội dung | Kết quả |
|---|---|---|
| TC01 | Lịch hôm nay | PASS |
| TC02 | Lịch ngày mai | PASS |
| TC03 | Có thời gian nhắc | PASS |
| TC04 | Thiếu giờ | PASS |
| TC05 | Thiếu ngày | PASS |
| TC06 | Telegram Trigger | PASS |
| TC07 | Agent 1 | PASS |
| TC08 | Agent 2 | PASS |
| TC09 | Wait Node | PASS |
| TC10 | Telegram Send Message | PASS |

---

# 🐛 13. Các lỗi đã gặp và cách xử lý

## Lỗi 1 - Agent 1 không nhận được tin nhắn

### Nguyên nhân

Sử dụng:

```text
{{ $json.chatInput }}
```

Trong khi dữ liệu thực tế của Telegram nằm tại:

```text
$json.message.text
```

### Cách sửa

```text
{{ $json.message.text }}
```

---

## Lỗi 2 - OpenAI API hết credit

Thông báo:

```text
Rate limit reached.
You have no credits remaining.
```

### Nguyên nhân

API của OpenAI có hệ thống billing riêng với tài khoản ChatGPT.

### Cách xử lý

Chuyển sang sử dụng Google Gemini API.

---

## Lỗi 3 - AI hiểu sai năm

AI có thể tự suy luận năm cũ nếu Prompt không cung cấp ngày hiện tại.

### Cách xử lý

Thêm:

```text
Ngày hiện tại:
{{ $now.setZone('Asia/Ho_Chi_Minh').toFormat('dd/MM/yyyy') }}
```

và quy định rõ:

```text
"Hôm nay" = ngày hiện tại
"Ngày mai" = ngày hiện tại + 1 ngày
"Ngày kia" = ngày hiện tại + 2 ngày
```

---

# 📁 14. Cấu trúc GitHub Repository

```text
AI-Schedule-Notification-Agent/
│
├── README.md
│
├── workflow/
│   └── ai-schedule-notification.json
│
├── docs/
│   ├── architecture.png
│   ├── agent1.png
│   ├── agent2.png
│   └── telegram-result.png
│
└── test-cases/
    └── test-case.xlsx
```

---

# 🔐 15. Bảo mật

Không được upload các thông tin bí mật lên GitHub:

```text
❌ Telegram Bot Token
❌ Google Gemini API Key
❌ OpenAI API Key
❌ Password
❌ n8n credentials
❌ Access Token
```

Ví dụ không được đưa trực tiếp API Key vào README:

```text
GOOGLE_API_KEY=xxxxxxxxxxxxxxxx
```

Thay vào đó chỉ mô tả:

```text
Google Gemini API Key
```

và cấu hình thông qua Credentials của n8n.

---

# 🚀 16. Export Workflow

Sau khi hoàn thành workflow trong n8n:

1. Mở workflow.
2. Chọn menu của workflow.
3. Chọn Export.
4. Export workflow dưới dạng JSON.
5. Đặt tên:

```text
ai-schedule-notification.json
```

6. Đưa file vào:

```text
workflow/
```

Repository:

```text
workflow/
└── ai-schedule-notification.json
```

---

# 🐙 17. Upload lên GitHub

Sau khi tạo repository:

### Bước 1

Upload:

```text
README.md
```

### Bước 2

Tạo folder:

```text
workflow/
```

và upload:

```text
ai-schedule-notification.json
```

### Bước 3

Tạo folder:

```text
docs/
```

và upload các ảnh screenshot.

### Bước 4

Tạo folder:

```text
test-cases/
```

và upload file test case.

---

# 📸 18. Screenshot cần đưa vào README

Nên chụp các màn hình sau:

### Ảnh 1 - Workflow tổng thể

Hiển thị:

```text
Telegram Trigger
       ↓
Agent 1
       ↓
Agent 2
       ↓
Wait
       ↓
Telegram
```

Lưu:

```text
docs/architecture.png
```

### Ảnh 2 - Agent 1

Chụp phần:

- Model
- System Message
- Prompt
- Expression

Lưu:

```text
docs/agent1.png
```

### Ảnh 3 - Agent 2

Chụp phần:

- Model
- System Message
- Prompt

Lưu:

```text
docs/agent2.png
```

### Ảnh 4 - Kết quả Telegram

Chụp tin nhắn Telegram mà Bot gửi.

Lưu:

```text
docs/telegram-result.png
```

---

# 💡 19. Hướng phát triển

Có thể nâng cấp hệ thống trong tương lai:

- Nhắc lịch chính xác theo giờ thực tế.
- Hỗ trợ nhiều lịch trong một tin nhắn.
- Lưu lịch vào Google Calendar.
- Lưu lịch vào Database.
- Cho phép sửa lịch.
- Cho phép xóa lịch.
- Cho phép xem danh sách lịch.
- Nhắc lại nhiều lần.
- Hỗ trợ Email.
- Hỗ trợ Discord.
- Hỗ trợ Zalo nếu có API phù hợp.
- Sử dụng Structured Output/JSON để truyền dữ liệu giữa các Agent.
- Thêm Webhook.
- Thêm logging và monitoring.

---

# 📈 20. Giá trị của dự án

Dự án thể hiện khả năng:

- Automation với n8n.
- AI Agent Workflow.
- Prompt Engineering.
- API Integration.
- Telegram Bot Integration.
- Xử lý dữ liệu đầu vào.
- Xử lý thời gian.
- Error Handling.
- Testing Workflow.
- Documentation.
- GitHub Project Management.

Đây có thể được sử dụng như một **mini project về Automation + Agentic AI** trong portfolio.

---

# ✅ 21. Checklist hoàn thành

- [x] Tạo Telegram Bot
- [x] Kết nối Telegram với n8n
- [x] Tạo Telegram Trigger
- [x] Tạo Agent 1
- [x] Kết nối Gemini
- [x] Agent 1 phân tích lịch
- [x] Xử lý ngày hiện tại
- [x] Tạo Agent 2
- [x] Agent 2 tạo thông báo
- [x] Tạo Wait Node
- [x] Tạo Telegram Send Message
- [x] Test toàn bộ workflow
- [ ] Nâng cấp Wait thành thời gian nhắc thực tế
- [ ] Export workflow JSON
- [ ] Upload workflow lên GitHub
- [ ] Upload screenshot
- [ ] Upload test case
- [x] Hoàn thiện README

---

# 👤 Author

**Nhi**

Project:

```text
AI Schedule Notification Agent
```

Technology:

```text
n8n + Google Gemini + Telegram
```

---

# 📄 License

Project được thực hiện với mục đích học tập, thực hành Automation và AI Agent.
