# Sơ Đồ Kiến Trúc Hệ Thống Chẩn Đoán Bệnh Tay-Chân-Miệng

## Kiến Trúc Tổng Quan

```
                    ┌─────────────┐
                    │  Bộ Giải    │
                    │   Thích     │
                    │  (Trace)    │
                    └──────┬──────┘
                           │
    ┌──────────┐           │           ┌──────────────┐
    │  Người   │◄─────────►│           │  Vùng Nhớ    │
    │  Dùng    │      ┌────┴─────┐     │  Làm Việc    │
    │          │      │          │     │  (Runtime    │
    └──────────┘      │  Giao    │     │   State)     │
                      │  Diện    │     └───────▲──────┘
    ┌──────────┐      │          │             │
    │  Kỹ Sư   │◄─────┤index.html│             │
    │ Tri Thức │      │script.js │             │
    │          │      │style.css │             │
    └──────────┘      └────┬─────┘             │
                           │                   │
                      ┌────┴──────┐            │
                      │  Quản Trí │            │
                      │ Tri Thức  │            │
                      │ (Flask    │            │
                      │  API)     │            │
                      └────┬──────┘            │
                           │                   │
        ╔══════════════════╧═══════════════════╧═════════════╗
        ║                                                     ║
        ║  ┌──────────────────────────────────────────────┐  ║
        ║  │         Động Cơ Suy Diễn                     │◄─╫──┐
        ║  │      (Inference Engine)                      │  ║  │
        ║  │  • Forward Chaining Algorithm                │  ║  │
        ║  │  • Priority-based Selection                  │  ║  │
        ║  │  • Sequential Diagnosis (4→3→2b→2a→1)        │  ║  │
        ║  │  • Rule Evaluation & Matching                │  ║  │
        ║  └────────────────┬─────────────────────────────┘  ║  │
        ║                   │                                 ║  │
        ║                   ▼                                 ║  │
        ║         ┌──────────────────┐                        ║  │
        ║         │   Cơ Sở Tri Thức │                        ║  │
        ║         │    (Knowledge    │                        ║  │
        ║         │      Base)       │                        ║  │
        ║         ├──────────────────┤                        ║  │
        ║         │ diagnosis_rules  │                        ║  │
        ║         │ classification_  │                        ║  │
        ║         │    level_rules   │                        ║  │
        ║         │   treatment.json │                        ║  │
        ║         └──────────────────┘                        ║  │
        ║                                                     ║  │
        ╚═════════════════════════════════════════════════════╝  │
                                                                 │
                                                    Kết quả suy diễn
```

## Luồng Xử Lý Chính

```
User Input → Frontend → API Endpoint → Inference Engine → Knowledge Base
                                                                │
                                                                ▼
User Output ← Frontend ← JSON Response ← Rule Evaluation ← Load Rules
```

## Công Nghệ Sử Dụng

| Tầng | Công Nghệ |
|------|-----------|
| **Frontend** | HTML5, CSS3, Vanilla JavaScript |
| **Backend** | Python Flask |
| **Logic** | Forward Chaining, Custom Inference Engine |
| **Data** | JSON (Knowledge Base) |
| **Deployment** | Gunicorn (Procfile ready) |

## Đặc Điểm Kiến Trúc

- **Kiến trúc phân tầng**: Tách biệt rõ ràng giữa UI, Logic, và Data
- **RESTful API**: Giao tiếp qua HTTP/JSON
- **Rule-based AI**: Sử dụng knowledge base dạng JSON
- **Dual Logic System**: 2 cơ chế suy luận khác nhau cho chẩn đoán và phân độ
- **Stateless**: Mỗi request độc lập, không lưu trữ session
