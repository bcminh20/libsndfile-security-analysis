# Static and Dynamic Security Analysis of Legacy C/C++ Software: Evaluating libsndfile

Dự án nghiên cứu và thực nghiệm phân tích bảo mật phần mềm (kết hợp phân tích tĩnh và phân tích động) nhằm phát hiện các lỗ hổng an toàn bộ nhớ[cite: 1, 3]. Dự án tập trung đánh giá hiệu quả và so sánh ưu/nhược điểm của các công cụ kiểm thử tự động[cite: 1] trên thư viện xử lý âm thanh mã nguồn mở **`libsndfile` (phiên bản 1.0.28)**[cite: 1, 4, 6].

---

## 1. Công cụ & Phương pháp tiếp cận

Dự án kết hợp 3 nhóm công cụ cốt lõi trên môi trường Linux[cite: 1, 5]:
* **Dynamic Analysis (Fuzzing):** Sử dụng **AFL++ (`afl-fuzz`, `afl-clang-fast`)** để đột biến dữ liệu đầu vào (seed corpus) và khám phá các nhánh thực thi gây lỗi[cite: 1, 3, 5].
* **Runtime Instrumentation:** Tích hợp **AddressSanitizer (ASan)** vào quá trình biên dịch C/C++ nhằm phát hiện chính xác các lỗi bộ nhớ không gian và thời gian (Spatial & Temporal memory flaws) như Heap/Stack Buffer Overflow, Use-After-Free.
* **Static Analysis (Bounded Model Checking):** Sử dụng **ESBMC** để kiểm tra mô hình mã nguồn tĩnh, dò tìm các nguy cơ tràn số nguyên (Arithmetic Overflow), chia cho 0 (Division by Zero) và lỗi truy cập con trỏ.

---

## 2. Phân công nhiệm vụ (Nhóm 4 thành viên)

| Thành viên | Vai trò chính | Nhiệm vụ cụ thể |
| :--- | :--- | :--- |
| **Thành viên 1** | Quản lý Target & Seed Corpus | Xử lý mã nguồn `libsndfile` v1.0.28[cite: 5, 6], thu thập và tối ưu hóa tập file âm thanh mẫu (`seeds/`)[cite: 1, 6], đối chiếu kết quả lỗi với cơ sở dữ liệu CVE[cite: 1]. |
| **Thành viên 2** | Chuyên trách Fuzzing (AFL++) | Thiết lập môi trường AFL++[cite: 3, 5], viết kịch bản tự động hóa chiến dịch fuzzing (`scripts/run_afl.sh`), giám sát độ phủ code và trích xuất các mẫu gây sập chương trình (`crashes`)[cite: 1]. |
| **Thành viên 3** | Chuyên trách Instrumentation (ASan) | Cấu hình cờ biên dịch tĩnh tích hợp ASan (`scripts/build_asan.sh`)[cite: 3, 5, 6], tinh chỉnh `ASAN_OPTIONS` và giải mã Stack Trace từ các file crash để tìm nguyên nhân gốc rễ[cite: 1]. |
| **Thành viên 4** | Chuyên trách Phân tích tĩnh (ESBMC) | Thiết lập ESBMC[cite: 4, 5], khoanh vùng bề mặt tấn công (Format Parsers), viết kịch bản quét tĩnh tự động (`scripts/run_esbmc.sh`) và tổng hợp bảng so sánh hiệu quả giữa các công cụ[cite: 1]. |

---

## 3. Cấu trúc thư mục dự án

```text
libsndfile-security-analysis/
├── README.md                 # Tài liệu giới thiệu và hướng dẫn sử dụng dự án
├── .gitignore                # Cấu hình bỏ qua các file biên dịch tạm và rác từ fuzzer
├── docs/                     # Tài liệu học thuật, báo cáo Checkpoint và Slide cuối kỳ
├── target/                   # Chứa kịch bản tải và mã nguồn của phần mềm mục tiêu
│   ├── get_libsndfile.sh     # Script tự động tải và giải nén libsndfile v1.0.28
│   └── libsndfile-1.0.28/    # Thư mục mã nguồn sau khi giải nén (được ẩn khỏi Git)
├── seeds/                    # Tập dữ liệu âm thanh mồi (WAV, FLAC, AIFF...) kích thước nhỏ
├── scripts/                  # Các kịch bản tự động hóa quá trình biên dịch và kiểm thử
│   ├── build_asan.sh         # Biên dịch tĩnh libsndfile kết hợp AFL++ và ASan
│   ├── run_afl.sh            # Khởi chạy chiến dịch fuzzing trên tiện ích sndfile-convert
│   └── run_esbmc.sh          # Chạy phân tích tĩnh ESBMC trên các module xử lý header
└── findings/                 # Lưu trữ kết quả đầu ra và bằng chứng lỗ hổng
    ├── afl_crashes/          # Các file âm thanh dị dạng gây crash thu được từ AFL++
    ├── asan_logs/            # Báo cáo Stack Trace chi tiết do ASan xuất ra
    └── esbmc_reports/        # Log phân tích tĩnh và bảng tổng hợp từ ESBMC