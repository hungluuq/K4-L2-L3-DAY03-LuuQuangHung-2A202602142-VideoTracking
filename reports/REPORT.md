# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Lưu Quang Hùng - 2A202602142`
Ngày: `2026-09-04`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `45` phút |
| Thời gian gán `clip_01` | `75` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `12` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Mất dấu vật thể : bật tính năng outside ( Phím tắt o ) ở frame bắt đầu mất dấu hoàn toàn và tắt outside khi xe xuất hiện trở lại nhằm đảm bảo ID không bị đứt gãy.`
2. `Xe di chuyển ở khoảng cách xa / kích thước nhỏ dần : Zoom cận cảnh để khoanh sát rìa vỏ xe nhất có thể.`
3. `Chồng chéo xe, dễ nhầm lẫn ID: Theo dõi chặt chẽ quỹ đạo (trajectory) của từng xe bằng cách tua đi tua lại nhiều lần. Đặt keyframe định kỳ tại các đoạn xe di chuyển nhanh hoặc đổi hướng để thuật toán của CVAT giữ bbox luôn ôm khít xe.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Đã tua toàn bộ clip từ đầu đến cuối chỉ tập trung vào một ID duy nhất để đảm bảo không bị lỗi đứt gãy ID, nhảy ID (ID Switch) giữa các xe đi gần nhau.`
- Lượt 2: `Kiểm tra kỹ thời điểm xe bắt đầu đi vào khung hình và thời điểm xe rời khỏi để kích hoạt/hủy trạng thái outside chính xác, tránh việc bbox xuất hiện lơ lửng khi không có xe (bbox treo/thừa).`
- Lượt 3: ` Kiểm tra độ khít của bbox ở các frame nội suy nằm giữa hai keyframe, điều chỉnh lại các góc khoanh nếu bbox bị trôi lệch khỏi xe.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `f24f112d4643761390079c755c80c1f4eb219908b897d60a5172888fda7ea002` |
| Thời điểm khóa | `2026-09-04` |
| Số row / frame / track trước khi mở reference | `619 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.813 | 0.795 | 0.833 | 0.882 | 0.955 | 0.906 | 0.870 | 50 | 4 | 0 |
| Sau rework | 0.813 | 0.795 | 0.833 | 0.882 | 0.955 | 0.906 | 0.870 | 50 | 4 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **CÓ**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo | 79-100 | 6 | Điều chỉnh lại keyframe bên lề biên của track và set thuộc tính outside sớm hơn tại frame xe rời khung hình |
| Bbox trôi | 106 | 6 | Bổ sung thêm keyframe định hướng tại phân đoạn xe chuyển đổi hướng di chuyển nhanh nhằm giữ bbox bám khít |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `...` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ban_vs_gold | 0.813 | 0.795 | 0.833 | 0.882 | 0.955 | 0.906 | 0.870 | 50 | 4 | 0 |
| bytetrack_vs_gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| reid_vs_gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| reid_vs_ban | 0.770 | 0.715 | 0.829 | 0.904 | 0.881 | 0.764 | 0.896 | 81 | 62 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA 0.906 thấp hơn một chút so với IDF1 0.955.Trong trường hợp này, cả hai chỉ số đều đạt mức rất cao (Cổng đạt). MOTA không phạt quá nặng lỗi lệch ID tích lũy dài hạn như IDF1 vì bản chất của MOTA tập trung nhiều vào độ chính xác phát hiện (FP/FN) ở từng frame riêng lẻ, trong khi IDF1 phản ánh tính nhất quán toàn diện của ID trên toàn bộ vòng đời track.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID (0.763 HOTA / 0.900 IDF1) vượt trội rõ rệt so với ByteTrack control (0.709 HOTA / 0.875 IDF1).Sự khác biệt lớn nhất nằm ở AssA (0.820 vs 0.776) nhờ tính năng ReID cung cấp các đặc trưng diện mạo (appearance embeddings) giúp duy trì định danh xe tốt hơn khi gặp occlusion nhẹ hoặc vật thể bị che khuất.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.649 lên 0.711.FN giảm đáng kể từ 54 xuống 26. Điều này chứng tỏ ReID hỗ trợ đắc lực cho Association bước 2, giúp hạn chế việc detector bỏ sót các frame yếu và tái lập vết chuyển động mượt mà hơn.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Một chỗ bạn đúng và ReID sai: Tại các frame đầu (ví dụ frame 16-116), ReID bắt thêm ID 7 (không khớp với gold hay nhãn thực tế) do nhầm lẫn vật thể tĩnh lân cận là xe (FP). Một chỗ ReID đúng và bạn cần xem lại: Ở các frame như 105, 106, model giữ bám đuổi tốt hơn trong khi nhãn của bạn có thể bị trôi hoặc lệch rìa nhẹ do khoanh không khít giữa các keyframe.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Cần bổ sung quy tắc thống nhất mốc đánh dấu outside chặt chẽ ngay tại frame xe bắt đầu bị che lấp hoàn toàn, đồng thời quy định tăng tần suất keyframe tại các phân đoạn xe chuyển hướng nhanh để tránh tình trạng trôi hay lệch bbox.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Bổ sung quy định chi tiết và nhất quán hơn về số frame tối đa được giữ ID khi xe bị che khuất (occlusion) để tránh lỗi tách một xe thành nhiều track (như lỗi đã gặp với các track 4, 5, 7).`
`Định nghĩa rõ ràng hơn về ngưỡng bắt đầu/kết thúc track và cách xử lý Bbox khi xe đi vào/đi ra khỏi rìa ảnh để tránh lỗi detector khoanh chưa khít.`
`Tăng cường thêm một vòng rà soát (review) tập trung riêng vào các đoạn xe đan chéo nhau hoặc bị che khuất tạm thời để hạn chế tối đa lỗi nhảy ID (ID Switch).`
`Kiểm tra kỹ lưỡng hơn việc bật/tắt tính năng "outside" ở các frame đầu và cuối quỹ đạo để tránh việc sinh ra các Bbox thừa lơ lửng khi xe không còn trong hình.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)