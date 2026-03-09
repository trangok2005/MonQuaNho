### 3.4.2 Database Choice



**ADR-06: Relational Database for Persistent Data Storage**

  * **Status:** `Accepted`

-----

#### 1\. Context (Bối cảnh)

Hệ thống đặt lịch khám cần lưu trữ nhiều loại dữ liệu cốt lõi có **mối quan hệ chặt chẽ** với nhau, bao gồm:

  * User accounts (Tài khoản người dùng)
  * Patient profiles (Hồ sơ bệnh nhân)
  * Doctor information & schedules (Thông tin và lịch làm việc của bác sĩ)
  * Appointments (Lịch khám bệnh)
  * Medical records (Hồ sơ bệnh án)

Các thực thể dữ liệu này có tính liên kết cao. Ví dụ:

  * Một bệnh nhân (Patient) có thể có nhiều lịch khám (Appointment).
  * Một bác sĩ (Doctor) sẽ tiếp nhận nhiều lịch khám (Appointment).
  * Mỗi lịch khám (Appointment) lại được liên kết trực tiếp với một hồ sơ bệnh án (Medical Record).

Do đó, hệ thống bắt buộc phải sử dụng một giải pháp cơ sở dữ liệu có khả năng:

1.  Quản lý tốt các mối quan hệ (relational structure) giữa các luồng dữ liệu.
2.  Đảm bảo tính nhất quán của dữ liệu (data consistency).
3.  Hỗ trợ giao dịch (transactional support) chặt chẽ để tránh xung đột khi đặt lịch.

-----

#### 2\. Decision (Quyết định)

Chúng tôi quyết định lựa chọn **Relational Database Management System (RDBMS)** cho hệ thống lưu trữ bền vững (persistent storage). Cụ thể, hệ thống sẽ sử dụng **MySQL** làm cơ sở dữ liệu chính.

Phía Backend được xây dựng bằng **Django**, sử dụng **Django ORM** để tương tác trực tiếp với MySQL. Việc sử dụng Django ORM cho phép ánh xạ (mapping) tự động các object/model trong code sang các bảng (tables) trong database.

**Bảng ánh xạ Model dự kiến:**

| Django Model | Database Table |
| :--- | :--- |
| `User` | `users` |
| `Patient` | `patients` |
| `Doctor` | `doctors` |
| `Appointment` | `appointments` |
| `MedicalRecord` | `medical_records` |

-----

#### 3\. Alternatives Considered (Các lựa chọn đã cân nhắc)

Trước khi đưa ra quyết định, chúng tôi đã đánh giá 3 tùy chọn cơ sở dữ liệu phổ biến nhất dựa trên đặc thù của dự án:

| Tùy chọn Database | Phân loại | Ưu điểm | Nhược điểm |
| :--- | :--- | :--- | :--- |
| **MySQL / PostgreSQL** | Relational Database | - Hỗ trợ cấu trúc dữ liệu quan hệ tốt.<br>- Đảm bảo ACID transactions.<br>- Tích hợp hoàn hảo với Django ORM. | - Schema cứng nhắc, đòi hỏi migration khi có thay đổi cấu trúc so với NoSQL. |
| **MongoDB** | Document (NoSQL) Database | - Schema cực kỳ linh hoạt.<br>- Phù hợp với các định dạng dữ liệu phi cấu trúc. | - Khó quản lý các mối quan hệ chéo phức tạp.<br>- Khả năng hỗ trợ transaction hạn chế hơn RDBMS. |
| **Redis** | In-memory Database | - Tốc độ đọc/ghi dữ liệu cực kỳ nhanh. | - Không phù hợp cho lưu trữ dữ liệu vĩnh viễn (persistent storage).<br>- Thường chỉ dùng làm caching hoặc message broker. |

-----

#### 4\. Rationale (Lý do lựa chọn MySQL)

Quyết định chọn MySQL được đưa ra dựa trên 4 lý do cốt lõi sau:

1.  **Phù hợp tuyệt đối với dữ liệu quan hệ:** Hệ thống mạng lưới dữ liệu dày đặc (Patient → Appointment, Doctor → Appointment, Appointment → MedicalRecord). Database quan hệ giúp quản lý các Foreign Keys và cấu trúc này một cách tối ưu nhất.
2.  **Hỗ trợ Transaction mạnh mẽ:** Các thao tác quan trọng (như khi user bấm đặt lịch khám) đòi hỏi tính nhất quán dữ liệu tuyệt đối (Data Consistency). MySQL hỗ trợ **ACID transactions**, giúp ngăn chặn tình trạng sai lệch hoặc mất mát dữ liệu khi có nhiều truy vấn xảy ra đồng thời.
3.  **Tương thích hoàn hảo với Backend Framework:** Django ORM được thiết kế để làm việc cực kỳ trơn tru với MySQL. Nó cho phép định nghĩa các Models bằng Python, tự động generate database schema, và thực hiện các câu lệnh query phức tạp dễ dàng mà không cần viết SQL raw.
4.  **Độ tin cậy và Cộng đồng:** MySQL là hệ quản trị CSDL phổ biến, tài liệu phong phú, dễ dàng triển khai (deploy), đồng thời có cộng đồng hỗ trợ khổng lồ giúp giảm thiểu rủi ro (development complexity) trong quá trình phát triển.

-----

#### 5\. Consequences (Hệ quả)

**Tích cực (Ưu điểm):**

  * Đảm bảo tính toàn vẹn và nhất quán của dữ liệu đặt lịch khám.
  * Quản lý các luồng dữ liệu có tính quan hệ (relational data) hiệu quả, dễ dàng query chéo.
  * Tăng tốc độ phát triển nhờ sự tích hợp sâu giữa MySQL và Django ORM.
  * Dễ bảo trì và mở rộng đội ngũ phát triển vì công nghệ phổ biến.

**Tiêu cực (Nhược điểm & Rủi ro):**

  * Việc thay đổi cấu trúc bảng (schema changes) cần phải cẩn trọng hơn và luôn đi kèm với quá trình database migration.
  * Nếu hệ thống phát triển quá lớn, việc scale ngang (horizontal scaling) của MySQL sẽ phức tạp hơn so với các giải pháp NoSQL.

<!-- end list -->