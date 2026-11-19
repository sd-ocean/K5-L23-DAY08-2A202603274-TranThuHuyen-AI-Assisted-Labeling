# Thông tin học viên

- Họ và tên: Trần Thu Huyền
- MSHV: 2A202603274
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: @sd-ocean
- Repo bài nộp (URL): https://github.com/sd-ocean/K5-L23-DAY08-2A202603274-TranThuHuyen-AI-Assisted-Labeling
- Commit chốt bài (hash): Chưa có; các thay đổi hiện tại chưa được commit
- Máy: Windows local; inference trên Google Colab GPU Tesla T4
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt

- Máy: accepted 328/487 = 67.4%; edited 87/487 = 17.9%; deleted 72/487 = 14.8%; added 52/467 = 11.1%.
- So với gold: precision 46/63 = 73.0%; recall 46/56 = 82.1% (4 ảnh gold).
- Bắt bẫy: 10/12 = 83.3%.
- Thời gian: gán tay 27/4 = 6.75 phút/ảnh; duyệt có pre-label 100/24 = 4.17 phút/ảnh.
- QC: 0/6 ảnh có lỗi, 0/101 đối tượng lỗi; cận trên Wilson 95% của tỷ lệ ảnh lỗi là 39%.
- **Quyết định:** Release theo QC hiện có; mẫu nhỏ không chứng minh được tỷ lệ lỗi thật dưới ngưỡng 2%.
