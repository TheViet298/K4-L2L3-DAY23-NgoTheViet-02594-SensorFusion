# Báo cáo bài nộp — Day 23 Sensor Fusion Lab

> Điền file này rồi commit. Cách nộp: [hướng dẫn nộp](../SUBMISSION.md).

## Thông tin học viên

- Họ tên: Ngo The Viet
- MSSV: 02594
- Email:
- Link repo (fork): https://github.com/TheViet298/K4-L2L3-DAY23-NgoTheViet-02594-SensorFusion
- Commit hash nộp (`git rev-parse HEAD`): xem `git rev-parse HEAD`

## Tóm tắt kết quả

- `fusion_mode`: `compare`; `frames`: `[0, 198]`; `segment`: `training_segment-1005081002024129653_5313_150_5333_150_with_camera_labels.tfrecord`; `seed`: `0`.
- `detection.precision`: `0.9700934579`; `detection.recall`: `0.7004048583`; `tp/fp/fn`: `519/16/222`.
- LiDAR: `rmse=0.1503227004`, `matches=502`, `sum_sq_err=11.3436509572`, `ghost_track_frames=0`, `missed_gt_frames=239`, `mean_confirmed_tracks=2.5226130653`.
- Fused: `rmse=0.1358667803`, `matches=502`, `sum_sq_err=9.2668105575`, `ghost_track_frames=0`, `missed_gt_frames=239`, `mean_confirmed_tracks=2.5226130653`.
- Hai mode có cùng 502 cặp ghép và cùng 0 ghost; fused giảm RMSE từ 0.1503 m xuống 0.1359 m, trong khi số missed GT vẫn là 239. Vì vậy camera update trong lần chạy này cải thiện ước lượng trạng thái, không làm tăng ghost/miss.

Chạy từ root repo:

```bash
fusion-run-lab --config student/config/paths.yaml --fusion compare --seed 0
```

`rmse = sqrt(sum_sq_err/matches)` trên vị trí 3D của confirmed tracks ghép
một-một với GT xe trong cửa sổ BEV, gate XY **2.0 m**; `null` nếu không có cặp.
Camera dùng tâm hộp 2D ground-truth FRONT có nhiễu seeded, **không** dùng camera
detector. Kết quả này không đo hiệu quả một perception system độc lập với GT.

`grade_run.log` là JSONL, mỗi `(mode,frame)` đúng một record với các trường:
`mode`, `frame`, `det_tp`, `det_fp`, `det_fn`, `valid_gt`, `confirmed`, `matches`,
`sum_sq_err`, `ghosts`, `misses`. Đảm bảo `matches+ghosts==confirmed` và
`matches+misses==valid_gt`; tổng/trung bình record phải khớp `metrics.json`.
File per-mode `metrics_lidar.json`, `metrics_fused.json`, `grade_run_lidar.log`,
`grade_run_fused.log` được giữ để đối chiếu.

## Giải thích ngắn (Parts E–H — tự viết)

1. LiDAR đo vị trí 3D trong vehicle frame nên `z` có 3 phần tử và `R` là covariance 3x3 theo mét. Camera đo pixel 2D sau phép chiếu pinhole nên `z` có 2 phần tử, `R` là covariance 2x2 theo pixel và EKF dùng Jacobian của phép chiếu.
2. Gating Mahalanobis chuẩn hóa residual theo covariance dự đoán, giúp loại các detection không phù hợp trước khi gán; ngưỡng Chi-square còn phụ thuộc số chiều measurement.
3. Đây là track-then-fuse: log có một tracker, predict một lần, lidar association/update trước rồi camera association/update trên cùng track trong mode fused.
4. Calibration lệch làm innovation/residual camera có bias lớn hoặc tăng phương sai; nhiều cặp có thể bị chi-square gate loại và trạng thái camera update bị kéo sai.
5. Frame rỗng vẫn cần truyền `sensor` để biết đây là pass LiDAR hay camera. Chỉ LiDAR được cộng/trừ score, sinh track và xóa track; camera chỉ gọi EKF update nên không ảnh hưởng lifecycle.
6. Track bắt đầu ở trạng thái initialized với score `1/window`; LiDAR hit tăng `1/window` tối đa 1 và vượt `confirmed_threshold` thì confirmed. Miss trong FOV giảm score nhưng track confirmed vẫn giữ trạng thái. Track bị xóa khi phương sai x/y vượt `max_P`, hoặc confirmed có score dưới `delete_threshold`, hoặc track chưa confirmed có score `<= 0`.

## Bonus (không bắt buộc)

Liệt kê phần bonus đã làm, file bằng chứng trong `student/bonus/` và kết quả chính
(xem [RUBRIC.md](../RUBRIC.md) mục 2). Không làm thì ghi "Không".

- Không.

## Khai báo sử dụng AI (bắt buộc)

Ghi rõ, kể cả khi không dùng ("Không dùng AI"). Xem [RULES.md](../RULES.md) mục 2.

- Công cụ đã dùng (ChatGPT, Copilot, Claude, …): ChatGPT.
- Dùng cho phần nào: đọc repo, hỗ trợ implement các hàm Part E–H và debug môi trường chạy test/pipeline.
- Cách đã kiểm tra lại: chạy `pytest student/tests -q` đạt 128 passed; chạy `fusion-run-lab --fusion compare --seed 0` đủ frame 0–198; đối chiếu công thức EKF, gating, camera projection và lifecycle với test/code platform.

## Checklist nộp

- [x] **Part E–H** trong `workspace/` đã implement; `pytest student/tests -q` đạt 128 passed
- [x] Part A–D không sửa
- [x] Lần chạy chấm điểm: `--fusion compare --seed 0`, `frame_start: 0`, `frame_end: 198`
- [x] Đã sinh `student/artifacts/metrics*.json` và `student/artifacts/grade_run*.log` từ runner
- [x] Đã điền báo cáo và khai báo AI
- [x] Không commit dữ liệu Waymo, weights, `paths.yaml`, API key
- [ ] `python tools/check_submission.py` báo `KẾT QUẢ: SẴN SÀNG NỘP` sau khi commit
- [ ] Đã push và nộp link repo + commit hash trên LMS ([hướng dẫn nộp](../SUBMISSION.md))
