# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: `Nguyễn Thị Thùy Linh - 2A202602234`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT 2.74.1 (chế độ Rectangle Track) |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 75 phút |
| Số track đã vẽ trong `clip_01` | 8 track |
| Số keyframe trung bình mỗi track | 14 keyframe / track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị khuất một phần và hình ảnh mờ ở xa (ID 6):** Khi xe xuất hiện ở rìa trên đường chân trời (frame 81-100), xe bị lẫn vào bóng cây và độ phân giải thấp. Xử lý: Ban đầu vẽ từ frame 81, nhưng sau khi đối chiếu đã lùi lại bắt đầu từ frame 101 khi xe đã hiện rõ dáng xe bốn bánh để tránh sinh bbox thừa (FP).
2. **Xe phanh và giảm tốc đột ngột (ID 5):** Giữa hai keyframe cách nhau khoảng 15 frame, xe phanh để nhường đường khiến chuyển động phi tuyến, dẫn đến bbox nội suy bị trôi (drift, IoU tụt xuống 0.53). Xử lý: Bổ sung thêm keyframe ngay tại frame 81-82 để kéo khít lại bbox bám sát thân xe.
3. **Xe rời khỏi khung hình (ID 4 và ID 8):** Rất dễ quên bấm `Outside` (phím `O`) tại frame xe vừa khuất hẳn biên ảnh. Xử lý: Rà soát lại frame cuối cùng còn thấy xe và bấm `Outside` ngay frame tiếp theo để chấm dứt track, không để bbox treo lơ lửng ngoài khung.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (Identity / Timeline):** Phát nhanh toàn bộ clip ở tốc độ cao, mắt tập trung vào con số ID trên các xe. Kết quả: Tất cả 8 xe đều giữ nguyên ID duy nhất xuyên suốt quãng đời, không xảy ra hiện tượng nhấp nháy hoặc đổi số ID giữa chừng (ID Switch = 0).
- **Lượt 2 (Entry / Exit):** Kiểm tra từng frame đầu và cuối của 8 track. Phát hiện 4 điểm nghi ngờ: ID 4 bị thừa 3 frame sau khi ra khung (frame 149-151), ID 8 bị thừa 3 frame (frame 169-171), ID 4 mở sớm 3 frame (frame 51-53) và ID 6 mở sớm 20 frame (frame 81-100). Đã tiến hành bấm Outside và cắt bớt các box thừa.
- **Lượt 3 (Geometry / Midpoint):** Nhảy vào các frame nằm chính giữa hai keyframe xa nhau. Bắt được hiện tượng trượt box ở ID 5 tại frame 81-82 (IoU đạt 0.53 - 0.57). Đã chèn thêm keyframe tại các frame này để đạt độ khít cao.

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `f83e1ef02bb1dbae5e6cfdfeb57886c06034d7f2e92fbe5da730319a39930e71` |
| Thời điểm khóa | `2026-09-15T03:43:34.253934+00:00` |
| Số row / frame / track trước khi mở reference | 612 rows / 190 frames / 8 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.836 | 0.819 | 0.856 | 0.894 | 0.964 | 0.925 | 0.884 | 41 | 2 | 0 |
| Sau rework | 0.865 | 0.852 | 0.878 | 0.910 | 0.985 | 0.962 | 0.905 | 15 | 1 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT (Cả 3 chỉ số đều vượt ngưỡng yêu cầu)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa (mở quá sớm) | 81-100 | 6 | Xóa bbox ở 20 frame này, dời điểm bắt đầu của track 6 sang frame 101 khi xe rõ nét |
| Bbox treo (quên Outside) | 149-151 | 4 | Bấm phím O (`Outside`) tại frame 149 để cắt dứt điểm 3 frame thừa ngoài rìa ảnh |
| Bbox thừa (mở quá sớm) | 51-53 | 4 | Chỉnh lại frame bắt đầu cho khớp với thời điểm xe lộ diện trong khung hình |
| Bbox treo (quên Outside) | 169-171 | 8 | Bấm phím O (`Outside`) tại frame 169 ngay sau khi xe biến mất |
| Bbox trôi (drift IoU thấp) | 81-82 | 5 | Chèn thêm keyframe tại frame 81 và 82 để căn chỉnh bbox ôm sát phần thân xe nhìn thấy |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cpu / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml & configs/trackers/botsort-reid.yaml |
| conf / IoU / imgsz / classes | conf=0.25 / iou=0.70 / imgsz=960 / classes=[2, 5, 7] |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.836 | 0.819 | 0.856 | 0.894 | 0.964 | 0.925 | 0.884 | 41 | 2 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.752 | 0.699 | 0.811 | 0.886 | 0.882 | 0.761 | 0.876 | 85 | 59 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả của tôi: IDF1 đạt **0.964**, cao hơn MOTA (**0.925**). Điều này phản ánh việc duy trì danh tính phương tiện (identity tracking) được thực hiện rất xuất sắc với **0 lần ID switch** trong suốt 190 frame.
- Nếu một hệ thống có **MOTA rất cao nhưng IDF1 lại thấp**, điều đó cho thấy mô hình phát hiện vị trí xe rất tốt (ít FP, ít FN) nhưng bị lỗi phân mảnh danh tính nghiêm trọng (ID switches hoặc track fragmentation).
- Nguyên nhân: Công thức MOTA chỉ đếm số lần chuyển đổi ID (IDSW) đúng 1 lần duy nhất tại frame xảy ra lỗi chuyển đổi ($1 - \frac{FP + FN + IDSW}{GT}$). Do đó, một track dài 100 frame bị cắt làm đôi chỉ mất 1 điểm phạt trong MOTA (MOTA vẫn có thể trên 90%). Ngược lại, **IDF1** đo lường mức độ trùng khớp danh tính trên toàn bộ vòng đời của xe ($IDTP / (IDTP + 0.5(IDFP + IDFN))$), nên một lần nhảy ID sẽ khiến một nửa số frame của track bị phạt thành IDFP/IDFN, làm IDF1 tụt dốc thảm hại.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Về số liệu:**
  - `BoT-SORT + ReID` vượt trội hơn `ByteTrack control` trên mọi phương diện tracking: IDF1 tăng từ **0.875 lên 0.900** (+0.025), AssA tăng từ **0.776 lên 0.820** (+0.044), HOTA tăng từ **0.709 lên 0.763** (+0.054). Số lượng FN giảm một nửa từ 54 xuống còn 26. Cả hai mô hình đều có 2 lần ID switch.
- **Phân tích Frame Sequence:**
  - Ở đoạn xe bị che khuất (occlusion) quanh frame 59 và 94: ByteTrack chỉ dựa vào chuyển động tuyến tính (Kalman filter) và IoU nên khi xe bị xe khác che mất vài frame, IoU với box dự đoán giảm sút khiến ByteTrack bị mất dấu và cấp ID mới (track 4 bị cắt thành ID [14, 15], track 5 bị cắt thành ID [23, 32]).
  - BoT-SORT + ReID nhờ trích xuất vector đặc trưng ngoại hình (appearance embedding) nên sau khi xe lộ diện trở lại, khoảng cách cosin đặc trưng thị giác vẫn nhận ra đây là xe cũ, giúp duy trì track ổn định hơn và khôi phục track sau che khuất tốt hơn.
- **Lưu ý quan trọng về tính nhân quả:** Kết quả này phản ánh sự so sánh giữa **hai hệ thống tổng thể** (system comparison). Do ByteTrack và BoT-SORT khác biệt về cơ chế liên kết dữ liệu, thuật toán khớp và bộ lọc Kalman, ta **không thể kết luận một mình ReID là nguyên nhân duy nhất** tạo ra sự chênh lệch này.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- Khi chuyển từ ByteTrack sang BoT-SORT + ReID:
  - FP tăng nhẹ từ **88 lên 91** (+3).
  - FN giảm mạnh từ **54 xuống 26** (giảm hơn 50%).
  - DetA tăng từ **0.649 lên 0.711** (+0.062).
- Nhận định: Lỗi chủ yếu còn lại nằm ở **detector (phần phát hiện)** chứ không phải do association. Các lỗi FP (91) phần lớn do detector nhận nhầm các chi tiết biển hiệu, tán cây hoặc góc phản chiếu thành vehicle ở ngưỡng confidence thấp (conf=0.25). Lỗi association đã được ReID cải thiện đáng kể thể hiện qua AssA đạt 0.820.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Vị trí:** Frame 111 (xem ảnh worst frame trong notebook).
- **Hiện tượng:** Tại frame 111, nhãn thủ công của tôi gán đúng **1 xe**, trong khi model ReID sinh ra **3 bbox** (thừa 2 bbox).
- **Nguyên nhân:** Model bị hiện tượng "ghost detection" và phân mảnh track: Detector bắt nhầm một vật thể cố định bên lề đường và chia một xe đang di chuyển thành hai bbox chồng lấn nhau. Con người có tri giác ngữ cảnh tốt hơn nên không bị mắc lỗi nhận dạng này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Vị trí:** Frame 81-100 ở xe ID 6.
- **Hiện tượng:** Model không phát hiện ra xe ID 6 ở đoạn frame 81-100, trong khi bản nhãn ban đầu của tôi đã bắt đầu vẽ từ frame 81.
- **Đánh giá evidence:** Khi soi lại từng frame ở đoạn này, xe ID 6 thực tế ở khoảng cách quá xa, kích thước chưa tới $10 \times 10$ pixel và bị bóng cây che gần hết. Việc model không bắt được ở đoạn này là hợp lý, và chính điều này đã giúp tôi nhận ra mình đã gán quá sớm (over-annotating), từ đó sửa lại điểm bắt đầu của ID 6 về frame 101 để trùng khớp với gold reference.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

1. **Cải tiến trong `GUIDELINE_MINI.md`:**
   - Đưa ra định lượng pixel tối thiểu cụ thể (ví dụ: chiều cao $\ge 20$ px, chiều rộng $\ge 20$ px) để bắt đầu mở track, tránh việc mỗi người ước lượng độ "rõ" theo cảm tính.
   - Thêm hình ảnh minh họa cho các trường hợp xe che khuất phức tạp (3 xe cắt nhau cùng lúc).
2. **Thay đổi trong quy trình làm việc:**
   - Áp dụng triệt để quy tắc **"Làm dứt điểm từng xe"**: Hoàn thành trọn vẹn 1 xe từ lúc xuất hiện đến lúc ra khung rồi mới chuyển sang xe khác.
   - Luôn thực hiện **"Quy tắc 3 lượt tua"** và chạy script `visualize_tracks.py` để kiểm tra trực quan trước khi chốt nhãn.
   - Tận dụng phím tắt CVAT hiệu quả hơn: Luôn dùng phím `O` cho frame rời khung và phím `Q` cho các đoạn bị che khuất.

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
