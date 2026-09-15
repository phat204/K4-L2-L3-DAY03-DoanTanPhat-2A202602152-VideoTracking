# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Đoàn Tấn Phát / Solo (2A202602152)`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `20 phút` |
| Thời gian gán `clip_01` | `60 phút` |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `8` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:
1. Các xe bị che khuất/cắt nhau nên dễ mất liên tục `track_id`. Xử lý: Tôi ưu tiên giữ cùng ID khi vật thể vẫn có thể xác định là cùng xe, thay vì tạo ID mới chỉ vì có occlusion.
2. Ở đầu/cuối quãng đời của xe, dễ bị thừa bbox khi xe chưa vào hẳn hoặc đã ra khỏi khung hình. Xử lý: Kiểm tra lại các frame đầu/cuối, dùng thuộc tính `outside` đúng thời điểm thay vì để bbox treo.
3. Bbox bị trôi giữa các keyframe ở những đoạn xe chuyển hướng hoặc thay đổi tốc độ. Xử lý: Soi lại các frame giữa (mid-frame) và thêm keyframe để bbox bám sát vật thể.

## 2. Tự kiểm (Làm solo - Bỏ qua kiểm chéo)

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):
- Lượt 1: Kiểm tra tính liên tục của `track_id`, tập trung vào các track bị che khuất/cắt nhau xem có bị nhảy ID không.
- Lượt 2: Kiểm tra frame bắt đầu/kết thúc của từng track; phát hiện và cắt bỏ các bbox treo trước khi xe xuất hiện hoặc sau khi xe rời khung.
- Lượt 3: Tua các frame giữa để kiểm tra bbox có bị trôi không; các điểm phát hiện trôi gồm frame 83, 96, 107, 109, 136 và 190.

*Ghi chú: Quá trình kiểm chéo (Peer Review) được bỏ qua do làm việc solo. Tập trung vào tự đánh giá (Self-QC) kĩ lưỡng 3 lượt trên.*

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `633f206b2e795c0959ce23c67217a71d5050a34b67ecd4569a201ddba14cbc62` |
| Thời điểm khóa | `2026-09-15 17:05:16` |
| Số row / frame / track trước khi mở reference | `646 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.768 | 0.748 | 0.791 | 0.869 | 0.932 | 0.855 | 0.856 | 78 | 5 | 0 |
| Sau rework | 0.768 | 0.748 | 0.791 | 0.869 | 0.932 | 0.855 | 0.856 | 78 | 5 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **Có (ĐẠT)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox trôi | 83, 96, 107, 109, 136, 190 | 5, 6, 8, 1 | Kiểm tra lại frame và thêm/chỉnh keyframe để bbox bám sát xe, cải thiện IoU. |
| Bbox treo ở đầu track | 50–53, 51–78, 78–100, 102–105, 133–135 | 4, 5, 6, 7, 8 | Xác định lại chính xác frame xe bắt đầu xuất hiện và set `outside` đúng frame trước đó. |
| Bbox treo ở cuối track | 149–151, 169–171 | 4, 8 | Kết thúc track đúng lúc khi xe vừa rời khung hình bằng `outside`. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0 (CUDA)` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.768 | 0.748 | 0.791 | 0.869 | 0.932 | 0.855 | 0.856 | 78 | 5 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.755 | 0.695 | 0.822 | 0.911 | 0.863 | 0.732 | 0.903 | 81 | 89 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**
- MOTA của tôi **thấp hơn** IDF1 (0.855 so với 0.932). 
- Trong trường hợp này, IDF1 rất cao vì bản gán nhãn của tôi duy trì danh tính (identity) track hoàn hảo (IDSW = 0). MOTA bị kéo xuống chủ yếu do có 78 lỗi FP (bắt dư bbox). MOTA tính toán dựa trên tổng các lỗi FP, FN, và IDSW so với số ground-truth. Nó xem IDSW chỉ như một lỗi đơn lẻ tại 1 frame (giống FP/FN) thay vì phạt sự sai lệch danh tính kéo dài trên toàn bộ vòng đời của đối tượng như cách IDF1 làm. Do đó, MOTA không phạt nặng lỗi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể.**
- IDF1 tăng từ 0.875 (ByteTrack) lên 0.900 (ReID). AssA tăng từ 0.776 lên 0.820. Tuy nhiên, IDSW giữ nguyên ở mức 2.
- Việc IDF1 và AssA tăng cho thấy BoT-SORT + ReID duy trì liên kết track (association) và danh tính đối tượng dài hạn tốt hơn control. Tuy nhiên nó không triệt tiêu được lỗi cắt track. Ví dụ, ở track gold 5: ByteTrack nhảy từ ID 23 sang 32 ở frame 94; ReID cũng nhảy từ ID 17 sang 18 ở frame 87. Lỗi IDSW vẫn xảy ra nhưng dời sang điểm khác.
*(Lưu ý: Không thể quy hoàn toàn sự chênh lệch này là do ReID, vì đây là bài test cấp hệ thống giữa hai tracker implementation khác nhau).*

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**
- Từ ByteTrack sang ReID: DetA tăng (0.649 -> 0.711), FN giảm mạnh (54 -> 26), nhưng FP lại tăng nhẹ (88 -> 91).
- Điều này chứng tỏ ReID "vớt" được nhiều xe bị thiếu dấu (giảm FN) nhưng đồng thời detector cũng khoanh dư thêm một chút (tăng FP). Lỗi còn lại là sự kết hợp của cả detector (FP còn cao) và association (IDSW vẫn bằng 2, vẫn tách track).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**
- **Frame 51–52, ReID ID 7:** ReID bắt một đối tượng tĩnh (biển/quầy quảng cáo có in hình xe hoặc vật thể giống xe). Tôi đúng khi không gán vì luật rõ ràng: chỉ gán xe thực tế, loại trừ các vật thể hình ảnh 2D, biển báo tĩnh. ReID bị dính False Positive tại đây.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**
- **Frame 105–106 (ID 6 bản tay vs T18 của ReID):** Đây là xe đang bị che khuất. Hai bên gán cùng một xe nhưng bbox lệch nhau rất nhiều dẫn đến mâu thuẫn lớn nhất trong disagree list. Evidence này nhắc tôi phải soi lại frame này để chỉnh lại mép của bbox sao cho chỉ ôm sát phần "xe còn nhìn thấy được" thay vì vẽ lấn sang vùng bị che khuất.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong GUIDELINE:** Nhấn mạnh luật dùng thuộc tính `outside` (bắt buộc đóng track ngay frame chiếc xe rời khỏi màn hình hoặc chưa xuất hiện) và luật "Chỉ ôm phần nhìn thấy" khi có occlusion. Bổ sung luật loại trừ False Positive nghiêm ngặt đối với vật thể tĩnh.
- **Đổi quy trình (làm Solo):** Vì làm độc lập không có peer review, tôi sẽ thêm "Lượt rà soát số 4" (kiểm tra đối chiếu chéo các mid-frame tự động bằng script hoặc tua ngược) để phát hiện bbox trôi. Chỉ dùng model tracking làm tham chiếu *sau khi* đã tự lock bản nhãn tay để học hỏi các FP/FN.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md`
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/REPORT.md`