# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** Nhóm 01 **Thành viên:** 2A202602454 - TRẦN THANH THÁI

Detector cố định: `yolo26n.pt`, ảnh 640 px, lớp người, Re-ID `osnet_x0_25_msmt17.pt`. Chỉ thay tracker và ngưỡng detector `conf`, `iou`.

## 1. Cấu hình đã chọn

Bản nộp nằm trong `runs/nop_bai/`. Cả năm video chạy toàn bộ ảnh, không giới hạn 150 frame. Quan sát dưới đây dựa trên các frame lấy mẫu từ preview, không phải xác nhận chất lượng của mọi frame. Không dùng số ID sinh ra hoặc số hộp làm điểm chất lượng khi không có nhãn.

| Video | Tracker | conf | iou | Quan sát và lý do chọn | Đã thử nhưng không chọn |
|---|---|---|---|---|---|
| video_1 | botsort | 0.3 | 0.5 | Tại một số frame trong nhóm 50/75/100/125/150 có thêm hộp trên người ở xa so với ByteTrack. Chọn theo HOTA cao nhất trong các lượt full-frame đã chấm, không khẳng định tối ưu toàn bộ không gian tham số. | ByteTrack 0.3/0.5: HOTA 26.912, thấp hơn BoT-SORT 29.460, dù ít hộp giả và ít đổi ID hơn. |
| video_2 | bytetrack | 0.25 | 0.5 | Tại frame 50/75/100/125/150, cả ByteTrack và BoT-SORT có hộp trên nhiều người gần camera, vẫn bỏ sót nhóm đông phía xa. Giữ cấu hình ByteTrack vì chưa thấy lợi thế rõ của Re-ID trong mẫu đã xem; không kết luận nó chính xác hơn toàn video. | BoT-SORT 0.25/0.5 đã chạy đủ 1050 frame. Có thêm hộp tại một số vị trí nhưng chưa đủ bằng chứng giữ ID tốt hơn. |
| video_3 | ocsort | 0.3 | 0.45 | Cả OC-SORT và BoT-SORT đều có hộp trên người áo sọc gần camera tại frame 50/75/100/125/150. Giữ OC-SORT vì mẫu quan sát chưa cho thấy lợi thế rõ để đổi sang Re-ID. | BoT-SORT 0.3/0.45 đã chạy đủ 837 frame. Không khẳng định kém hơn trên toàn video. |
| video_4 | botsort | 0.35 | 0.5 | Trong các frame 50/100/150/300/500, hai tracker đều bám được người áo đỏ và áo trắng ở nhiều thời điểm. Giữ BoT-SORT làm lựa chọn có ngoại hình cho cảnh camera chuyển động, nhưng mẫu quan sát chưa chứng minh nó hơn ByteTrack hoặc phân biệt được bóng kính. | ByteTrack 0.35/0.5 đã chạy đủ 900 frame; chất lượng hộp trên người gần camera tương tự trong mẫu đã xem. |
| video_5 | botsort | 0.3 | 0.5 | Tại frame 500, BoT-SORT có hộp trên hai người gần cửa hàng trong khi ByteTrack không có. Một số frame trước cũng có thêm hộp trên người phía xa; chọn BoT-SORT dựa trên lợi thế bao phủ quan sát được này. | ByteTrack 0.3/0.5 đã chạy đủ 750 frame; bỏ sót hai người tại frame 500 trong ảnh đối chiếu. |

### Thực nghiệm và bằng chứng

- Cấu hình ban đầu: video_1–2 ByteTrack, video_3 OC-SORT, video_4–5 BoT-SORT. Bản video_1 ByteTrack được giữ tại `runs/sao_luu_video1_bytetrack/`.
- Đối chứng full-frame: `runs/doi_chung_20261008_224326/`. Video_1–3 thử BoT-SORT; video_4–5 thử ByteTrack. Mỗi video giữ nguyên conf/iou khi đổi tracker.
- Quét ngưỡng: `runs/quet_nguong_20261008_224918/`, 25 lượt, mỗi lượt 150 frame đầu. Tracker: ByteTrack cho video_1–2, OC-SORT cho video_3, BoT-SORT cho video_4–5.
- Mỗi video có năm cấu hình quanh mốc 0.3/0.5: conf 0.15/0.3/0.5 với iou=0.5; iou 0.4/0.5/0.7 với conf=0.3. Mỗi so sánh với mốc chỉ thay một tham số; không dùng bản 150 frame làm bản nộp.
- Video_1 còn chạy BoT-SORT đủ 600 frame ở conf=0.15 và 0.5, iou=0.5, rồi chấm cùng nhãn với cấu hình 0.3.
- Ảnh đối chiếu: `runs/kiem_tra_anh_20261008/`. So tracker ở video_1–3 dùng frame 50/75/100/125/150; video_4–5 dùng frame 50/100/150/300/500. So conf ở video_2–5 dùng frame 75/150.

### Nhận xét thử ngưỡng

- Video_1: ByteTrack trên 150 frame sinh 674/651/561 dòng kết quả với conf 0.15/0.3/0.5. Đây chỉ là số hộp đầu ra, không phải recall. Quyết định cuối dựa trên bảng chấm full-frame bên dưới.
- Video_2: tại frame 150, conf=0.5 bỏ hộp trên một số người phía xa mà hai ngưỡng thấp hơn còn giữ. Ba TXT ở conf=0.3, iou=0.4/0.5/0.7 giống hệt nhau; không có căn cứ xếp hạng ba ngưỡng IoU từ lượt thử này.
- Video_3: conf=0.15 thêm hộp nhỏ phía xa ở frame lấy mẫu; chưa xác nhận tất cả là người thật. Conf=0.5 giảm nhiều hộp đầu ra. Giữ cấu hình trung gian hiện có, không chọn chỉ vì số hộp nhiều hơn.
- Video_4: các ngưỡng conf đều có hộp trên người gần camera tại frame 75/150; khác biệt nằm ở người phía xa. Chưa thấy bằng chứng đủ mạnh để thay conf=0.35 của bản full-frame hiện có.
- Video_5: tại frame 75, conf=0.5 có ít hộp hơn trên nhóm người bên trái so với 0.15 và 0.3. Giữ 0.3 để tránh mất thêm người trong vùng tối, chưa khẳng định 0.15 tạo hộp giả nhiều hơn vì không có nhãn.

## 2. Số liệu video_1

Các kết quả đều chấm trên toàn bộ video_1. Chỉ video_1 có nhãn; không điền HOTA/MOTA/IDF1 cho video_2–5.

| Tracker | conf | iou | HOTA | MOTA | IDF1 | Bỏ sót (CLR_FN) | Hộp giả (CLR_FP) | Đổi ID (IDSW) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| ByteTrack | 0.3 | 0.5 | 26.912 | 17.292 | 25.713 | 15249 | 107 | 12 |
| **BoT-SORT — nộp** | **0.3** | **0.5** | **29.460** | **19.811** | **29.354** | **14538** | **337** | **25** |
| BoT-SORT | 0.15 | 0.5 | 29.343 | 20.731 | 29.561 | 14197 | 505 | 27 |
| BoT-SORT | 0.5 | 0.5 | 27.167 | 15.252 | 24.558 | 15508 | 229 | 10 |

Bảng trích kết quả TrackEval cho cấu hình nộp, run `nhom01_video1_botsort_224326`:

```text
Sequence   HOTA    MOTA    IDF1    CLR_Re  CLR_Pr  CLR_TP  CLR_FN  CLR_FP  IDSW
video_1    29.460  19.811  29.354  21.759  92.306  4043    14538   337     25
```

Ưu tiên HOTA vì cân bằng phát hiện và liên kết. Conf=0.15 có MOTA và IDF1 nhỉnh hơn nhưng HOTA thấp hơn 0.117 điểm và hộp giả nhiều hơn; không khẳng định chênh lệch nhỏ này có ý nghĩa thống kê. File video_1 nộp được sao chép từ đúng lượt BoT-SORT 0.3/0.5 đã chấm, kiểm tra hash trùng khớp.

## 3. Phân tích

### Video_1 — camera tĩnh, đánh giá định lượng

BoT-SORT 0.3/0.5 đạt HOTA 29.460, cao hơn ByteTrack 26.912 khi giữ nguyên detector và ngưỡng. Số bỏ sót giảm từ 15.249 xuống 14.538, nhưng hộp giả tăng từ 107 lên 337 và đổi ID tăng từ 12 lên 25. Vì vậy lựa chọn BoT-SORT là đánh đổi để cải thiện chất lượng tổng hợp, không phải kết luận giữ ID hoàn hảo. Recall của cấu hình nộp chỉ 21.759%, nên bỏ sót vẫn là hạn chế lớn cần xem lại. Camera tĩnh không tự động có nghĩa tracker chuyển động luôn tốt hơn Re-ID; kết quả thực nghiệm ở đây ủng hộ BoT-SORT theo tiêu chí HOTA.

### Video_5 — camera trên xe, đánh giá bằng mắt

Ảnh đối chiếu tại frame 500 cho thấy BoT-SORT có hộp trên hai người gần cửa hàng trong khi ByteTrack không có. Một số frame 50/100 cũng cho thấy BoT-SORT có thêm hộp ở nhóm người phía xa, nhưng chưa chứng minh mọi hộp thêm đều đúng. Với camera di chuyển và người nhỏ, việc giữ được hộp ở các vị trí đã quan sát là lý do chọn BoT-SORT thay vì ByteTrack. Re-ID cung cấp thông tin ngoại hình ngoài chuyển động, song thực nghiệm này không tách riêng tác dụng Re-ID khỏi các khác biệt khác giữa hai tracker. Không có nhãn và chỉ xem frame lấy mẫu nên không suy ra số lần đổi ID hoặc chất lượng toàn video từ các ảnh này.

## 4. Nếu có thêm thời gian

Nhóm sẽ xem liên tục các đoạn che khuất và lấy thêm mẫu ở cuối video để đánh giá độ bền ID, thay vì chỉ xem frame rời rạc. Sau đó quét conf/iou mịn hơn, giữ nguyên detector, kích thước ảnh và mô hình Re-ID theo quy định.
