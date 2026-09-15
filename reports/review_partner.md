# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Đoàn Tấn Phát (2A202602152)` |
| Reviewer | `Đoàn Tấn Phát (Self-Review / Solo)` |
| Pair ID | `N/A (Solo)` |
| CVAT version | `[Điền phiên bản CVAT]` *(Cách lấy: Nhìn trên thanh công cụ của giao diện trang web CVAT bạn đang dùng, thường có ghi phiên bản góc trên cùng hoặc trong menu "About", ví dụ: 2.15.0)* |
| Thời điểm review | `2026-09-15` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 83 | 83 | 5 | Bbox trôi | Bbox bị lệch khỏi vật thể (chỉ còn IoU 0.57). Rule: Bbox ôm phần nhìn thấy, không bị interpolation drift. | Tua đến frame 83, thêm keyframe và điều chỉnh lại mép bbox cho khớp với xe. | fixed |
| 2 | 149-151 | 149-151 | 4 | Bbox treo | Xe đã rời khung nhưng bbox vẫn còn treo 3 frame sau đó. Rule: Entry/exit đúng. | Đặt thuộc tính `outside` cho track 4 bắt đầu từ frame 149. | fixed |
| 3 | 51-78 | 51-78 | 5 | Bbox treo | Đã có bbox trước khi track 5 thực sự xuất hiện trong 28 frame. Rule: Entry/exit đúng. | Tìm frame xe bắt đầu lộ diện, đặt `outside` ở các frame trước đó. | fixed |
| 4 | 16-116 | 16-116 | T7 | Bbox thừa (từ Model) | BoT-SORT+ReID tạo bbox không khớp track nào (bắt vật thể tĩnh). Rule: Chỉ gán xe bốn bánh. | Không gán, đây là False Positive của model, nhãn tay đã làm đúng. | not-a-defect |

*(Hướng dẫn lấy thêm lỗi nếu cần: Mở terminal, xem kết quả của lệnh chạy `evaluate_tracking.py` lúc nãy. Lấy các frame nằm ở mục "3. BBOX TREO" hoặc "4. BBOX TRÔI" để điền thêm vào bảng trên).*

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đã vẽ đủ 8 track, đều là phương tiện bốn bánh hợp lệ. |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Kết quả evaluate cho thấy IDSW = 0. |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 5 và 6 có overlap/bị che khuất nhưng không đổi ID. |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Phát hiện lỗi ở các frame 51-78 (ID 5), 149-151 (ID 4)... |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | IoU giảm do nội suy (interpolation drift) hoặc vẽ lấn. |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Lệch ở các frame 83, 96, 107... do chuyển động của xe không tuyến tính. |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Hệ thống `motlib` parse thành công 646 bbox. |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đã ghi nhận và đóng toàn bộ. |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Xác nhận 8 track ID liên tục, không bị nhảy số ngẫu nhiên. |
| 2 — endpoint/scope | ĐÃ SỬA | Đã cắt gọt bằng `outside` các đoạn đầu/cuối của ID 4, 5, 6, 7, 8. |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã thêm keyframe rà soát lại các frame 83, 96, 107, 109, 136, 190. |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Lỗi bbox trôi giữa 2 keyframe (interpolation drift). Rule: Frame giữa hai keyframe không bị interpolation drift. Cần soi mid-frame chứ không phó mặc cho phần mềm nội suy.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Lỗi số 4. Model ReID báo có track T7 kéo dài 43 frame nhưng thực tế là đối tượng tĩnh không phải xe, tác giả đã không gán là chính xác.`
3. Một rule cần Lab Coach làm rõ (nếu có): `Quy định cụ thể số lượng frame bị che khuất tối đa (occlusion) là bao nhiêu thì nên ngắt track và khởi tạo ID mới?`