---
status: ACTIVE_SSOT
ttl: 30d
owner: antigravity
created: 2026-11-26
expires: 2026-12-26
---

# Day 14 — Checkpoint 0 & Checkpoint 1 Status (SSOT)

- **Học viên:** Phạm Phú Nguyễn Hưng
- **MSSV:** `2A202604062`
- **GitHub:** `phamphunguyenhung`
- **Repository:** `D:\06-Learning\AIA-VINGROUP\learning\day14\labs\K5-L23-Day14-PointPillars-Prelabel` (`main` @ `d86c24f`)

## Bảng trạng thái

| Hạng mục | Trạng thái | Bằng chứng |
|---|---|---|
| Repository | PASS | Branch `main`, commit `d86c24f` (`origin`: `https://github.com/VinUni-AI20k/K5-L23-Day14-PointPillars-Prelabel.git`) |
| Python environment | PASS | `.venv` Python `3.14.6` |
| Dependencies | PASS | `torch==2.15.0+cpu`, `numpy==2.5.5`, `Pillow==13.0.0`, `cvat-sdk==2.84.0` (import test OK) |
| PointPillars weights | PASS | `third_party/PointPillars/pretrained/epoch_160.pth` (19.4 MB), upstream commit `620e6b0d07e4cb37b7b0114f26b934e8be92a0ba` |
| Data-VF | PASS | `D:\06-Learning\AIA-VINGROUP\learning\day13\labs\K5-L23-Day13-PointCloud-Cuboid-Lab\data\Data-VF` (`sequence_01`–`sequence_05`, mỗi sequence 60 frames) |
| Selftest CP0 | PASS | `python scripts/pp_selftest.py --data "$DataVF"` -> `TẤT CẢ OK` (`s05f40 35 box, 4.4s`) |
| Coordinate system | PASS | Đã kiểm chứng `vf_pp/core.py`: `prepare_points` (`x'=-y, y'=x, z'=z+dz`), `raw_detect` hoàn nguyên (`x=cy, y=-cx, z=zb+h/2-dz`), yaw/lwh, 4 passes `_rot(..., ±a)` (`COORD_ROUNDTRIP_OK`) |
| Official SEED | MISSING | Chưa có trong prompt và biến môi trường `D14_SEED` trống |
| Frames A | BLOCKED | Chờ SEED chính thức từ coach |
| Frames B | BLOCKED | Chờ SEED chính thức từ coach |

## Bước tiếp theo khi có SEED

```powershell
$Seed = "<SEED_CHINH_THUC>"
$DataVF = "D:\06-Learning\AIA-VINGROUP\learning\day13\labs\K5-L23-Day13-PointCloud-Cuboid-Lab\data\Data-VF"
.\.venv\Scripts\python.exe scripts/vf_cvat.py day14 --student "2A202604062" --seed "$Seed" --data "$DataVF"
```
