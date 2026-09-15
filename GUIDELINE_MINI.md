# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Thị Thùy Linh - 2A202602234`
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

Bổ sung của nhóm:
- Xe đang dừng chờ đèn đỏ hoặc đỗ ven đường vẫn phải gán nhãn đầy đủ và giữ nguyên `track_id` trong suốt thời gian tồn tại trong khung hình.
- Xe kéo rơ-moóc hoặc thùng hàng: Gán chung một bbox bao trọn cả đầu kéo và rơ-moóc nếu chúng di chuyển cùng nhau như một khối thống nhất.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Tránh hiện tượng ID Switch (IDSW) và phân mảnh track (fragmentation). Trong khoảng 2 giây, quỹ đạo chuyển động của xe vẫn có tính liên tục cao. |
| Xe bị che lâu hơn ngưỡng trên | Mở `track_id` mới | Quá 25 frame, khả năng nhầm lẫn với phương tiện khác tăng vọt; gán ID mới an toàn hơn việc đoán mò danh tính. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Quy tắc bất biến của MOT: Đã biến mất hoàn toàn khỏi biên khung hình là kết thúc vòng đời của track đó. |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID gốc của từng xe, không hoán đổi | Hai xe di chuyển khác làn/hướng: xe đi trước che xe sau thì xe sau bật cờ `Occluded`, khi tách ra xe nào giữ nguyên ID xe đó. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh (tọa độ giới hạn ở biên ảnh), **tuyệt đối không đoán** phần thân xe ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox chỉ ôm sát **phần nhìn thấy được** (visible part) của xe; kích hoạt thuộc tính `Occluded` (`phím Q`). |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định rõ ràng là xe bốn bánh (tối thiểu kích thước $15 \times 15$ pixel, thấy rõ đèn/kính/nóc xe). |
| Xe đang đỗ, không di chuyển | Duy trì bbox bao quanh xe, đặt keyframe thưa (10-20 frame/lần) nếu góc camera tĩnh; kiểm tra tránh trôi box. |
| Keyframe đặt dày ở đâu | Đặt keyframe dày (cách 2-5 frame) ở các đoạn: xe rẽ cua, tăng/giảm tốc đột ngột, bắt đầu bị che khuất và vừa ló ra khỏi vật cản. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 81-100 / ID 6`
- Tình huống: Xe xuất hiện ở rìa xa, hình ảnh còn mờ và bị lẫn vào bóng cây trước khi tiến hẳn vào làn đường chính.
- Quyết định: Ban đầu gán sớm từ frame 81 nhưng đối chiếu lại gold reference chỉ tính từ frame 101 khi xe đã lộ rõ toàn bộ thân trước.
- Lý do: Tránh bbox thừa (False Positive) khi vật thể chưa đủ điều kiện nhận dạng chắc chắn là xe bốn bánh đang lưu thông.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 51-53 & 149-151 / ID 4`
- Tình huống: Xe di chuyển ra khỏi rìa khung hình bên phải ở frame 148, nhưng bbox vẫn bị kéo dài thêm 3 frame (treo lơ lửng).
- Quyết định: Bấm `Outside` (phím `O`) ngay tại frame 149 để cắt đuôi track.
- Lý do: Không bấm `Outside` sẽ khiến CVAT tiếp tục suy diễn bbox ở khoảng trống ngoài khung hình, tạo ra bbox "ma" (ghost box) làm tăng FP.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 81-82 / ID 5`
- Tình huống: Bbox tại frame giữa hai keyframe xa nhau bị trôi (drift), IoU tụt xuống 0.53 - 0.57 do xe bắt đầu giảm tốc nhường đường.
- Quyết định: Chèn bổ sung một keyframe ngay tại frame 81 để điều chỉnh lại bbox ôm khít thân xe.
- Lý do: Tuyến tính nội suy (linear interpolation) không phản ánh đúng gia tốc thực tế của xe khi phanh đột ngột.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy tắc bấm Outside triệt để:** Bắt buộc bấm phím `O` ngay frame đầu tiên mà xe không còn nhìn thấy bất kỳ điểm ảnh nào trong khung hình (loại bỏ dứt điểm các lỗi ghost bbox ở frame 149-151 và 169-171).
- **Ngưỡng bắt đầu gán xe mới vào khung:** Chỉ mở track khi xe đã tiến vào khung hình tối thiểu 20% thân xe hoặc nhận diện được rõ ràng cụm đèn/bánh xe (khắc phục lỗi mở track quá sớm ở ID 4 frame 51-53 và ID 6 frame 81-100).
- **Rà soát điểm trôi (Midpoint check):** Bắt buộc tua lại kiểm tra các điểm giữa hai keyframe cách nhau trên 10 frame, đặc biệt tại các khúc xe phanh hoặc rẽ làn.

