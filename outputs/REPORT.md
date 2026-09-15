# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lương Tuấn Anh`
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị    |
| ------------------------------------ | ------------ |
| Công cụ                            | CVAT        |
| Thời gian gán`clip_02` (warm-up) | `43` phút |
| Thời gian gán`clip_01`           | `83` phút |
| Số track đã vẽ trong`clip_01`  | `8`        |
| Số keyframe trung bình mỗi track  | `...`      |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất: Giữ nguyên track_id của xe khi xe bị che rồi xuất hiện lại, đồng thời đặt keyframe dày hơn ở những đoạn xe bị che hoặc chuyển động thay đổi. Quy tắc của guideline là xe bị che rồi hiện lại vẫn giữ ID cũ.
2. Xe rời khỏi khung hình: sử dụng outside tại frame xe thực sự rời khung để tránh bbox tiếp tục tồn tại ở những frame không còn xe.
3. Bbox bị trôi giữa hai keyframe: tua lại giữa các keyframe để kiểm tra interpolation và bổ sung keyframe tại những đoạn xe đổi hướng, phanh hoặc bị che. Đây cũng là cách guideline yêu cầu để hạn chế bbox trôi.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: kiểm tra tính liên tục của track_id,
- Lượt 2: kiểm tra frame bắt đầu và kết thúc của từng track, tập trung vào việc xe đã thực sự xuất hiện hay đã rời khung và có bbox treo hay không
- Lượt 3: kiểm tra giữa các keyframe, đặc biệt những đoạn keyframe cách xa nhau hoặc xe thay đổi hướng/chuyển động.

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                               | Giá trị                                                        |
| ------------------------------------------------------ | ---------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | c25c15c80ba79c4ec885476f36184b4b86bad8f4975ae12c7551c4cc92c5f9cf |
| Thời điểm khóa                                     | `2026-09-15T10:32:10.841857+00:00`                             |
| Số row / frame / track trước khi mở reference      | `631/190/8`                                                    |

|               |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP |    FN |  IDSW |
| ------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | ----: | ----: |
| Bản pre-gold | 0.847 | 0.832 | 0.865 | 0.893 | 0.974 | 0.948 | 0.881 | 20 |    10 |     0 |
| Sau rework    | 0.807 | 0.789 | 0.829 | 0.885 | 0.949 | 0.892 | 0.875 | 60 | `2` | `0` |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** 

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi             | Frame   | ID | Đã sửa thế nào                                 |
| ---------------------- | ------- | -- | --------------------------------------------------- |
| Bbox treo / bbox thừa | 79–100 | 6  | Đặt`outside` đúng thời điểm xe rời khung |
| Bbox treo / bbox thừa | 63–78  | 5  | Đặt`outside` đúng thời điểm xe rời khung  |
| Bbox trôi             | 55      | 4  | Thêm keyframe quanh frame 55                       |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                 |
| ---------------------------------- | ----------------------------------------- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker              | yolo26n.pt / ByteTrack / BoT-SORT + ReID  |
| conf / IoU / imgsz / classes       | 0.25 / 0.70 / 960 /**[2, 5, 7]**    |
| device                             | `0` (GPU)                               |

| So sánh                  |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| bạn vs gold              | 0.807 | 0.789 | 0.829 | 0.885 | 0.949 | 0.892 | 0.875 | 60 |  2 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 |    2 |
| BoT-SORT + ReID vs gold   | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 |    2 |
| ReID vs bạn              | 0.778 | 0.720 | 0.841 | 0.923 | 0.868 | 0.739 | 0.919 | 85 | 78 |    2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi là `0.892`, thấp hơn IDF1 0.949. Khoảng cách này cho thấy chất lượng annotation về ID rất tốt, đồng thời vẫn còn một số lỗi FP/FN.MOTA chủ yếu tổng hợp FP, FN và ID switch, trong khi mỗi ID switch chỉ được tính một lần.Vì vậy nếu một track bị tách thành nhiều ID thì MOTA không phạt mạnh theo toàn bộ quãng đời bị sai. Ngược lại, IDF1 và AssA đánh giá việc duy trì đúng danh tính trên toàn bộ quãng đời của track nên nhạy hơn với lỗi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, BoT-SORT + ReID cải thiện IDF1 từ `0.875` lên `0.900` và AssA từ `0.776` lên `0.820`. Tuy nhiên IDSW không thay đổi, đều bằng `2`. ByteTrack có ID switch ở frame `59` và `94`; ReID có ID switch ở frame `87` và `113`. Như vậy treatment có khả năng duy trì association tốt hơn trên tổng thể, nhưng không loại bỏ hoàn toàn ID switch.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Từ ByteTrack sang BoT-SORT + ReID, DetA tăng từ `0.649` lên `0.711`, FN giảm mạnh từ `54` xuống `26`, trong khi FP tăng nhẹ từ `88` lên `1`. Điều này cho thấy treatment bắt được nhiều xe hơn và giảm bỏ sót, nhưng đồng thời tạo thêm một số bbox dư.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

các frame model-only của ReID, đặc biệt nhóm `104–106`. Ở các frame này ReID có bbox nhưng annotation không có bbox tương ứng.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame `113` là một điểm cần xem lại. ReID đổi track gold 6 từ ID `24` sang ID `31`, tạo một ID switch. Điều này cho thấy đoạn xe/track này có thể gây khó cho association. Tuy nhiên vì gold xác định track đó là cùng một đối tượng, bằng chứng hiện có nghiêng về việc ReID bị đổi ID chứ không phải annotation sai.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Khi xe bị che rồi xuất hiện lại: luôn giữ `track_id`cũ nếu có đủ bằng chứng đó là cùng xe.

Khi xe rời khung: phải dùng `outside` ngay tại frame thích hợp, tránh bbox treo

Các ca mơ hồ như xe rất nhỏ, bị che nhiều hoặc nhiều xe cắt nhau phải ghi lại quyết định và lý do

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
