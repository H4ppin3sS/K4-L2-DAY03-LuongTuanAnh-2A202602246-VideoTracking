# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lương Tuấn Anh`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                            | Không gán                                                   |
| ------------------------------- | ------------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                               |
| van, minivan                    | xe đạp                                                      |
| xe buýt, minibus               | **xe máy / mô tô**                                   |
| xe tải, xe đầu kéo          | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống                          | Luật của nhóm                                                                                             | Vì sao                                                                                                                  |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che**dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | 25 frame tương đương khoảng 2 giây ở 12.5 fps, giúp duy trì cùng một identity khi xe chỉ bị che tạm thời |
| Xe bị che lâu hơn ngưỡng trên   | Chưa có quy định riêng trong dữ liệu đã ghi nhận                                                   | cần thống nhất thêm khi gặp trường hợp thực tế                                                                 |
| Xe rời khung hình rồi quay lại    | mặc định:**track mới**                                                                             | tránh nối nhầm một xe cũ với một xe khác có appearance tương tự                                              |
| Hai xe cắt nhau / chồng lên nhau   | giữ ID theo identity trước khi overlap; soi frame trước và sau vùng chồng để xác định xe        | khi overlap, chỉ dựa vào vị trí có thể gây đổi ID; cần theo dõi liên tục trước/sau vùng che             |

## 3. Luật bbox

| Tình huống                                   | Luật của nhóm                                                                                                                                       |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Xe bị cắt bởi rìa ảnh                     | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                |
| Xe bị xe khác che một phần                 | bbox ôm phần**nhìn thấy được**                                                                                                            |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn:`...`                                             |
| Xe đang đỗ, không di chuyển               | vẫn gán nếu là vehicle hợp lệ và còn nằm trong scene; không loại chỉ vì không chuyển động                                             |
| Keyframe đặt dày ở đâu                   | đặt dày hơn tại vùng xe đổi hướng, overlap/occlusion, vào hoặc ra khỏi khung; thêm keyframe khi interpolation làm bbox lệch đáng kể |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: Clip / frame / ID: `clip_01 / frame 107 / ID 6`
- Tình huống: hai bản annotation khoanh cùng một xe nhưng bbox khác nhau rõ rệt, IoU chỉ còn 0.53.
- Quyết định: thống nhất lại cách khoanh bbox và sửa theo cùng một quy tắc.
- Lý do: `đây là sai khác về phạm vi bbox của cùng một xe, không phải khác identity. Kết quả kiểm chéo ghi nhận frame 107, track 6 có IoU 0.53.`

### Ca 2

- Clip / frame / ID: clip_01 / frame 140 / ID 3
- Tình huống: hai bản cùng gán một xe nhưng bbox khác nhau, IoU chỉ còn 0.56.
- Quyết định: thống nhất luật khoanh bbox rồi sửa cả hai bản.
- Lý do: cần dùng một quy tắc nhất quán về việc bbox ôm phần xe nhìn thấy, thay vì mỗi người khoanh rộng/hẹp khác nhau.

### Ca 3

- Clip / frame / ID: clip_01 / frame 113 / ID 6
- Tình huống: `xảy ra đồng thời vấn đề về ID/bbox trong vùng khó; kiểm chéo ghi nhận khác biệt ID và bbox giữa hai bản.`
- Quyết định: kiểm tra frame trước/sau để xác định identity, sau đó thống nhất bbox theo cùng quy tắc.
- Lý do: frame 113 nằm trong nhóm bbox lệch của track 6 và cũng là một frame có khác biệt identity được phát hiện trong kiểm chéo.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

* Luật bắt đầu/kết thúc track cần rõ hơn: không để bbox tồn tại trước khi xe tham chiếu xuất hiện hoặc sau khi xe đã rời khung. Kết quả so với gold phát hiện các bbox thừa/treo ở ID 6, 5, 4, 7 và 8.
* Luật khoanh bbox cần thống nhất hơn: cùng một xe phải được khoanh nhất quán, đặc biệt ở các frame khó như 107, 113–115, 138, 140 và 190.
* Cần quy định rõ cách xử lý vùng occlusion/overlap: phải kiểm tra identity ở frame trước và sau vùng che, không đổi ID chỉ vì hai xe tạm thời chồng lên nhau.
* Cần thêm quy tắc khi xe đã rời khung: sử dụng `outside` đúng frame kết thúc để tránh bbox bị kéo dài sang các frame không còn xe. Kết quả gold cho thấy đây là một nguồn lỗi thực tế.
