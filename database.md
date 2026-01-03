# Cấu trúc Database - Hệ thống chẩn đoán bệnh Tay Chân Miệng

Hệ thống sử dụng 3 file JSON chính để lưu trữ cơ sở tri thức, nằm trong thư mục `data/`:

## 1. File `diagnosis_rules.json` - Luật chẩn đoán lâm sàng

File này chứa các câu hỏi lâm sàng để thu thập triệu chứng từ bệnh nhân và xác định chẩn đoán ban đầu.

### Cấu trúc:
```json
{
  "metadata": {
    "description": "Hệ luật chẩn đoán lâm sàng bệnh Tay Chân Miệng (TCM)",
    "version": "1.0",
    "total_rules": 4,
    "author": "CS217 Team"
  },
  "clinical_questions": { ... }
}
```

### Các thành phần chính:

- **metadata**: Thông tin meta về file
  - `description`: Mô tả mục đích file
  - `version`: Phiên bản
  - `total_rules`: Tổng số luật
  - `author`: Tác giả

- **clinical_questions**: Các nhóm câu hỏi lâm sàng
  - `basic_info`: Thông tin cơ bản (tuổi, giới tính, ...)
  - `symptoms_stage_1`: Triệu chứng giai đoạn khởi phát (1-2 ngày)
  - `symptoms_stage_2`: Triệu chứng giai đoạn toàn phát (3-5 ngày)
  - `symptoms_stage_3`: Triệu chứng giai đoạn hồi phục (sau 5-7 ngày)

### Cấu trúc câu hỏi:
```json
{
  "id": "fever_status",
  "question": "Tình trạng sốt (nếu có)?",
  "type": "select",
  "required": true,
  "options": [
    {"value": "no_fever", "label": "Không sốt"},
    {"value": "mild_fever", "label": "Sốt nhẹ (< 38.5°C)"}
  ]
}
```

Các thuộc tính:
- `id`: Mã định danh câu hỏi
- `question`: Nội dung câu hỏi
- `type`: Loại câu hỏi (`number`, `select`, `yes_no`)
- `required`: Bắt buộc trả lời hay không
- `options`: Các lựa chọn (đối với type `select`)
- `validation`: Ràng buộc giá trị (min, max)

---

## 2. File `classification_level_rules.json` - Luật phân loại mức độ bệnh

File này chứa các luật để phân loại mức độ nghiêm trọng của bệnh (Độ 1, 2a, 2b, 3, 4) dựa trên triệu chứng.

### Cấu trúc:
```json
{
  "conclusion_rules": [
    {
      "id": "R1-1",
      "name": "Độ 1 - Trẻ chỉ có loét miệng",
      "grade": "1",
      "priority": 0,
      "conditions": [ ... ],
      "conclusion": { ... }
    }
  ]
}
```

### Các thành phần chính:

- **conclusion_rules**: Mảng các luật phân loại

### Cấu trúc từng luật:
```json
{
  "id": "R2a-1",
  "name": "Độ 2a - Giật mình < 2 lần/30 phút",
  "grade": "2a",
  "priority": 0,
  "conditions": [
    {
      "field": "startle_reflex",
      "operator": "==",
      "value": "rare"
    }
  ],
  "conclusion": {
    "disease_level": "2a",
    "description": "Bệnh có biến chứng thần kinh nhẹ"
  }
}
```

Các thuộc tính:
- `id`: Mã định danh luật (R + độ + số thứ tự)
- `name`: Tên mô tả luật
- `grade`: Độ nghiêm trọng (`"1"`, `"2a"`, `"2b"`, `"3"`, `"4"`)
- `priority`: Độ ưu tiên (số càng lớn càng ưu tiên)
- `conditions`: Mảng các điều kiện cần thỏa mãn
  - `field`: Tên trường triệu chứng
  - `operator`: Toán tử so sánh (`==`, `!=`, `>`, `<`, `>=`, `<=`)
  - `value`: Giá trị cần so sánh
- `conclusion`: Kết luận khi luật được kích hoạt
  - `disease_level`: Mức độ bệnh
  - `description`: Mô tả chi tiết

### Phân loại các độ bệnh:
- **Độ 1**: Bệnh không biến chứng (chỉ có loét miệng, phát ban)
- **Độ 2a**: Biến chứng thần kinh nhẹ (giật mình hiếm, tay chân cử động bất thường)
- **Độ 2b**: Biến chứng thần kinh nặng (giật mình thường xuyên, run chi)
- **Độ 3**: Suy tim, phù phổi cấp
- **Độ 4**: Sốc, suy hô hấp nặng

---

## 3. File `treatment.json` - Phương pháp điều trị

File này chứa các phương pháp điều trị tương ứng với từng mức độ bệnh.

### Cấu trúc:
```json
{
  "treatment_rules": [
    {
      "rule_id": "treatment_degree_1",
      "disease_level": "1",
      "disease_name": "Tay chân miệng không có biến chứng",
      "treatment_location": "Điều trị ngoại trú và theo dõi tại y tế cơ sở",
      "treatments": [ ... ]
    }
  ]
}
```

### Các thành phần chính:

- **treatment_rules**: Mảng các phương án điều trị

### Cấu trúc từng phương án điều trị:
```json
{
  "rule_id": "treatment_degree_2a",
  "disease_level": "2a",
  "disease_name": "Tay chân miệng có biến chứng thần kinh (độ 2a)",
  "treatment_location": "Nhập viện điều trị",
  "treatments": [
    {
      "category": "Dinh dưỡng",
      "interventions": [ ... ]
    },
    {
      "category": "Liều lượng thuốc sử dụng",
      "medications": [
        {
          "name": "Paracetamol",
          "dosage": "10-15 mg/kg/lần"
        }
      ]
    }
  ],
  "monitoring": {
    "vital_signs": [ ... ],
    "neurological": [ ... ]
  },
  "warning_signs": [ ... ]
}
```

Các thuộc tính:
- `rule_id`: Mã định danh phương án điều trị
- `disease_level`: Mức độ bệnh tương ứng
- `disease_name`: Tên bệnh
- `treatment_location`: Nơi điều trị (ngoại trú/nhập viện/ICU)
- `treatments`: Mảng các can thiệp điều trị
  - `category`: Loại can thiệp (Dinh dưỡng, Thuốc, Chăm sóc, Chế độ, ...)
  - `interventions`: Các biện pháp cụ thể
  - `medications`: Danh sách thuốc (nếu có)
    - `name`: Tên thuốc
    - `dosage`: Liều lượng
    - `route`: Đường dùng (optional)
    - `indication`: Chỉ định (optional)
- `monitoring`: Theo dõi (đối với các độ nặng)
  - `vital_signs`: Các dấu hiệu sinh tồn
  - `neurological`: Theo dõi thần kinh
  - `laboratory`: Xét nghiệm
- `warning_signs`: Dấu hiệu cảnh báo cần chuyển tuyến

---

## Mối quan hệ giữa các file

```
diagnosis_rules.json (Thu thập triệu chứng)
            ↓
classification_level_rules.json (Phân loại độ bệnh)
            ↓
treatment.json (Đưa ra phương án điều trị)
```

1. **diagnosis_rules.json** thu thập thông tin triệu chứng từ bệnh nhân
2. **classification_level_rules.json** sử dụng các triệu chứng để xác định mức độ bệnh (1, 2a, 2b, 3, 4)
3. **treatment.json** cung cấp phương pháp điều trị tương ứng với mức độ bệnh đã xác định

---

## Lưu ý kỹ thuật

- Tất cả các file đều sử dụng định dạng **JSON** (JavaScript Object Notation)
- Encoding: **UTF-8** để hỗ trợ tiếng Việt
- Cấu trúc tuân thủ chuẩn JSON schema để dễ dàng validate và mở rộng
- Các `id` và `field` sử dụng snake_case (chữ thường, phân cách bằng dấu `_`)
- Các giá trị boolean: `true`/`false`
- Các giá trị string: đặt trong dấu ngoặc kép `"..."`
