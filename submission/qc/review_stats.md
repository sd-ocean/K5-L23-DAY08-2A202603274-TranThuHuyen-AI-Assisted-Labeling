# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 487; đối tượng sau duyệt: 467; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 328 | 328/487 = 67.4% | 487 box máy |
| edited (sửa) | 87 | 87/487 = 17.9% | 487 box máy |
| deleted (xoá) | 72 | 72/487 = 14.8% | 487 box máy |
| added (vẽ thêm) | 52 | 52/467 = 11.1% | 467 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 42.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 198 | 28 | 22 | 12 |
| bicycle | 4 | 2 | 4 | 2 |
| motorbike | 78 | 40 | 10 | 13 |
| car | 40 | 6 | 8 | 16 |
| bus | 2 | 0 | 1 | 2 |
| truck | 0 | 0 | 3 | 3 |
| three_wheeler | 6 | 11 | 24 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 46 | 17 | 10 | 73.0% (KTC 95%: 61–82%) | 82.1% (KTC 95%: 70–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - missing: 1/3
  - extra: 3/3
  - wrong_class: 3/3
  - lọt: img_005.jpg (missing), img_009.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
