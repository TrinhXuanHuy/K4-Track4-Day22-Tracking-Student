# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** Nhóm AI **Thành viên:** Trịnh Xuân Huy - 2A202602995


Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.15 | 0.5 | Bắt tốt người ở xa và giữ ID ổn định khi người đi bộ cắt ngang nhau nhờ kết hợp đặc trưng ngoại hình Re-ID; ít bị đứt đoạn tracklet | bytetrack (conf=0.3, iou=0.5: HOTA 26.91 thấp hơn, bỏ sót người ở xa); bytetrack (conf=0.15, iou=0.5: HOTA 27.31, MOTA 18.31 kém hơn BoT-SORT) |
| video_2 (phố đêm, tĩnh, rất đông) | bytetrack | 0.20 | 0.5 | Cảnh ban đêm rất đông đúc, độ tương phản kém và người che khuất lẫn nhau liên tục; cơ chế kết hợp 2 giai đoạn (low-confidence association) của ByteTrack giúp duy trì tracklet tốt, không bị phụ thuộc vào đặc trưng ngoại hình Re-ID vốn bị suy giảm nghiêm trọng trong bóng tối | botsort (conf=0.3, iou=0.5: chạy chậm hơn đáng kể trên 1050 frame đông đúc, Re-ID dễ nhầm lẫn giữa các bóng người tối màu) |
| video_3 (camera di động, ảnh nhỏ) | ocsort | 0.25 | 0.5 | Camera di chuyển, độ phân giải thấp và FPS chậm khiến vận tốc Kalman filter tuyến tính dễ bị sai lệch; OC-SORT sử dụng động lượng quan sát (OCM) và làm mượt trực tuyến (OCOS) giúp bù quán tính khi camera lia góc, giữ ID không bị văng hộp hay nhảy ID liên tục | bytetrack (conf=0.3, iou=0.5: khi camera lia nhanh hoặc FPS thấp, vận tốc tuyến tính của Kalman filter bị lệch làm mất dấu người) |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.35 | 0.5 | Camera tiến dần về phía trước làm kích thước người phóng to dần, có cửa kính gây phản chiếu bóng ảo; đặt conf=0.35 loại bỏ hiệu quả các hộp phát hiện giả trên kính, đồng thời BoT-SORT với Re-ID giúp nhận diện đúng danh tính người thật khi quy mô bounding box thay đổi | bytetrack (conf=0.2, iou=0.5: bị phát hiện nhiều hộp giả trên bề mặt phản chiếu của kính) |
| video_5 (trên xe bus, giao lộ đông) | ocsort | 0.25 | 0.5 | Xe bus rung lắc mạnh và đột ngột tại giao lộ đông người; OC-SORT với cơ chế Observation-Centric xử lý rất tốt các chuyển động giật cục, không bị phụ thuộc vào giả định vận tốc mượt mà, giúp track không bị gãy khi xe xóc nảy | bytetrack (conf=0.3, iou=0.5: khi xe bị rung xóc mạnh, hộp dự đoán bị văng lệch khỏi người thực tế gây nhảy ID) |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
Evaluating nhom01_video1_botsort

HOTA: nhom01_video1_botsort-pedestrian
                HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1         29.343    19.236    45.113    19.959    75.856    48.123    81.458    82.806    29.935    36.423    76.959    28.031    
COMBINED        29.343    19.236    45.113    19.959    75.856    48.123    81.458    82.806    29.935    36.423    76.959    28.031    

CLEAR: nhom01_video1_botsort-pedestrian
                MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag      
video_1         20.731    80.377    20.876    23.594    89.671    12.903    20.968    66.129    16.101    4384      14197     505       27        8         13        41        72        
COMBINED        20.731    80.377    20.876    23.594    89.671    12.903    20.968    66.129    16.101    4384      14197     505       27        8         13        41        72        

Identity: nhom01_video1_botsort-pedestrian
                IDF1      IDR       IDP       IDTP      IDFN      IDFP      
video_1         29.561    18.67     70.955    3469      15112     1420      
COMBINED        29.561    18.67     70.955    3469      15112     1420      

Count: nhom01_video1_botsort-pedestrian
                Dets      GT_Dets   IDs       GT_IDs    
video_1         4889      18581     55        62        
COMBINED        4889      18581     55        62        
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- **Phân tích video_1 (Quảng trường ngoài trời, camera tĩnh, ban ngày):** 
  BoT-SORT vượt trội rõ rệt so với ByteTrack ở cả 3 chỉ số chính: HOTA đạt 29.34 (so với 27.31 của ByteTrack), MOTA đạt 20.73 (so với 18.31), và IDF1 đạt 29.56 (so với 26.98). Trong điều kiện ánh sáng ngoài trời ban ngày rõ nét và camera tĩnh, mạng trích xuất đặc trưng Re-ID (OSNet) phát huy tối đa hiệu quả khi cung cấp các embedding ngoại hình ổn định. Nhờ đó, khi hai người đi bộ cắt ngang qua nhau hoặc bị che khuất tạm thời, BoT-SORT duy trì chính xác danh tính người mà không bị hoán đổi ID như các tracker thuần chuyển động. Ngưỡng `conf=0.15` giúp phát hiện thêm nhiều người đi bộ ở khoảng cách xa (nâng CLR_TP từ 3534 lên 4384), trong khi Re-ID giữ cho các tracklet này không bị gán nhầm lẫn.

- **Phân tích video_2 (Phố đêm, camera tĩnh trên cao, rất đông người):**
  Trong bối cảnh ban đêm, ánh sáng yếu và độ tương phản thấp khiến đặc trưng ngoại hình Re-ID bị suy biến nghiêm trọng, dễ dẫn đến so khớp sai danh tính nếu dựa vào ngoại hình. Ngược lại, ByteTrack với thuật toán liên kết 2 giai đoạn (first associate high-conf boxes, then associate low-conf boxes) hoạt động cực kỳ hiệu quả trong cảnh đông đúc. Các hộp phát hiện có độ tin cậy thấp do bóng tối hay che khuất một phần vẫn được gom chính xác vào quỹ đạo hiện có mà không bị bỏ sót. Quan sát video thực tế cho thấy các hộp bao bám sát từng người đi bộ từ trên cao, hiện tượng nhấp nháy mất dấu giảm hẳn và tốc độ xử lý nhanh hơn rất nhiều so với các tracker có Re-ID.

- **Phân tích video_5 (Trên xe bus, rung lắc mạnh, giao lộ đông):**
  Video quay từ xe bus có đặc thù rung lắc cơ học và chuyển động giật cục liên tục khi xe di chuyển và dừng đỗ. Mô hình chuyển động vận tốc không đổi (constant velocity) của Kalman filter truyền thống (trong ByteTrack hay SORT) bị sai lệch nghiêm trọng mỗi khi có cú xóc, khiến bounding box dự đoán bị văng lệch khỏi vị trí người thực tế. OC-SORT giải quyết triệt để vấn đề này nhờ cơ chế Observation-Centric Momentum (OCM), sử dụng trực tiếp độ dời của các hộp quan sát thực tế để định hướng cập nhật trạng thái thay vì tin tưởng mù quáng vào dự đoán nội suy. Nhờ vậy, ngay cả khi xe bus bị rung nảy mạnh tại ngã tư, ID của người đi bộ bên đường vẫn được duy trì liên tục và không bị gãy quỹ đạo.

## 4. Nếu có thêm thời gian

- Thử nghiệm quét mịn hơn dải ngưỡng detector `conf` (bước nhảy 0.02) kết hợp tinh chỉnh các siêu tham số bên trong tracker (như khoảng thời gian duy trì tracklet bị mất `track_buffer`, ngưỡng khoảng cách IoU và Re-ID matching threshold).
- Tích hợp thêm các mô hình Re-ID chuyên dụng cho môi trường thiếu sáng hoặc góc nhìn camera từ trên cao (top-down view) để cải thiện độ chính xác cho video ban đêm (`video_2`).
- Phân tích chi tiết các frame gây lỗi ID Switch hoặc False Negative bằng công cụ trực quan hóa frame-by-frame để xác định chính xác nguyên nhân (do detector bỏ sót hay do tracker liên kết sai).
