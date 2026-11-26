# Báo cáo Day 14 — Phạm Phú Nguyễn Hưng — 2A202604062

- Mã học viên dùng với `--student`: `2A202604062`
- `seed_tag` trong `frames_A.txt`: `82ac3100fd`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- Tham số tôi nghĩ giảm công sửa nhiều nhất: `intensity_mode=log` kết hợp nâng ngưỡng lọc `thr_car` (khoảng `0.30–0.40`, hoặc `score_thr=0.30` + `min_points=5`), đồng thời giữ `thr_ped` / `thr_cyc` thấp (`0.10–0.20`).
- Vì sao (nhìn ảnh BEV tập A: box thừa nhiều hay box sót nhiều?):
  - Quan sát 3 ảnh BEV + 7 camera trong `d14_2A202604062/png_A/` (`s01f23`: 28 box, `s02f17`: 31 box, `s04f46`: 14 box), **box thừa (`Car` ảo)** xuất hiện dày đặc hơn nhiều so với box sót: cả 3 frame đều có `Car` ảo ngay tâm ego `(0, 0)`, `s01f23` có nhiều `Car` ảo trên bụi cây/bãi xe đạp (`cam07`, `cam08`), `s02f17` có `Car` ảo dọc dải phân cách kim loại và vỉa hè (`cam05`–`cam09`), `s04f46` có `Car` ảo trên hàng rào công trình và dải phân cách vàng (`cam03`, `cam06`, `cam09`).
  - Mặc định `intensity_mode="clip"` làm bão hòa điểm LiDAR VF thang `[0, 255]` thành `1.0`, khiến lá cây/hàng rào dễ bị nhận nhầm thành thân xe. Chuyển sang `intensity_mode=log` và tăng nhẹ `thr_car` / `min_points` sẽ cắt bớt các box `Car` ảo này (giảm chi phí `0.3 * xóa`), nhưng tránh tăng `score_thr` chung quá cao (`> 0.45`) để không làm rớt `Cyclist`/`Pedestrian` thật (chi phí `1.0 * thêm` đắt gấp 3.33 lần xóa).
  - Các xe lớn như `Bus` ở `s01f23` (`cam05`, `cam09`) bị model bỏ sót do giới hạn 3 lớp của KITTI, nên phần này bắt buộc phải thêm thủ công trên CVAT.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | | | | |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202604062/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202604062/frames_B.txt --config d14_2A202604062/best_config.json --gt d14_2A202604062/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Nêu ít nhất 2 giới hạn (ví dụ: chỉ 3 frame tune; model KITTI không có Truck/Bus; chi phí là công thức, chưa phải thời gian sửa thật): ...
