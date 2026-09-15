# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Đoàn Tấn Phát (Solo)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung cá nhân: Không gán các vật thể tĩnh có hình thù tương tự như quầy hàng, hộp đèn báo hiệu.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật cá nhân | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Do đối tượng vẫn có thể được xác định tính liên tục một cách rõ ràng. |
| Xe bị che lâu hơn ngưỡng trên | **Cắt track, gán ID mới** khi hiện lại | Để tránh nhầm lẫn identity với xe khác có ngoại hình tương tự và làm giảm sai số AssA kéo dài. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Một lần rời cảnh là một vòng đời kết thúc. |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID, vẽ bbox bám chặt vào phần còn nhìn thấy. | Đảm bảo tính liên tục của xe bị che khuất một phần. |

## 3. Luật bbox

| Tình huống | Luật cá nhân |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được hình khối là xe bốn bánh; ngưỡng cá nhân: khoảng 15x15 pixels và có di chuyển |
| Xe đang đỗ, không di chuyển | Đặt 1 keyframe lúc xuất hiện và 1 keyframe lúc kết thúc/bắt đầu di chuyển. Interpolation tự xử lý ở giữa. |
| Keyframe đặt dày ở đâu | Ở các đoạn xe thay đổi gia tốc, phanh lại, chuyển làn hoặc đang xen vào vùng bị occlusion. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01` / Frame 51-52 / ID Model T7.
- Tình huống: Model ReID bắt nhận diện một vật thể tĩnh bên đường (quầy hàng/biển quảng cáo).
- Quyết định: Bỏ qua, không gán nhãn.
- Lý do: Đối tượng không phải là xe ô tô lưu thông thực tế. Đây là False Positive điển hình của model AI.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame 149-151 / ID 4.
- Tình huống: Xe đã hoàn toàn di chuyển ra ngoài khung hình nhưng bbox (chưa set outside) vẫn bị treo lơ lửng ở mép viền trong 3 frame tiếp theo.
- Quyết định: Kích hoạt thuộc tính `outside` ngay từ frame 149.
- Lý do: Bbox treo tạo ra lỗi False Positive ở đoạn cuối track (theo đánh giá của công cụ Evaluate).

### Ca 3
- Clip / frame / ID: `clip_01` / Frame 105-106 / ID 6.
- Tình huống: Xe đi vào khu vực bị xe buýt che một nửa, bbox nội suy (interpolation) đang bị trôi rộng ra và bao gồm cả phần đã bị che.
- Quyết định: Thêm keyframe mới tại frame 105 và thu hẹp bbox lại.
- Lý do: Tuân thủ luật "chỉ ôm phần nhìn thấy được", tránh IoU bị drop xuống dưới 0.5 dẫn đến lệch với Gold.

## 5. Sửa gì sau khi chấm với gold

- **Quy tắc Outside tuyệt đối:** Bắt buộc phải đánh dấu `outside` để cắt track ở frame đầu tiên xe chưa vào và frame đầu tiên xe ra khỏi khung. Không được để bbox trôi tự do ở viền.
- **Rà soát Mid-Frame:** Không được tin tưởng tuyệt đối vào linear interpolation của CVAT. Bắt buộc phải cuộn chuột kiểm tra frame ở giữa đường đi của track, vì thực tế xe di chuyển theo cung tròn, gia tốc không đều (làm phát sinh lỗi trôi bbox ở các frame 83, 96...).