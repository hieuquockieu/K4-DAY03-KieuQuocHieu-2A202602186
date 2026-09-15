# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Kiều Quốc Hiếu`
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

Bổ sung của nhóm (nếu có): chỉ gán khi chắc chắn là xe bốn bánh; không suy đoán
từ một vật thể quá nhỏ hoặc bị che gần như hoàn toàn.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Vẫn nhận ra cùng xe trước và sau khi bị che. |
| Xe bị che lâu hơn ngưỡng trên | đặt `Outside` khi xe biến mất; nếu xuất hiện lại thì tạo track/ID mới | Không nối một track khi mất dấu quá lâu. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Khi xe rời khỏi ảnh, không có đủ bằng chứng để đảm bảo danh tính lúc quay lại. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo hướng di chuyển, vị trí và đặc điểm xe trước/sau khi cắt; không đổi ID chỉ vì bị che | Kiểm tra frame-by-frame quanh lúc hai xe giao nhau. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; không gán nếu chưa thể phân biệt chắc chắn với vật thể khác |
| Xe đang đỗ, không di chuyển | vẫn giữ bbox và cùng ID ở các frame xe còn nhìn thấy; chỉ dùng keyframe khi hình dạng/vị trí thay đổi |
| Keyframe đặt dày ở đâu | đặt dày khi xe vào/ra khung, bị che, cắt nhau, đổi hướng hoặc bbox thay đổi nhanh; đoạn ổn định để CVAT nội suy |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / MOT frame 11-12 / ID 1
- Tình huống: ID 1 đi ra khỏi mép trái; có bbox ở frame 11 nhưng không còn ở frame 12.
- Quyết định: kết thúc track tại frame 11, không tạo lại ID 1 nếu xe xuất hiện lại.
- Lý do: xe đã rời khung hình; theo luật nhóm, lần xuất hiện sau là track mới.

### Ca 2
- Clip / frame / ID: `clip_01` / MOT frame 45-46 / ID 3
- Tình huống: ID 3 chạm mép trái ở frame 45 rồi biến mất từ frame 46.
- Quyết định: bbox chạm đúng rìa ảnh ở frame 45 và kết thúc track sau đó; không đoán phần xe ngoài ảnh.
- Lý do: bbox chỉ bao quanh phần nhìn thấy, đúng quy tắc xe bị cắt bởi rìa ảnh.

### Ca 3
- Clip / frame / ID: `clip_01` / MOT frame 61-62 / ID 4 và ID 5
- Tình huống: ID 4 xuất hiện sát mép phải ở frame 61; ID 5 xuất hiện ở frame 62 tại khu vực bên phải.
- Quyết định: giữ hai ID riêng, mỗi xe có một track; không gộp chỉ vì chúng gần hoặc chồng lên nhau.
- Lý do: hai xe là hai vật thể riêng và cần kiểm tra chuyển động từng xe qua frame kế tiếp.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi frame CVAT bắt đầu từ 0, phải ghi thêm MOT frame tương ứng (MOT frame = CVAT frame + 1) trong evidence.
- Sau khi tự kiểm và đối chiếu gold theo đúng mốc lab, nếu phát hiện ID hoặc bbox sai thì sửa trực tiếp trong CVAT, export lại MOT và ghi rõ frame, ID, lỗi và cách sửa ở đây.
