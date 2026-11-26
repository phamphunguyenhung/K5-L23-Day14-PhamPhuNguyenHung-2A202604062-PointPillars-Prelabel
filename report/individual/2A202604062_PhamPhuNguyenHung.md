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
| Gốc | 35 | 10 | 9 | 25.00 |
| `best_config.json` | 7 | 12 | 8 | 18.10 |

- Tham số đã đổi trong `best_config.json`: `score_thr: 0.1 -> 0.5` (giữ `intensity_mode=clip`, `ground_mode=fixed`, `min_points=0`).
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra):
  - `s01f23: cấu hình chọn {'intensity_mode': 'clip', 'ground_mode': 'fixed', 'score_thr': '0.5', 'min_points': '0'} -> 11.2 (tốt nhất có thể 11.2)`
  - `s02f17: cấu hình chọn {'intensity_mode': 'clip', 'ground_mode': 'fixed', 'score_thr': '0.5', 'min_points': '0'} -> 5.1 (tốt nhất có thể 4.8)`
  - `s04f46: cấu hình chọn {'intensity_mode': 'clip', 'ground_mode': 'fixed', 'score_thr': '0.5', 'min_points': '0'} -> 1.8 (tốt nhất có thể 1.8)`
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: Cả 3 fold leave-one-out đều chọn `clip, fixed, score_thr=0.5` và đạt đúng/sát mức oracle (`11.2`, `5.1`, `1.8`). Tuy nhiên ngưỡng `0.5` khá cao nên có rủi ro làm giảm recall khi sang tập B (xem Mục 4).

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202604062/compare/s02f17.jpg)

- `scripts/pp_compare.py` báo giảm 28% công sửa trên tập A (`25.0 -> 18.1`).
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: Dự đoán ở mục 1 **đúng phần nâng ngưỡng lọc** (nâng `score_thr=0.5` cắt mạnh `Car` ảo từ `35` xóa xuống `7` xóa, `sửa` giảm từ `9` xuống `8`, dù `thêm` tăng nhẹ từ `10` lên `12`, tổng chi phí giảm từ `25.0` xuống `18.1`). Tuy nhiên dự đoán dùng `intensity_mode=log` **chưa đúng** vì `log` làm chi phí tăng lên `20.7–28.4` do mất recall nhiều hơn.

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202604062/frames_B.txt --config d14_2A202604062/best_config.json --gt d14_2A202604062/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| 32 | 62 | 6 | 74.60 |

- Loại box tốn công sửa nhất ở tập B: **Box bị sót (`62` box cần thêm, `1.0 × 62 = 62.0`)**, do ngưỡng `score_thr=0.5` chọn từ 3 frame tập A quá cao khi áp sang 12 frame tập B (làm rớt nhiều xe xa/khuất và xe 2 bánh có điểm tin cậy `0.3–0.49`, cộng thêm `Bus`/`Truck` mà KITTI không có). Với ngưỡng `score_thr=0.4` chi phí tập B là `38.80` (`51` xóa, `20` thêm, `7` sửa).
- Nhận xét thực tế về chi phí cao: Khi kiểm tra trực quan trên CVAT vẫn thấy pre-label còn nhiều lỗi xóa/sót/lệch thực tế cần người gán nhãn can thiệp thủ công. Ngay cả khi hạ `score_thr=0.05`, model vẫn bỏ sót `18` box thật (trần tối thiểu `18.0 > 15.0`), nên không thể ép chi phí xuống dưới `15` bằng tinh chỉnh tham số mà không làm sai lệch quy chuẩn kiểm tra nhãn thật (golden truth QC) trên CVAT.

## 5. Giới hạn

- **Tập tune rất nhỏ (chỉ 3 frame):** Kết quả sweep dễ bị overfit theo đặc thù 3 cảnh tập A (`score_thr=0.5` tối ưu cho tập A `18.10` nhưng làm giảm recall mạnh khi sang 12 frame tập B, đẩy chi phí lên `74.60` so với `38.80` ở `score_thr=0.4`).
- **Domain gap & giới hạn 3 lớp của KITTI:** Model PointPillars chỉ được huấn luyện trên KITTI (3 lớp `Car`, `Pedestrian`, `Cyclist`), nên bỏ sót hoàn toàn `Bus`/`Truck` lớn (như xe buýt ở `s01f23`, `s02f11`), hay tạo box `Car` ảo trên bụi cây/hàng rào, và vẫn còn `18` box thật trên tập B không có raw proposal ngay cả ở `score_thr=0.05`.
- **Chi phí công thức chưa phản ánh hết thời gian sửa thật:** Công thức `0.3·xóa + 1.0·thêm + 0.5·sửa` tính bằng số lượng box, chưa đo thời gian thao tác thực tế khi căn 3D cuboid (`L/W/H`, `z`, `yaw`) trên CVAT. Vì giữ nguyên chuẩn QC trung thực trên CVAT thay vì chỉnh ground truth cho khớp model, chi phí sửa thực tế phản ánh đúng khối lượng can thiệp thủ công cần thiết.
