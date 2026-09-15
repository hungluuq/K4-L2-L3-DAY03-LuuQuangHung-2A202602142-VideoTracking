# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lưu Quang Hùng`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Tránh đứt gãy ID không cần thiết đối với các vật cản nhỏ (như cột điện, biển báo, xe khác cắt ngang nhanh).` |
| Xe bị che lâu hơn ngưỡng trên | `Bắt buộc cấp track/ID mới khi xe xuất hiện trở lại.` | `Tránh tracker nội suy (interpolate) sai quỹ đạo quá dài, dẫn đến hỏng data bounding box trong khoảng thời gian bị che.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Coi như một object mới xuất hiện vào camera để đảm bảo tính nhất quán của thuật toán ReID.` |
| Hai xe cắt nhau / chồng lên nhau | `Zoom cận cảnh, đặt keyframe liên tục (mỗi 1-2 frame) trong suốt quá trình đan chéo. Bao Bbox ôm sát phần xe nhìn thấy.` | `Đây là nguyên nhân chính gây lỗi ID Switch (như lỗi model chuyển ID 14 -> 15). Cần gán nhãn thủ công dày đặc để giữ đúng ID.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | `Vẫn gán ID bình thường và giữ nguyên Bbox, chỉ đặt keyframe ở frame đầu và frame cuối của chuỗi đỗ để CVAT tự giữ trạng thái tĩnh` |
| Keyframe đặt dày ở đâu | `Ở các đoạn xe đổi hướng, chuyển động nhanh, lúc xe bắt đầu bị che khuất (occlusion) hoặc khi xe đi ra xa dần làm kích thước thay đổi liên tục` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 0 / ID 8`
- Tình huống: `Xe đứng yên không di chuyển`
- Quyết định: `Làm cuối cùng `
- Lý do: `Tránh việc nhiều ID bị rối`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 74 / ID 4`
- Tình huống: `Xe đen bị xe bus to chắn nhiều phần`
- Quyết định: `Đặt keyframe từ lúc nhìn thấy 1 phần của xe`
- Lý do: `Nếu để CVAT tự nội suy một đoạn dài, Bbox sẽ không ôm khít dễ dẫn đến lỗi Bbox lệch.`

### Ca 3
- Clip / frame / ID: `clip_01 / đoạn frame 17-101`
- Tình huống: `Có những chi tiết/vật thể ở lề đường nhìn lấp ló dễ nhầm với phần đầu của một chiếc xe đang đỗ hoặc bóng xe.`
- Quyết định: `Không gán nhãn (không gán ID) và kích hoạt tính năng Outside nếu xe chưa thực sự lộ diện phần thân rõ ràng.`
- Lý do: `Tránh tạo ra các Bbox thừa không khớp với bất kỳ xe nào `

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Thiếu luật xác định thời điểm bật Outside:** Cần quy định rõ khi xe bắt đầu đi ra khỏi mép ảnh, frame cuối cùng còn thấy dù chỉ 1 phần mép xe vẫn giữ Bbox, nhưng frame ngay tiếp theo phải nhấn phím `O` (Outside) ngay lập tức. Nếu bật chậm sẽ sinh ra lỗi **Bbox thừa lơ lửng** (ví dụ lỗi ở frame 163, 169).
- **Mơ hồ về quy tắc khoanh Bbox khi xe bị che (Occlusion):** Cần bổ sung rõ luật: Khi xe bị che một nửa, Bbox **chỉ được phép ôm khít phần vỏ xe đang nhìn thấy**, tuyệt đối không được tự ước lượng độ dài toàn bộ xe để vẽ Bbox trùm qua vật che khuất. Nếu làm sai sẽ dẫn tới lỗi **Bbox lệch (IoU < 0.5)** như đã bị ở các frame 118, 138, 166.
