# Demo thuật toán Bầy Sói Xám (Grey Wolf Optimizer – GWO)

Repo này minh hoạ **thuật toán bầy sói xám (Grey Wolf Optimizer – GWO)** trên **một bài toán cụ thể**:  
khớp đường cong suy hao công suất trên sợi quang.

---

## 1. Mục tiêu demo

- Cài đặt **thuật toán GWO chuẩn**.
- Áp dụng GWO để ước lượng bộ tham số \((a, b, c)\) trong mô hình:
  
  \[
  P(L) = a \cdot e^{-b L} + c
  \]
  
  trong đó:
  - \(L\): chiều dài sợi quang (km),
  - \(P(L)\): công suất đo được tại vị trí \(L\) (mW),
  - \(a\): công suất đầu vào xấp xỉ \(P_0\),
  - \(b\): hệ số suy hao (liên quan tới \(\alpha\) tính theo dB/km),
  - \(c\): mức nhiễu nền (noise floor).

- Đo độ phù hợp bằng **MSE (Mean Squared Error)** và tìm bộ tham số \((a, b, c)\) sao cho MSE nhỏ nhất.

---

## 2. Các file trong repo

- `demo_gwo_soi_quang.c`  
  Mã nguồn C cài đặt:
  - sinh dữ liệu mô phỏng sợi quang,
  - hàm mục tiêu MSE,
  - thuật toán Grey Wolf Optimizer,
  - in ra tham số ước lượng tốt nhất.



---

## 3. Ý tưởng thuật toán GWO (tóm tắt ngắn)

Thuật toán Bầy Sói Xám mô phỏng hành vi săn mồi của đàn sói với 4 cấp bậc:

- **Sói đầu đàn (alpha – α)**: nghiệm tốt nhất.
- **Sói phụ tá (beta – β)**: nghiệm tốt nhì.
- **Sói cấp ba (delta – δ)**: nghiệm tốt thứ ba.
- **Các sói còn lại (omega – ω)**.

Trong mỗi vòng lặp:

1. Tất cả sói di chuyển dựa trên vị trí của 3 con **α, β, δ**  
   (bao vây con mồi, thu hẹp khoảng cách).
2. Tham số \(a\) giảm dần từ 2 → 0 để chuyển dần từ **khai phá** (nhảy xa, tìm vùng mới) sang **khai thác** (tinh chỉnh quanh nghiệm tốt).
3. Luôn cập nhật lại 3 con \(α, β, δ\) theo kết quả đánh giá mới.

Thuật toán kết thúc sau khi đủ số vòng lặp hoặc hội tụ.

---

## 4. Mô tả bài toán demo trong code

1. **Sinh dữ liệu sợi quang**:
   - Chọn bộ tham số “thực”:
     - `P0_true = 1.0 mW`,
     - `alpha_db_true = 0.2 dB/km`,
     - `noise_floor_true = 0.01 mW`.
   - Tính `beta_true = alpha_db_true * ln(10) / 10`.
   - Tạo `N_POINTS` (mặc định 30) điểm \(L\) đều từ 0 → 50 km.
   - Tính công suất sạch \(P_\text{clean}(L)\) theo mô hình trên.
   - Cộng thêm nhiễu Gaussian nhỏ để mô phỏng số đo thực tế.

2. **Định nghĩa hàm mục tiêu**:
   - Từ một bộ tham số \((a, b, c)\), tính giá trị mô hình
     \(P_\text{mô hình}(L_i)\) và so với dữ liệu đo \(P_\text{đo}(L_i)\).
   - Tính **MSE**:
     \[
     \text{MSE} = \frac{1}{N} \sum_{i=1}^N (P_\text{mô hình}(L_i) - P_\text{đo}(L_i))^2
     \]
   - Thêm **ràng buộc mềm**:
     - nếu \(a \le 0\) hoặc \(b \le 0\) hoặc \(c < 0\) → trả về một giá trị rất lớn (phạt).

3. **Cài đặt GWO**:
   - Khởi tạo `N_WOLVES` con sói, mỗi con là một bộ \((a, b, c)\) ngẫu nhiên trong khoảng:
     - `a ∈ [0.1, 2.0]`
     - `b ∈ [0.001, 0.2]`
     - `c ∈ [0.0, 0.1]`
   - Lặp `MAX_ITER` vòng:
     - Cập nhật tham số \(a\) giảm tuyến tính.
     - Với mỗi con sói, cập nhật vị trí dựa trên vị trí của 3 con tốt nhất (α, β, δ).
     - Cắt nghiệm về trong đoạn [LB, UB] nếu bị vượt.
     - Tính lại MSE, cập nhật lại α, β, δ.
   - Sau cùng, in ra:
     - bộ tham số tốt nhất,
     - MSE tương ứng,
     - giá trị suy hao xấp xỉ theo dB/km (từ `b_hat`).

---

## 5. Cách biên dịch và chạy



Biên dịch:

```bash
gcc -O2 -lm -o demo_gwo_soi_quang demo_gwo_soi_quang.c
