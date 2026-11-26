---
status: ACTIVE_SSOT
ttl: 14d
owner: antigravity
created: 2026-11-26
expires: 2026-12-10
---

# Báo cáo kiểm tra trực quan `out/lec_filtered_png/` (6 frame buổi giảng)

- **Cấu hình chạy:** `score_thr=0.4`, `min_points=5` (so với mặc định `score_thr=0.1, min_points=0`).
- **Nguồn ảnh kiểm tra:** `out/lec_filtered_png/{s01f39,s02f07,s03f15,s03f18,s05f09,s05f40}.jpg` (7 góc camera + BEV 50 m).
- **Tổng số cuboid sau lọc:** 128 box trên 6 frame (20–24 box/frame).

## 1. Tổng hợp các lỗi hệ thống (Pattern chung trên 6 frame)

1. **Box ảo (False Positive) do điểm LiDAR của cây, bụi rậm, hàng rào, vỉa hè:**
   - Rất nhiều box `Car` (đỏ) và `Cyclist` (xanh lá) bị đặt lên dải cây phân cách, hàng rào tôn công trường, bồn cây và cột đèn (`s01f39`, `s02f07`, `s03f15`, `s05f09`, `s05f40`).
   - Lý do: PointPillars pretrained trên KITTI chỉ dùng LiDAR 3D (không dùng camera), các cụm điểm dày từ bụi cây/tường/hàng rào có mật độ >= 5 điểm và độ tự tin >= 0.4 vẫn vượt qua bộ lọc.
2. **Box ảo tại tâm / sát xe ego:**
   - Xuất hiện box `Car` ngay gần tâm `(0, 0)` hoặc cắt vào thân xe ego (`s02f07`, `s03f18`, `s05f09`), tạo khung dây đỏ khổng lồ vắt qua nhiều camera.
3. **Sót hoàn toàn hoặc nhận sai lớp xe lớn (`Bus`, `Truck`, `Van`):**
   - KITTI không có lớp `Bus` / `Truck` riêng (chỉ có `Car` rồi suy luận theo luật kích thước trong `vf_pp/core.py`). Kết quả là xe buýt/xe tải van sát cạnh hoặc phía trước bị **sót hoàn toàn** (`s02f07`: 2 xe buýt lớn; `s03f15`: xe van `货拉拉APP`; `s05f09`: xe tải thùng xanh lá; `s05f40`: xe buýt vàng/nâu) hoặc bị cắt thành 2–3 box `Car` nhỏ chồng chéo (`s05f40`, `s03f15`).
4. **Sót người đi bộ (`Pedestrian`) và `Cyclist` thực tế:**
   - Sau khi nâng `score_thr=0.4`, nhiều `Pedestrian` và `Cyclist` bị lọc mất (`s01f39`, `s02f07`, `s03f15`, `s03f18`, `s05f09`), trong khi các box `Cyclist` ảo trên cây/cột lại còn giữ nguyên.

---

## 2. Chi tiết từng frame

### `s01f39.jpg` (20 box — Car: 8, Cyclist: 12)
- **Tổng quan:** Đường đô thị ban ngày, có sedan trắng chạy sát bên trái ego, xe con/xe lớn phía trước, sedan xanh phía sau và xe hai bánh dọc làn phụ.
- **Box ảo:** Box `Car` (đỏ) trên dải cây/lề đường (`cam08` phải, `cam09` trái, rìa `cam05`/`cam06`). Nhiều box `Cyclist` (xanh lá) đặt lên thân cây (`cam01` phải), bụi cây (`cam09` trái) và chồng lên mũi/cửa chiếc sedan trắng (`cam05`, `cam06`).
- **Box lệch / sai kích thước:** Cuboid `Car` của sedan trắng sát bên trái (`cam05`, `cam06`, `cam09`) bị **cao quá nóc xe** rất nhiều; các box quanh thân xe trắng chồng lấn nhau.
- **Vật thể thật bị sót:** Người đi xe đạp áo đỏ (`cam01` phải / `cam07` trái), xe buýt/tải đỏ & xe vàng phía trước (`cam03`), một số xe ở xa trên `cam01`.

### `s02f07.jpg` (24 box — Car: 20, Cyclist: 4)
- **Tổng quan:** Giao lộ rộng, có sedan đen ngay trước ego, 2 xe buýt lớn bên trái, một số ô tô và người đi bộ/xe đạp qua đường.
- **Box ảo:**
  - **Box `Car` ngay tâm `(0, 0)`** trùng xe ego -> tạo khung dây đỏ khổng lồ bao `cam01`, chắn giữa `cam09`, cắt vào `cam05`/`cam07`/`cam08`.
  - Nhiều box `Car` và 1 box `Cyclist` nằm trên hàng rào/bụi cây/vỉa hè trống (`cam05` giữa, `cam08` nửa phải, `cam09` hai mép).
- **Box lệch / sai kích thước:** Chồng khít `Cyclist` + `Car` tại xe van/SUV trắng ở xa (`cam03`); box `Cyclist` ở giữa `cam07` quá rộng/cao so với người đi xe đạp thật.
- **Vật thể thật bị sót:** **2 xe buýt trắng-xanh lớn** bên trái (`cam01`, `cam03`, `cam06`) và 1 xe buýt/tải lớn tối màu phía sau (`cam09`) **bị sót hoàn toàn (0 box Bus)**; sót người đi bộ (`cam06`, `cam03`, `cam05`) và 1 xe máy (`cam07`).

### `s03f15.jpg` (20 box — Car: 17, Cyclist: 3)
- **Tổng quan:** Đường đô thị đông xe, hai bên có hàng rào tôn công trường và bồn cây; cạnh ego có xe tải van lớn (`货拉拉APP`), phía trước có xe tải nhỏ thùng hở.
- **Box ảo:** Chuỗi 4–6 box `Car` (đỏ) nối tiếp nằm hoàn toàn trên bồn cây, tủ gỗ và hàng rào tôn bên phải (`cam07`, `cam08`, rìa trái `cam09`).
- **Box lệch / sai lớp:** Xe tải nhỏ thùng hở (`cam01`, `cam03`, `cam06`) bị gán `Car` thay vì `Truck` và dư chiều cao/chưa khít đầu xe; cụm `Cyclist` góc ngã tư phải chồng chéo; các box phía sau (`cam05`, `cam09`) hơi lệch/chồng lấn.
- **Vật thể thật bị sót:** **Xe tải van `货拉拉APP`** sát bên cạnh (`cam05`, `cam06`, mép `cam09`) không có cuboid bao trọn thân xe; xe van/MPV bạc chạy ngang ở ngã tư xa (`cam03`); người đi bộ góc ngã tư phải (`cam03`).

### `s03f18.jpg` (21 box — Car: 15, Cyclist: 3, Pedestrian: 3)
- **Tổng quan:** Ngã tư rộng có hàng rào công trường bên phải/sau, 1 sedan tối màu ngay trước ego, hàng ô tô nối đuôi phía sau và 3–4 xe 2 bánh sát lề phải.
- **Box ảo:**
  - **Box `Car` ngay tâm `(0, 0)`** trên BEV -> tạo khung đỏ khổng lồ vắt qua mặt đường trống/bầu trời trên `cam01`, `cam05`, `cam06`, `cam07`, `cam08`, `cam09`.
  - Cụm box `Pedestrian` góc ngã tư (`cam01`, `cam07`) xếp chồng chéo và lấn ra khoảng trống/cột biển báo.
- **Box lệch / sai kích thước:** SUV đỏ phía sau (`cam05`, `cam09`) có cạnh dưới phình rộng trước bánh xe; các box `Cyclist` (`cam07`, `cam08`) hơi cao/rộng và lệch mép giữa 2 xe; vài xe ở xa (`cam03`, `cam06`) trượt nhẹ.
- **Vật thể thật bị sót:** 2 người đi bộ gần góc vỉa hè phải (`cam03`), 1 người đi xe 2 bánh kẹp giữa ở cụm trái `cam08`/`cam07`, 1 người đi xe 2 bánh mép phải `cam01`.

### `s05f09.jpg` (23 box — Car: 22, Cyclist: 1)
- **Tổng quan:** Ngã tư rộng ~15–20 xe; chủ yếu `Car` và 1 `Cyclist` (ID `22`).
- **Box ảo:** Box `Car` nhỏ đè lên thân xe ego (`cam05` mép trái); box `Cyclist` ID `22` (`cam05`) nằm ở bờ cây/cọc ven đường không thấy người đi xe đạp; box `Car` nhỏ dưới trụ đèn/vỉa hè (`cam07` phải, `cam08` trái-giữa); box `Car` nhỏ lồng bên trong thân sedan đen (`cam05`/`cam06`).
- **Box lệch / sai kích thước:** Xe MPV trắng (`cam08`, `cam09`) bị box quá khổ/cao và lệch trái-trước; SUV đen (`cam01`, `cam03`) dư chiều dài ra phía sau; sedan trắng (`cam03` trái) hơi lệch về đuôi.
- **Vật thể thật bị sót:** **Xe tải thùng xanh lá lớn** ngay phía sau (`cam09` giữa-trái, `cam08` mép phải) bị sót hoàn toàn; xe MPV trắng/bạc phía trước (`cam03` giữa-phải); xe vàng và xe trắng làn trước (`cam01`, `cam03`); người đi bộ trên vỉa hè (`cam01`, `cam07`).

### `s05f40.jpg` (20 box — Car: 14, Cyclist: 6)
- **Tổng quan:** Đường có nhiều bụi cây rậm hai bên và dải phân cách; gần ego có xe buýt lớn vàng/nâu bên trái, xe con trắng bên phải, xe van xanh dương làn giữa.
- **Box ảo:** Rất nhiều box `Car` (đỏ) và `Cyclist` (xanh lá) đặt nhầm lên bụi cây, tán cây cao, cột và vỉa hè dọc hai bên đường (`cam05`, `cam06`, `cam08`, `cam09`, `cam01`, `cam03`, `cam07`).
- **Box lệch / sai lớp:** Xe buýt lớn bên trái không có box `Bus` (vàng) mà bị gắn 2–3 box `Car` (đỏ) nhỏ chồng chéo ở đầu/cạnh xe; box `Cyclist` trên tán cây (`cam05`, `cam06`) cao quá cỡ; xe con trắng (`cam01`, `cam07`) bị box `Cyclist` ảo chồng lên phía sau/nóc.
- **Vật thể thật bị sót:** **Xe buýt lớn vàng/nâu** bên trái (sót nhãn `Bus`), **xe van xanh dương** ở làn giữa phía trước (`cam01`, `cam03`), và các xe ở xa (`cam01`, `cam03`, `cam09`).

---

## 3. Hàm ý cho bước Tune (CP2 / bước 5b–6)
- Lọc `score_thr=0.4, min_points=5` giảm được một phần nhiễu nhưng **vẫn giữ nhiều box ảo trên cây/hàng rào/vỉa hè và box tâm xe ego**, đồng thời **làm sót thêm `Pedestrian` và `Cyclist` thật**.
- Khi sửa tay trên CVAT và tune ở bước 6: cần đặc biệt chú ý **chi phí thêm box sót (`1.0`) cao gấp >3 lần xóa box thừa (`0.3`)** — không nên đẩy `score_thr` quá cao nếu làm mất người đi bộ/xe 2 bánh thật.
