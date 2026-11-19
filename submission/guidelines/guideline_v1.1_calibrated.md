# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603274 - Trần Thu Huyền

> Bản này ghi các bổ sung dựa trên ví dụ model và nhãn đã export. Những quy tắc phân lớp vốn đã có trong v1.0 được nhắc lại như bước kiểm tra, không thay thế label contract gốc.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 5 | Kính/cửa sổ xe thuộc mask xe hay không? | Phần kính, khung và cấu trúc xe nhìn thấy thuộc mask xe; người nhìn xuyên qua hoặc ngồi trong xe vẫn là instance `person` riêng, không gộp vào mask xe. Dùng click âm/chỉnh mask nếu SAM dính người vào xe. | img_013 | CAL-02 |

Các câu hỏi 1–4, 6–7 của v1.0 chưa được giải quyết bằng các exports hiện có; cần ca ảnh phù hợp trước khi chốt thay vì đặt quy tắc không có bằng chứng.

## 2. Ca khó mới

| Ảnh | Tình huống | Máy/nhãn hiện có | Quyết định | Bổ sung |
|---|---|---|---|---|
| img_013 | Tuk-tuk bị nhận thành car; box YOLO26 ví dụ có conf 0.42 | YOLO26 sai lớp; SAM chỉ tạo mask từ box đầu vào | Dựa vào loại phương tiện/số bánh để gán `three_wheeler`; không hy vọng đổi confidence sẽ sửa nhầm lớp. | Mục 2: thêm lưu ý lỗi lớp cần kiểm semantic, không chỉ threshold. CAL-01 |
| img_013 | Người nhìn thấy trong khoang tuk-tuk bị mask xe chồng lên | Mask SAM phủ một phần người trong khoang | Mask xe theo cấu trúc xe; người nhìn thấy là nhãn `person` riêng, tách bằng negative click hoặc chỉnh mask. | Mục 4.2.4: làm rõ instance xe/người. CAL-02 |
| img_023 | Annotation #267 `bus` cao 15.1 px | Export có `can_review`; chiều cao nhỏ hơn ngưỡng 20 px | Không giữ nhãn riêng cho vật nhỏ hơn 20 px; chỉ dùng `ignore_region` cho cụm ít nhất 5 vật nhỏ sát nhau. | Mục 4.1.6: nhấn mạnh áp dụng ngưỡng tối thiểu nhất quán. CAL-03 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản label contract gốc | — |
| v1.1 | Làm rõ lỗi nhầm lớp không sửa bằng threshold; tách người khỏi mask xe; nhắc thực thi nhất quán ngưỡng box 20 px. | CAL-01, CAL-02, CAL-03 |
