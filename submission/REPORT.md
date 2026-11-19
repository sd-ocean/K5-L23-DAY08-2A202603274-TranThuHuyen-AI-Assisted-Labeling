# Báo cáo Day 08 — 2A202603274 - Trần Thu Huyền

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng dùng khi merge | Căn cứ |
|---|---|---:|---|
| person | YOLO26m | 0.3 mặc định | Không có ngưỡng <= 0.5 đạt recall mục tiêu 90% trên gold; recall tối đa 75% với 24 vật gold. |
| bicycle | YOLO26m | 0.2 | Đạt recall 100%, precision 83% trên 5 vật gold. |
| motorbike | YOLO26m | 0.35 | Đạt recall 91%, precision 91% trên 11 vật gold. |
| car | YOLO26m | 0.3 mặc định | Không có ngưỡng <= 0.5 đạt recall mục tiêu; recall tối đa 88% trên 8 vật gold. |
| bus | YOLO26m | 0.3 mặc định | Chỉ có 1 vật gold, không đủ mẫu để chọn ngưỡng riêng. |
| truck | YOLO26m | 0.3 mặc định | Không có vật truck trong gold, không đủ mẫu để chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | Đạt recall 100%, precision 78% trên 7 vật gold. |

Ngưỡng của các lớp không có giá trị riêng được đặt theo `--conf 0.3`. Kết quả sweep chỉ dùng 4 ảnh gold; precision/recall tại ngưỡng được chọn là số đo in-sample, không phải ước lượng độc lập trên dữ liệu mới.

Lệnh gộp đã dùng:

```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Sau khi áp ngưỡng và loại box chồng lặp, merge giữ **573 box**: person 288, motorbike 143, car 65, three_wheeler 52, bicycle 17, truck 4, bus 4.

Kết quả tour model trên Colab T4:

| Lần chạy | Split / tham số | Box | Thời gian |
|---|---|---:|---:|
| YOLO26m | pilot, conf 0.25 | 215 | 1.27 giây/ảnh |
| YOLO26m | pilot, conf 0.5 | 139 | 0.43 giây/ảnh |
| YOLOE-26s | pilot, three_wheeler, conf 0.25 | 8 | 0.22 giây/ảnh (sau 7.5 giây nạp model) |
| YOLO26m + SAM 2.1 base | 2 ảnh, conf 0.3 | 16 box có mask | 1.68 giây/ảnh |

Trên pilot, YOLO26m ở conf 0.25 ghi 131 person, 55 motorbike, 17 car, 8 bicycle, 3 truck và 1 bus; ở conf 0.5 ghi lần lượt 87, 32, 14, 3, 2 và 1. YOLOE tạo 8 box `three_wheeler` trên pilot. Trong hình kết quả đã chia sẻ, có box tuk-tuk được gán `three_wheeler` với điểm 0.93 và các box điểm 0.37 cần kiểm tra bằng ảnh gốc. Hình YOLO26 + SAM trên `img_013` cho thấy tuk-tuk có box `car` điểm 0.42; SAM tạo mask từ box nhưng không sửa được nhãn lớp đầu vào.

**SAM đã chọn:** SAM 2.1 base (`sam2.1_b.pt`). Đây là phiên bản chạy được trên Colab T4; SAM chỉ tạo mask theo prompt hình học, không tự quyết định lớp theo label contract.

**Kết luận từ kết quả:** YOLO26 phù hợp làm nguồn ứng viên cho các lớp COCO, nhưng không giải quyết được lớp xe ba bánh ngoài taxonomy COCO; YOLOE bổ sung `three_wheeler` nhưng các box vẫn cần kiểm bằng mắt. Tăng conf từ 0.25 lên 0.5 làm số box pilot giảm 215 xuống 139, nên ngưỡng cao giảm lượng box phải review nhưng cũng làm giảm độ phủ. Ví dụ trong hình tour: YOLO26 gán tuk-tuk thành `car` (0.42), còn YOLOE có box `three_wheeler` 0.93 và một số box điểm thấp hơn cần xác minh. Điểm conf không thể thay thế việc kiểm đúng lớp/đúng vật.

## 2. AI error profile (từ `submission/qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---:|
| accepted | 328 | 328/487 = 67.4% | 487 box máy |
| edited | 87 | 87/487 = 17.9% | 487 box máy |
| deleted | 72 | 72/487 = 14.8% | 487 box máy |
| added | 52 | 52/467 = 11.1% | 467 đối tượng sau review |

Review stats loại 30 đối tượng quanh các bẫy khỏi profile để lỗi nhân tạo không bị tính thành lỗi model. Có 10 trường hợp đổi lớp trong deleted/added; ước tính 42 vật bị máy bỏ sót thật (`52 - 10`).

| Lớp | accepted | edited | deleted | added |
|---|---:|---:|---:|---:|
| person | 198 | 28 | 22 | 12 |
| bicycle | 4 | 2 | 4 | 2 |
| motorbike | 78 | 40 | 10 | 13 |
| car | 40 | 6 | 8 | 16 |
| bus | 2 | 0 | 1 | 2 |
| truck | 0 | 0 | 3 | 3 |
| three_wheeler | 6 | 11 | 24 | 4 |

So với gold, trên 4 ảnh và 56 đối tượng gold, máy đạt **precision 46/63 = 73.0%** và **recall 46/56 = 82.1%**. Khoảng tin cậy 95% được script báo lần lượt là 61–82% và 70–90%; mẫu nhỏ và việc chọn ngưỡng trên cùng gold làm kết quả lạc quan.

Trong batch, `three_wheeler` có 24 box bị xóa trong 41 box máy được chấm (58.5%); đây là số lượng xóa cao nhất theo lớp, nhưng cần đọc cùng 10 trường hợp đổi lớp và không diễn giải toàn bộ là tuk-tuk bị bỏ sót. `car` có số `added` cao nhất: 16/467 đối tượng sau review; tổng `added` là 52, trong đó thống kê tách 10 trường hợp đổi lớp và ước tính 42 vật máy bỏ sót thật. Đây là bằng chứng `accepted` đơn lẻ không đo được độ đầy đủ của nhãn.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Đã bắt **10/12 = 83.3%** bẫy, đạt ngưỡng tối thiểu 80%. Theo loại: loose 3/3, missing 1/3, extra 3/3, wrong_class 3/3. Hai bẫy bị lọt là `missing` ở `img_005.jpg` và `img_009.jpg`. KTC 95% cho tỷ lệ bắt bẫy là 55–95%.

Hai lỗi lọt đều là `missing`, tức vật không có box sẵn dễ bị bỏ qua hơn box sai/lỏng đang hiện trên màn hình. Từ kết quả này, các quy tắc kiểm chứng được cho lượt sau là: (1) ẩn box và quét toàn ảnh trước khi hiện nhãn ở mọi ảnh; (2) sau khi review box, quét lần hai các mép ảnh và vùng khuất để tìm vật không có box; (3) sau mỗi 10 ảnh nghỉ ngắn rồi tiếp tục theo đúng checklist, không dùng conf cao làm lý do bỏ qua kiểm tra. Vì chỉ có 12 bẫy, một vài trường hợp làm tỷ lệ thay đổi mạnh; cần đọc cả hai ảnh bị lọt chứ không chỉ nhìn phần trăm.

## 4. Máy có đáng dùng không?

Theo `submission/time_log.csv`, gán tay gold mất **27 phút/4 ảnh = 6.75 phút/ảnh**; review batch mất **100 phút/24 ảnh = 4.17 phút/ảnh**. So sánh thô là khoảng 38.3% ít thời gian hơn mỗi ảnh khi review batch, nhưng hai tập ảnh khác nhau và người gán đã quen cảnh sau khi làm gold nên đây không phải phép so sánh kiểm soát.

**Kết luận:** pre-label có ích như công cụ tăng tốc có điều kiện, không phải nhãn tự động có thể chấp nhận hàng loạt. Thời gian review đo được thấp hơn gán tay khoảng 38.3% mỗi ảnh, nhưng chỉ 67.4% box máy được giữ nguyên; 17.9% phải sửa và 14.8% bị xóa. Thêm vào đó, 52 đối tượng được thêm sau review cho thấy máy còn bỏ sót. Do gold chỉ có 4 ảnh và hai tập ảnh không giống nhau, kết quả thời gian/precision-recall chưa chứng minh mức tiết kiệm hay chất lượng sẽ lặp lại trên lô khác. Cách dùng phù hợp là để model đề xuất, sau đó người duyệt quét ảnh độc lập và sửa theo guideline. `time_log.csv` đã được chuẩn hóa tên bước `batch_review` và cột phút dạng số.

## 5. Mask SAM cho 3 vật (Part 5)

Các lệnh đã chạy dùng SAM 2.1 base, mỗi vật có 2 click dương và 1 click âm:

| Ảnh | SAM | Số click | Diện tích mask theo output | File |
|---|---|---:|---:|---|
| img_013 | SAM 2.1 base | 3 | 317,255 px (45.4% ảnh) | `masks/tuktuk_img013.jpg` |
| img_004 | SAM 2.1 base | 3 | 93,190 px (13.3% ảnh) | `masks/tuktuk_img004.jpg` |
| img_017 | SAM 2.1 base | 3 | 84,532 px (12.1% ảnh) | `masks/tuktuk_img017.jpg` |

Quan sát trực tiếp các ảnh kết quả: `img_013` có mask lớn (45.4% ảnh), bám mái và thân xe nhưng còn chồng lên một phần người trong khoang; `img_004` và `img_017` lần lượt phủ 13.3% và 12.1% ảnh, nhìn chung tách được hình xe nhưng vẫn cần kiểm các vùng khung hở/bánh xe. Kết quả cho thấy click âm chưa đảm bảo tự tách hoàn toàn người ngồi trong xe. Khi nhãn cần mask riêng, người và xe phải là hai instance; người nhìn thấy không nên bị gộp vào mask xe. SAM là đề xuất mask, không phải nhãn chuẩn để import mà không review.

## 6. Calibration (Part 6)

Đã lập 3 quyết định dựa trên bằng chứng trong lượt chạy:

| Case | Nguồn / ảnh | Vấn đề và quyết định |
|---|---|---|
| CAL-01 | `loi_may`, `img_013` | YOLO26 gán tuk-tuk thành `car` (ví dụ điểm 0.42 trong hình tour). Giữ lớp `three_wheeler` theo định nghĩa/số bánh; đổi ngưỡng confidence không sửa được lỗi phân loại. |
| CAL-02 | `cau_hoi_mo`, `img_013` | Mask SAM chồng lên người trong khoang xe. Bổ sung quy tắc mask xe theo phần cấu trúc xe nhìn thấy; người nhìn thấy là instance `person` riêng và cần click âm/chỉnh mask để tách khi cần. |
| CAL-03 | `can_review`, `img_023` | Annotation `#267` là `bus` cao 15.1 px, dưới mức 20 px. Không giữ nhãn riêng cho vật nhỏ hơn ngưỡng; chỉ dùng một `ignore_region` khi là cụm ít nhất 5 vật nhỏ sát nhau. |

Các quyết định đã được ghi vào `submission/qc/calibration_decisions.csv` và phần bổ sung cùng lịch sử thay đổi nằm trong `submission/guidelines/guideline_v1.1_calibrated.md`. Export cũ có 8 box dưới 20 px tại `img_018`, `img_023`, `img_027`; sau khi sửa và export lại, `validate_labels.py --all` xác nhận batch mới 24 ảnh/482 đối tượng không còn cảnh báo kích thước.

## 7. QC và quyết định (Part 6)

Đã tạo lại release candidate gồm **24 ảnh, 482 đối tượng** (`release_manifest.json`) từ export mới. Mẫu QC theo seed 42 gồm **6/24 ảnh, 101 đối tượng**: 1 ảnh có `can_review` (`img_022`) và 5 ảnh chọn ngẫu nhiên (`img_003`, `img_006`, `img_013`, `img_014`, `img_033`). Học viên xác nhận đã kiểm đủ 6 ảnh; phiếu ghi nhận **0/6 ảnh có lỗi** và **0/101 đối tượng lỗi**.

**Quyết định: Release (theo QC hiện có).** Export mới qua validation; mẫu không ghi nhận lỗi, và tỷ lệ bắt bẫy 10/12 = 83.3% đạt ngưỡng tối thiểu 80%. Tuy nhiên, với 0/6 ảnh lỗi, cận trên Wilson 95% vẫn là **39%**; mẫu nhỏ không chứng minh được tỷ lệ lỗi thật dưới ngưỡng 2%. Đây là quyết định dựa trên mẫu QC, không phải bảo đảm không còn lỗi trong toàn bộ batch.

Các lệnh đã chạy trên export mới:

```powershell
python scripts/build_release.py
python scripts/qc_sample.py --n 5 --seed 42 --force
```

Kết quả QC được ghi trong `submission/qc/qc_review.csv` và phép tổng hợp trong `submission/qc/qc_report_auto.md`. Nếu tiếp tục sửa nhãn sau QC, cần tạo lại release và mẫu, kiểm tra lại mẫu mới, rồi cập nhật kết luận này.

## 8. Ghi chú cá nhân

- **Dùng AI:** có dùng ChatGPT để giải thích quy trình repo, xử lý lỗi lệnh/Colab và hỗ trợ tổng hợp/phân tích bản nháp báo cáo, model log và calibration. Các số định lượng được đối chiếu với exports; các quyết định và hướng dẫn do AI đề xuất cần được người học xác nhận, giải thích được và chỉnh lại nếu khác với quan sát khi review.
- **Gold:** theo trình tự đã trao đổi, gold được gán và validate trước khi chạy các lượt model; cần xác nhận không xem pre-label hoặc hỏi AI về nội dung ảnh gold trước khi export.
- **Thiết bị:** inference chạy trên Google Colab GPU Tesla T4. Review nhãn thực hiện trên CVAT local. CVAT ban đầu gặp lỗi Docker/Traefik và thiếu label `motorbike` trong Task; sau khi cấu hình Project/Task, import và review đã hoàn tất.
- **Cache:** không dùng `data/cache/`; YOLO26, YOLOE và SAM được chạy trên Colab.
- **Vướng mắc kỹ thuật:** ban đầu gọi `python threshold_sweep.py` sai đường dẫn; lệnh đúng là `python scripts/threshold_sweep.py`. Colab cũng cần làm việc từ `/content/lab` để tìm thấy `scripts/`.
- **Điều sẽ làm khác lần sau:** học viên tự bổ sung sau khi hoàn tất QC và tổng kết quy trình.
