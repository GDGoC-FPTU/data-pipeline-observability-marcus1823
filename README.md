[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23574071&assignment_repo_type=AssignmentRepo)

# Day 10 Lab: Data Pipeline & Data Observability

**Student Email:** marcuschill1823@gmail.com
**Student ID:** 2A202600438
**Name:** Trương Minh Tiền

---

## Mô tả

Bài lab này xây dựng một **ETL Pipeline** cơ bản (Extract - Validate - Transform - Load) bằng Python + Pandas, và thực hiện thí nghiệm **Data Observability** để thấy rõ ảnh hưởng của chất lượng dữ liệu lên kết quả của AI Agent.

Những gì đã làm trong bài:

- **Extract:** Đọc dữ liệu sản phẩm từ `raw_data.json`.
- **Validate:** Loại bỏ các record không hợp lệ (price <= 0, category rỗng), in ra log số record được giữ / bị loại.
- **Transform:** Tính `discounted_price = price * 0.9`, chuẩn hoá `category` về Title Case, thêm cột `processed_at` (timestamp) để phục vụ observability.
- **Load:** Xuất kết quả ra `processed_data.csv`.
- **Stress Test:** Chạy `agent_simulation.py` với 2 bộ dữ liệu (clean vs garbage) để so sánh và viết báo cáo trong `experiment_report.md`.

---

## Cách chạy (How to Run)

### Prerequisites

```bash
pip install pandas
```

### Chạy ETL Pipeline

```bash
python solution.py
```

Sau khi chạy sẽ tạo ra file `processed_data.csv`.

### Chạy Agent Simulation (Stress Test)

```bash
# Bước 1: Sinh file garbage_data.csv (dữ liệu "rác" có chủ đích)
python generate_garbage.py

# Bước 2: Chạy Agent với cả 2 bộ dữ liệu (clean + garbage) để so sánh
python agent_simulation.py
```

Kết quả Agent với từng bộ dữ liệu sẽ được ghi lại trong `experiment_report.md`.

---

## Cấu trúc thư mục

```
├── raw_data.json            # Dữ liệu gốc (input cho pipeline)
├── solution.py              # ETL Pipeline script (Extract/Validate/Transform/Load)
├── generate_garbage.py      # Script sinh dữ liệu "rác" để stress test
├── agent_simulation.py      # Agent RAG-like để test chất lượng dữ liệu
├── processed_data.csv       # Output của pipeline (clean data)
├── garbage_data.csv         # Dữ liệu rác (duplicate, wrong type, outlier, null)
├── experiment_report.md     # Báo cáo thí nghiệm Clean vs Garbage
└── README.md                # File này
```

---

## Kết quả

**ETL Pipeline (`solution.py`):**

- Input: 5 records từ `raw_data.json`.
- Validate: Giữ lại **3 records hợp lệ** (Laptop, Chair, Monitor); loại **2 records** (id=3 price <= 0, id=4 missing category).
- Transform: Tính giảm giá 10%, chuẩn hoá category, thêm `processed_at`.
- Load: Xuất **3 records** ra `processed_data.csv`.

**Agent Simulation (Stress Test):**

- Với **Clean Data** (`processed_data.csv`): Agent trả lời **đúng** — "Laptop at $1200".
- Với **Garbage Data** (`garbage_data.csv`): Agent trả lời **sai** — "Nuclear Reactor at $999999" (bị đánh lừa bởi outlier).

=> Kết luận: **Quality Data > Quality Prompt**. Nếu dữ liệu đầu vào không được validate, dù prompt có tốt đến đâu Agent cũng sẽ đưa ra kết quả sai. Chi tiết phân tích ở `experiment_report.md`.
