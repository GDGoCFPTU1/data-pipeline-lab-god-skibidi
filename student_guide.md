# 2A202600141-NguyenCongNhatTan

# Student Guide: The Multi-Modal Minefield Lab

## 1. Tổng quan
Chào mừng bạn đến với bài lab về **Data Pipeline Engineering**. Trong bài tập này, bạn sẽ làm việc với dữ liệu phi cấu trúc (Unstructured Data) - loại dữ liệu chiếm 80% thế giới thực nhưng cực kỳ khó xử lý.

**Mục tiêu**: Xây dựng một đường ống (Pipeline) tự động để đọc, làm sạch, kiểm tra chất lượng và hợp nhất dữ liệu từ hai nguồn: **PDF (Văn bản OCR)** và **Video (Metadata & Transcript)**.

---

## 2. Phân vai trong nhóm (Roles)
Để mô phỏng một đội ngũ kỹ sư thực tế, nhóm bạn sẽ chia thành 4 vai trò:

| Vai trò | File phụ trách | Nhiệm vụ chính |
| :--- | :--- | :--- |
| **Lead Data Architect** | `schema.py` | Thiết kế "Hợp đồng dữ liệu" chung cho cả team. |
| **ETL Builder** | `process_unstructured.py` | Viết logic chuyển đổi dữ liệu thô và làm sạch text (Regex). |
| **Observability Engineer** | `quality_check.py` | Xây dựng "Cổng kiểm soát chất lượng" (Quality Gates). |
| **DevOps Specialist** | `orchestrator.py` | Kết nối các thành phần và vận hành toàn bộ Pipeline. |

---

## 3. Hướng dẫn các bước thực hiện

### Bước 1: Chuẩn bị môi trường (Venv)
Việc sử dụng môi trường ảo giúp tránh xung đột thư viện giữa các bài tập khác nhau.

1.  **Tạo môi trường ảo**:
    ```powershell
    python -m venv .venv
    ```
2.  **Kích hoạt môi trường ảo**:
    *   **Trên Windows (PowerShell)**:
        ```powershell
        .\.venv\Scripts\Activate.ps1
        ```
    *   **Trên Windows (CMD)**:
        ```cmd
        .venv\Scripts\activate
        ```
    *   **Trên Mac/Linux**:
        ```bash
        source .venv/bin/activate
        ```
3.  **Cài đặt thư viện**:
    Sau khi đã kích hoạt (thấy chữ `(.venv)` ở đầu dòng lệnh), hãy chạy:
    ```bash
    pip install -r requirements.txt
    ```

### Bước 2: Thực hiện bài tập (Starter Code)
Các bạn di chuyển vào thư mục `starter_code/` và hoàn thiện các phần có đánh dấu `TODO`:

1.  **Thiết kế Schema**: Architect định nghĩa các trường dữ liệu chuẩn.
2.  **Xử lý dữ liệu**: Builder viết logic map dữ liệu từ PDF/Video sang Schema chuẩn. Đừng quên dùng Regex để xóa Header/Footer trong PDF.
3.  **Kiểm soát chất lượng**: Watchman viết logic để chặn dữ liệu rác hoặc dữ liệu có chứa mã lỗi (Toxic content).
4.  **Vận hành**: Connector viết vòng lặp để xử lý tất cả các file trong thư mục `raw_data`.

### Bước 3: Chạy và Kiểm tra
*   **Chạy Pipeline**:
    ```bash
    python starter_code/orchestrator.py
    ```
    Nếu thành công, bạn sẽ thấy file `processed_knowledge_base.json` xuất hiện ở thư mục gốc.
*   **Chạy Test tự động (Autograde)**:
    ```bash
    pytest tests/test_lab.py
    ```

---

## 4. Tiêu chí chấm điểm
*   **Execution (40%)**: Pipeline chạy trơn tru, làm sạch được nhiễu trong PDF.
*   **Observability (20%)**: Phát hiện và loại bỏ được các bản ghi lỗi (ví dụ: `doc2_corrupt.json`).
*   **Harmonization (30%)**: Không có xung đột schema (Dữ liệu từ 2 nguồn phải đồng nhất).
*   **Final Result (10%)**: File kết quả đúng định dạng JSON List.

---

## 5. Báo Cáo Giải Pháp & Code (Solution Report)
Dưới đây là phần giải thích chi tiết toàn bộ các `TODO` đã hoàn thành trong hệ thống, bạn có thể tham khảo hoặc trình bày với giảng viên chức năng của từng file mạch lạc như sau:

### 5.1. `schema.py` (Vai trò Lead Data Architect)
**Mục đích:** Xây dựng Data Contract (Bản thiết kế kiểu dữ liệu chuẩn tên là `UnifiedDocument`) để bắt tất cả dữ liệu từ PDF hoặc Video đều phải đóng gói dưới cấu trúc này. Điều này tránh được lỗi **Conflict Schema** (Ví dụ bên viết `doc_id`, bên viết `videoID`).

**Code:**
```python
class UnifiedDocument(BaseModel):
    document_id: str
    source_type: str
    author: str
    category: str
    content: str
    timestamp: str
```

### 5.2. `process_unstructured.py` (Vai trò ETL Builder)
**Mục đích:** Lấy dữ liệu RAW rác rưởi do hệ thống OCR hoặc Speech-To-Text sinh ra, dọn dẹp kỹ và nhét nó vào chuẩn `UnifiedDocument`.

**Code:**
```python
def process_pdf_data(raw_json: dict) -> dict:
    # Bước 1: Loại bỏ nhiễu "HEADER_PAGE_X" và "FOOTER_PAGE_X" cứng đầu bằng Regex 
    raw_text = raw_json.get("extractedText", "")
    cleaned_content = re.sub(r'(HEADER_PAGE_\d+|FOOTER_PAGE_\d+)', '', raw_text).strip()
    
    # Bước 2: Ép kiểu và map dữ liệu 
    return {
        "document_id": raw_json.get("docId", ""),
        "source_type": "PDF",
        "author": raw_json.get("author", "Unknown"),
        "category": raw_json.get("docCategory", ""),
        "content": cleaned_content,
        "timestamp": raw_json.get("createdAt", "")
    }

def process_video_data(raw_json: dict) -> dict:
    return {
        "document_id": raw_json.get("video_id", ""),
        "source_type": "Video",
        "author": raw_json.get("creator_name", "Unknown"),
        "category": raw_json.get("category", ""),
        "content": raw_json.get("transcript", ""),
        "timestamp": raw_json.get("published_timestamp", "")
    }
```

### 5.3. `quality_check.py` (Vai trò Observability Engineer)
**Mục đích:** Đóng vai trò là cái Phễu lọc rác (Quality Gates). AI Agent mà cắn phải rác là model sẽ tính toán sai. Do đó, hàm này đánh rớt toàn bộ bản ghi rác.

**Code:**
```python
def run_semantic_checks(doc_dict: dict) -> bool:
    content = doc_dict.get("content", "")
    
    # 1. Nếu văn bản trống hoặc ít hơn 10 ký tự -> Không mang ý nghĩa ngữ nghĩa -> Chặn
    if not content or len(content) < 10:
        return False
    
    # 2. Quét mảng các mã báo lỗi toxic (phòng trường hợp OCR chết, lưu traceback vào DB)
    toxic_keywords = ["Null pointer exception", "OCR Error", "Traceback"]
    for keyword in toxic_keywords:
        if keyword in content:
            return False
            
    return True
```

### 5.4. `orchestrator.py` (Vai trò DevOps)
**Mục đích:** Sắp xếp toàn bộ pipeline từ `Raw Data -> ETL Parse -> Quality Check -> DB`. 

**Code hoàn thành (Trích đoạn luồng xử lý chính):**
```python
    ...
    # Xử lý PDF
    for file_path in pdf_files:
        with open(file_path, 'r', encoding='utf-8') as f:
            raw_data = json.load(f)
        processed_data = process_pdf_data(raw_data) # B1: Parse and Clean
        if run_semantic_checks(processed_data):     # B2: Quality Check passes
            final_kb.append(processed_data)         # B3: Load into target 
    ...
```
