# Giới Thiệu Hệ Thống Chẩn Đoán Bệnh Tay Chân Miệng (HFMD)

## 📌 Tổng quan

Hệ thống chuyên gia chẩn đoán và phân loại mức độ nghiêm trọng của bệnh **Tay-Chân-Miệng (Hand-Foot-Mouth Disease - HFMD)** cho trẻ em, sử dụng **forward chaining inference** dựa trên cơ sở tri thức y khoa.

### Mục đích
- Hỗ trợ bác sĩ, nhân viên y tế trong việc chẩn đoán nhanh và chính xác
- Phân loại mức độ bệnh (Độ 1, 2a, 2b, 3, 4) để đưa ra phương án điều trị phù hợp
- Cung cấp giải thích rõ ràng về quyết định chẩn đoán dựa trên luật y khoa

---

## 🏗️ Kiến trúc hệ thống

### Mô hình 2 tầng (Client-Server)

```
┌─────────────────────────────────────────┐
│           FRONTEND (Client)             │
│  HTML5 + CSS3 + Vanilla JavaScript      │
│  - Giao diện nhập triệu chứng           │
│  - Hiển thị kết quả chẩn đoán           │
│  - Gợi ý điều trị                       │
└─────────────────┬───────────────────────┘
                  │ HTTP/JSON API
                  │
┌─────────────────▼───────────────────────┐
│           BACKEND (Server)              │
│  Flask + Python 3.11+                   │
│  - REST API endpoints                   │
│  - Forward Chaining Engine              │
│  - Knowledge Base Management            │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│        KNOWLEDGE BASE (Data)            │
│  JSON Files                             │
│  - diagnosis_rules.json                 │
│  - classification_level_rules.json      │
│  - treatment.json                       │
└─────────────────────────────────────────┘
```

---

## 🎨 Frontend (Giao diện người dùng)

### Công nghệ sử dụng
- **HTML5**: Cấu trúc trang web, semantic markup
- **CSS3**: Styling hiện đại, responsive design
  - Flexbox/Grid layout
  - Gradients, shadows, animations
  - Mobile-friendly
- **Vanilla JavaScript**: Logic xử lý phía client
  - Fetch API để gọi REST API
  - Dynamic DOM manipulation
  - Form validation
  - Event handling

### Tính năng
- ✅ Giao diện 2 giai đoạn (Phase):
  - **Phase 1**: Chẩn đoán lâm sàng (có bệnh TCM hay không)
  - **Phase 2**: Phân độ bệnh (Độ 1-4)
- ✅ Form nhập liệu động với validation
- ✅ Hiển thị kết quả trực quan với màu sắc phân biệt độ bệnh
- ✅ Giải thích luật y khoa được áp dụng
- ✅ Gợi ý phác đồ điều trị chi tiết
- ✅ Test cases mẫu để demo nhanh

### File structure
```
templates/
  └── index.html          # Giao diện chính
static/
  ├── style.css          # CSS styling
  └── script.js          # JavaScript logic
```

---

## ⚙️ Backend (Máy chủ xử lý)

### Công nghệ sử dụng

#### Framework & Libraries
- **Flask 3.0.0**: Web framework nhẹ, dễ mở rộng
- **Flask-CORS**: Xử lý Cross-Origin Resource Sharing
- **Gunicorn**: WSGI server cho production deployment
- **Python 3.11+**: Ngôn ngữ lập trình chính

#### Utilities
- **python-dotenv**: Quản lý biến môi trường
- **json**: Xử lý dữ liệu cơ sở tri thức
- **pandas** (optional): Xử lý dữ liệu nâng cao

### REST API Endpoints

#### 1. `POST /api/diagnose` - Chẩn đoán lâm sàng
**Input**: Triệu chứng ban đầu (sốt, loét miệng, phát ban, ...)
```json
{
  "age_months": 36,
  "fever_status": "high_fever",
  "mouth_ulcer": true,
  "rash_hand_foot_mouth": true,
  "contact_patient": true
}
```

**Output**: Kết quả có bệnh TCM hay không
```json
{
  "success": true,
  "has_hfmd": true,
  "confidence": "Cao",
  "diagnosis_type": "Điển hình",
  "message": "Bệnh nhân có biểu hiện điển hình của TCM",
  "matched_rules": ["R0-1"]
}
```

#### 2. `POST /api/classify` - Phân độ bệnh
**Input**: Triệu chứng chi tiết (giật mình, mạch, huyết áp, SpO2, ...)
```json
{
  "startle_per_30min": 3,
  "hr_no_fever": 140,
  "ataxia": true,
  "spo2": 93
}
```

**Output**: Mức độ bệnh (1, 2a, 2b, 3, 4)
```json
{
  "success": true,
  "disease_level": "2b",
  "description": "Biến chứng thần kinh rõ",
  "matched_rules": [
    {
      "id": "R2b-2",
      "name": "Độ 2b - Giật mình ≥ 2 lần/30 phút",
      "conclusion": {...}
    }
  ]
}
```

#### 3. `POST /api/treatment` - Gợi ý điều trị
**Input**: Độ bệnh
```json
{
  "disease_level": "2b"
}
```

**Output**: Phác đồ điều trị chi tiết
```json
{
  "success": true,
  "treatment": {
    "treatment_location": "Nhập viện điều trị",
    "treatments": [...],
    "monitoring": {...},
    "warning_signs": [...]
  }
}
```

#### 4. `GET /api/diagnosis-questions` - Lấy danh sách câu hỏi
**Output**: Tất cả câu hỏi lâm sàng từ knowledge base

### File structure
```
backend/
  ├── simple_inference.py      # Forward Chaining Engine
  ├── knowledge_base.py         # Knowledge Base Manager
  ├── tcm_diagnosis.py          # TCM-specific logic
  └── test_degree*.py           # Unit tests
app.py                          # Flask application entry point
```

---

## 🧠 Phương pháp suy luận (Inference Methods)

### 1. Forward Chaining (Suy luận tiến)

Hệ thống sử dụng **forward chaining** - phương pháp suy luận từ dữ liệu (data-driven):

```
Dữ liệu (Triệu chứng) → Luật (Rules) → Kết luận (Diagnosis)
```

#### Nguyên lý hoạt động:
1. **Nhận dữ liệu đầu vào**: Triệu chứng, chỉ số sinh tồn của bệnh nhân
2. **Khớp luật (Pattern Matching)**: Kiểm tra các luật trong knowledge base
3. **Đánh giá điều kiện**: Xem luật nào thỏa mãn với dữ liệu hiện tại
4. **Kích hoạt luật (Fire Rule)**: Áp dụng kết luận của luật phù hợp
5. **Giải thích**: Trả về luật đã sử dụng để người dùng hiểu quyết định

### 2. Chiến lược giải quyết xung đột (Conflict Resolution)

Khi có nhiều luật cùng thỏa mãn, hệ thống sử dụng 2 chiến lược:

#### A. Phân độ bệnh (Classification Rules)
**Chiến lược**: Duyệt từ độ cao xuống độ thấp
```
Độ 4 (Nguy kịch) → Độ 3 (Nặng) → Độ 2b (Trung bình nặng) 
→ Độ 2a (Trung bình) → Độ 1 (Nhẹ)
```
- Chọn độ cao nhất (nghiêm trọng nhất) nếu có nhiều luật match
- Ví dụ: Nếu match cả Độ 2a và Độ 2b → Chọn Độ 2b

#### B. Chẩn đoán lâm sàng (Diagnosis Rules)
**Chiến lược**: Độ tin cậy (Confidence-based)
- Chẩn đoán xác định bằng RT-PCR: Confidence = "Xác định"
- Triệu chứng điển hình: Confidence = "Cao"
- Dịch tễ học hỗ trợ: Confidence = "Trung bình"

### 3. Logic đánh giá điều kiện

#### Điều kiện đơn (Simple Condition)
```json
{
  "field": "spo2",
  "operator": "<",
  "value": 92
}
```
Ý nghĩa: SpO2 < 92%

#### Điều kiện kết hợp (Compound Condition)
```json
{
  "type": "OR",
  "conditions": [
    {"field": "mouth_ulcer", "operator": "==", "value": true},
    {"field": "rash_hand_foot_mouth", "operator": "==", "value": true}
  ]
}
```
Ý nghĩa: Có loét miệng HOẶC có phát ban tay chân miệng

### 4. Các toán tử được hỗ trợ
- `==`: Bằng
- `!=`: Khác
- `<`, `<=`: Nhỏ hơn, nhỏ hơn hoặc bằng
- `>`, `>=`: Lớn hơn, lớn hơn hoặc bằng
- `in`: Nằm trong danh sách

---

## 📚 Cơ sở tri thức (Knowledge Base)

### Cấu trúc 3 file JSON

#### 1. `diagnosis_rules.json` (Luật chẩn đoán)
- **Mục đích**: Xác định có bệnh TCM hay không
- **Số lượng**: 4 luật chẩn đoán + danh sách câu hỏi lâm sàng
- **Nội dung**:
  - Câu hỏi thu thập triệu chứng (clinical_questions)
  - Luật chẩn đoán dựa trên triệu chứng điển hình
  - Luật chẩn đoán dựa trên dịch tễ học
  - Luật cảnh báo biến chứng sớm
  - Luật xác định bằng RT-PCR

#### 2. `classification_level_rules.json` (Luật phân độ)
- **Mục đích**: Phân loại mức độ nghiêm trọng
- **Số lượng**: 38 luật phân độ
- **Nội dung**:
  - **Độ 1** (3 luật): Không biến chứng
  - **Độ 2a** (7 luật): Biến chứng thần kinh nhẹ
  - **Độ 2b** (10 luật): Biến chứng thần kinh rõ
  - **Độ 3** (10 luật): Rối loạn thần kinh thực vật nặng
  - **Độ 4** (6 luật): Suy hô hấp tuần hoàn nặng

#### 3. `treatment.json` (Phác đồ điều trị)
- **Mục đích**: Gợi ý điều trị cho từng độ bệnh
- **Số lượng**: 5 phác đồ (độ 1, 2a, 2b, 3, 4)
- **Nội dung**:
  - Địa điểm điều trị (ngoại trú/nhập viện/ICU)
  - Dinh dưỡng
  - Thuốc và liều lượng
  - Chăm sóc tại chỗ
  - Theo dõi (vital signs, neurological, laboratory)
  - Dấu hiệu cảnh báo

### Ví dụ cấu trúc luật

```json
{
  "id": "R2b-2",
  "name": "Độ 2b - Giật mình ≥ 2 lần/30 phút",
  "grade": "2b",
  "conditions": [
    {
      "field": "startle_per_30min",
      "operator": ">=",
      "value": 2
    }
  ],
  "conclusion": {
    "disease_level": "2b",
    "description": "Biến chứng thần kinh rõ - Giật mình ≥ 2 lần/30 phút"
  }
}
```

---

## 🔄 Quy trình hoạt động

### Workflow tổng thể

```
┌─────────────────────────────────────────────────────┐
│  BƯỚC 1: Nhập triệu chứng ban đầu                   │
│  - Sốt, loét miệng, phát ban, tiếp xúc bệnh nhân... │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│  BƯỚC 2: CHẨN ĐOÁN LÂM SÀNG (Phase 1)              │
│  API: POST /api/diagnose                            │
│  Engine: diagnosis_engine                           │
│  Rules: diagnosis_rules.json                        │
│  Output: has_hfmd = true/false                      │
└────────────────────┬────────────────────────────────┘
                     │
                     ├─── NO ──→ Không có TCM → Dừng
                     │
                     └─── YES
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│  BƯỚC 3: Nhập triệu chứng chi tiết                  │
│  - Giật mình, mạch, huyết áp, SpO2, tri giác...     │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│  BƯỚC 4: PHÂN ĐỘ BỆNH (Phase 2)                    │
│  API: POST /api/classify                            │
│  Engine: classification_engine                      │
│  Rules: classification_level_rules.json             │
│  Output: disease_level = 1/2a/2b/3/4                │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│  BƯỚC 5: GỢI Ý ĐIỀU TRỊ                             │
│  API: POST /api/treatment                           │
│  Data: treatment.json                               │
│  Output: Phác đồ điều trị chi tiết                  │
└─────────────────────────────────────────────────────┘
```

### Chi tiết từng bước

#### 1. Thu thập triệu chứng ban đầu
- Người dùng nhập thông tin qua form HTML
- JavaScript validate dữ liệu
- Gửi request đến `/api/diagnose`

#### 2. Chẩn đoán lâm sàng
```python
# Backend: app.py
diagnosis_engine.diagnose(patient_data)
  → Đọc diagnosis_rules.json
  → Khớp luật với dữ liệu
  → Trả về has_hfmd = true/false
```

#### 3. Phân độ bệnh (nếu có TCM)
```python
# Backend: app.py
classification_engine.diagnose(patient_data)
  → Đọc classification_level_rules.json
  → Duyệt từ Độ 4 → 3 → 2b → 2a → 1
  → Chọn độ cao nhất match
  → Trả về disease_level
```

#### 4. Gợi ý điều trị
```python
# Backend: app.py
Tìm treatment_rule theo disease_level trong treatment.json
  → Trả về phác đồ điều trị
```

#### 5. Hiển thị kết quả
- JavaScript nhận response
- Render kết quả lên giao diện
- Hiển thị luật được áp dụng
- Hiển thị phác đồ điều trị

---

## 🧪 Testing & Validation

### Unit Tests
```
backend/
  ├── test_degree1.py     # Test cases cho Độ 1
  ├── test_degree4.py     # Test cases cho Độ 4
  └── check_degree4.py    # Validation cho Độ 4
```

### Test Cases mẫu
```javascript
// Frontend: script.js
const DIAGNOSIS_EXAMPLES = [
  {
    name: "CÓ TCM - Loét miệng + Phát ban",
    diagnosis: {
      mouth_ulcer: true,
      rash_hand_foot_mouth: true
    }
  },
  // ... more test cases
]
```

### Validation
- ✅ Kiểm tra tính hợp lệ của dữ liệu đầu vào
- ✅ Kiểm tra logic luật (conditions, operators)
- ✅ Kiểm tra kết quả với cases y khoa thực tế

---

## 🚀 Deployment

### Development
```bash
# Activate virtual environment
cs217_venv\Scripts\activate

# Run Flask development server
python app.py
```
Truy cập: `http://localhost:5000`

### Production
```bash
# Use Gunicorn WSGI server
gunicorn -w 4 -b 0.0.0.0:8000 app:app
```

### Environment Variables
```bash
# .env file
FLASK_ENV=production
FLASK_DEBUG=False
SECRET_KEY=your-secret-key
```

---

## 📊 Ưu điểm của hệ thống

### 1. Kiến trúc
- ✅ **Modular**: Tách biệt Frontend, Backend, Knowledge Base
- ✅ **Scalable**: Dễ mở rộng thêm luật, triệu chứng mới
- ✅ **Maintainable**: Code rõ ràng, dễ bảo trì

### 2. Cơ sở tri thức
- ✅ **Chuẩn y khoa**: Dựa trên tài liệu chính thức
- ✅ **JSON format**: Dễ đọc, dễ chỉnh sửa
- ✅ **Versioning**: Có thể theo dõi thay đổi

### 3. Suy luận
- ✅ **Forward chaining**: Phù hợp với chẩn đoán y khoa
- ✅ **Explainable AI**: Giải thích được quyết định
- ✅ **Conflict resolution**: Chiến lược rõ ràng

### 4. Giao diện
- ✅ **User-friendly**: Dễ sử dụng, trực quan
- ✅ **Responsive**: Hoạt động tốt trên mọi thiết bị
- ✅ **Fast**: Không cần framework nặng

---

## 🔮 Hướng phát triển

### Tính năng mở rộng
- [ ] Lưu trữ lịch sử bệnh án
- [ ] Xuất báo cáo PDF
- [ ] Multi-language support
- [ ] Machine learning để cải thiện độ chính xác
- [ ] Tích hợp với hệ thống bệnh viện

### Cải tiến kỹ thuật
- [ ] Database (PostgreSQL/MySQL) thay vì JSON
- [ ] Authentication & Authorization
- [ ] Real-time collaboration
- [ ] API documentation (Swagger/OpenAPI)
- [ ] Docker containerization

---

## 👥 Đội ngũ phát triển

**CS217 Team**
- Môn học: Mô hình hóa tri thức và suy diễn
- Framework tham khảo: COPD Expert System

---

## 📄 License & Credits

### Nguồn tri thức
- Tài liệu y khoa chính thức về bệnh Tay Chân Miệng
- Hướng dẫn chẩn đoán và điều trị của Bộ Y tế

### Technology Stack
- Python Flask Framework
- Vanilla JavaScript
- HTML5/CSS3

---

**Phiên bản**: 1.0  
**Ngày cập nhật**: December 2025
