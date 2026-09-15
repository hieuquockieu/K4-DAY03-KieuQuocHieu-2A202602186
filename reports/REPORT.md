# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Kiều Quốc Hiếu` / cá nhân
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Track Mode |
| Thời gian gán `clip_02` (warm-up) | `chưa ghi lại` phút |
| Thời gian gán `clip_01` | `chưa ghi lại` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `chưa thể suy ra từ file MOT` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe đi ra khỏi mép ảnh: kết thúc track ở frame cuối còn nhìn thấy, không đoán phần ngoài ảnh và không reuse ID khi xuất hiện lại.
2. Hai xe gần/chồng lên nhau quanh frame 61–62: giữ hai ID riêng, kiểm tra hướng di chuyển và vị trí ở các frame kế tiếp.
3. Bbox thay đổi nhanh hoặc bị che: đặt keyframe dày hơn ở vùng chuyển động/che khuất, chỉ ôm phần xe nhìn thấy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: không phát hiện ID switch khi đối chiếu với gold (`IDSW = 0`).
- Lượt 2: phát hiện các đoạn bbox treo/biên track cần kiểm tra ở ID 4 frame 61–78, ID 6 frame 87–100 và ID 5 frame 149–151.
- Lượt 3: phát hiện bbox trôi ở các vùng quanh frame 70–81, 113–124 và 153; cần thêm keyframe hoặc chỉnh hình học.

Validator MOT: cả `clip_01` và `clip_02` đều có `0 lỗi định dạng`; có cảnh báo bbox gần như đứng im ở một số track cần kiểm tra bằng mắt.

Kiểm chéo với: `chưa có reports/review_partner.md`. Chi tiết peer review chưa được lưu.
Số lỗi bạn tìm được trong bản của bạn ấy: `chưa có dữ liệu`. Số lỗi bạn ấy tìm được trong bản của bạn: `chưa có dữ liệu`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có biên bản peer review nên chưa thể kết luận ca bất đồng. Luật cần ghi rõ thêm là luôn quy đổi `MOT frame = CVAT frame + 1` trong evidence và phải kết thúc track ngay khi xe rời khung, không để bbox treo.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `cc74c5193d3338601dd872e79d0394484b8aae00cf650d5e19d3a0f58ca604e3` |
| Thời điểm khóa | `2026-09-15T10:14:33.404117+00:00` |
| Số row / frame / track trước khi mở reference | `590 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `0.6971` | `0.6765` | `0.7204` | `0.8403` | `0.8822` | `0.7609` | `0.8288` | `77` | `60` | `0` |
| Sau rework | `chưa chạy` | `chưa chạy` | `chưa chạy` | `chưa chạy` | `chưa chạy` | `chưa chạy` | `chưa chạy` | `-` | `-` | `-` |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo trước khi track tham chiếu xuất hiện | 61–78 | 4 | Chưa có bản export sau rework để xác nhận đã sửa |
| Bbox treo trước khi track tham chiếu xuất hiện | 87–100 | 6 | Chưa có bản export sau rework để xác nhận đã sửa |
| Bbox treo sau khi track tham chiếu rời khung | 149–151 | 5 | Chưa có bản export sau rework để xác nhận đã sửa |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / 2, 5, 7` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | `0.6971` | `0.6765` | `0.7204` | `0.8403` | `0.8822` | `0.7609` | `0.8288` | `77` | `60` | `0` |
| ByteTrack control vs gold | `0.7085` | `0.6487` | `0.7761` | `0.8463` | `0.8746` | `0.7487` | `0.8226` | `88` | `54` | `2` |
| BoT-SORT + ReID vs gold | `0.7635` | `0.7110` | `0.8204` | `0.8721` | `0.9001` | `0.7923` | `0.8595` | `91` | `26` | `2` |
| ReID vs bạn | `0.6370` | `0.5803` | `0.7053` | `0.8307` | `0.8208` | `0.6322` | `0.8100` | `131` | `83` | `3` |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA thấp hơn IDF1 (`0.7609` so với `0.8822`). MOTA cộng FP, FN và IDSW trên tổng số ground-truth detection, còn IDF1 đo độ nhất quán của identity sau ghép cặp. Vì vậy MOTA không phạt riêng và đủ mạnh cho các lỗi nhận nhầm identity nếu số FP/FN vẫn nhỏ; một tracker có thể bắt đúng vị trí nhưng đổi ID mà MOTA vẫn tương đối cao.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack trên các chỉ số chính: IDF1 `0.9001` so với `0.8746`, AssA `0.8204` so với `0.7761`, HOTA `0.7635` so với `0.7085`, và MOTA `0.7923` so với `0.7487`. Cả hai đều có `2 IDSW`, nên ReID cải thiện identity association tổng thể nhưng không giảm số switch trong clip này. ByteTrack switch ở frame 59 (gold ID 4: model ID 14 -> 15) và frame 94 (gold ID 5: 23 -> 32); ReID switch ở frame 87 (gold ID 5: 17 -> 18) và frame 113 (gold ID 6: 24 -> 31). ReID cũng bắt phủ tốt hơn, FN giảm `54 -> 26`, nhưng FP tăng `88 -> 91` và số bbox dự đoán tăng `607 -> 638`. Đây không phải causal effect cô lập của ReID vì hai tracker implementation khác nhau, dù detector input và cấu hình detection được giữ cố định.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với gold, ByteTrack có `DetA 0.6487`, `FP 88`, `FN 54`, còn ReID có `DetA 0.7110`, `FP 91`, `FN 26`. ReID giảm FN `28` và tăng DetA, nhưng tăng FP `3`; vì vậy treatment bắt được nhiều xe hơn nhưng cũng sinh thêm bbox thừa. Association của ReID nhìn tốt hơn qua AssA `0.8204` và IDF1 `0.9001`, nhưng vẫn còn switch ở frame 87 và 113. Với annotation của mình, `DetA = 0.6765`, `FP = 77`, `FN = 60`, `IDSW = 0`: lỗi còn lại chủ yếu là coverage/bbox, không phải association ID.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở frame 87, annotation của mình giữ ID 4 cho xe có bbox gần `x=685.84`, đúng với gold ID 5. ReID lại tách xe này từ model ID 17 ở frame 85–86 sang ID 18 từ frame 87; JSON ghi rõ switch `gold track 5: 17 -> 18`. Vì gold và annotation đều giữ cùng identity qua đoạn này, đây là một chỗ annotation đúng còn ReID sai.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Ở frame 113, ReID đổi gold ID 6 từ model ID 24 sang ID 31, trong khi annotation của mình vẫn giữ ID 6 liên tục và gold cũng giữ ID 6. Đây là evidence để xem lại association của model, không phải lý do sửa annotation. Ngoài ra, ReID-vs-me có `IDSW = 3`, `FP = 131`, `FN = 83`, thấp hơn rõ rệt so với annotation-vs-gold (`IDSW = 0`, `FP = 77`, `FN = 60`), nên model không đủ bằng chứng để thay thế nhãn tay.

## 6. Nếu phải gán thêm 10 clip nữa

Mình sẽ bổ sung vào `GUIDELINE_MINI.md` quy tắc quy đổi frame CVAT/MOT, điều kiện kết thúc track khi chạm mép ảnh, và cách ghi finding bắt buộc theo `frame + ID + rule + cách sửa`. Trong quy trình, mình sẽ khóa pre-gold sớm, tua riêng ba lượt theo identity/endpoint/geometry, đặt keyframe dày ở vùng chuyển động nhanh hoặc giao nhau, rồi chạy validator và evaluate trước khi chạy model. Mỗi lần rework sẽ export lại MOT và ghi metric trước/sau.

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
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md`