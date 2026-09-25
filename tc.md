# TỔNG HỢP TESTCASE KIỂM THỬ TỰ ĐỘNG (EXIMBANK POC AUTOMATION TEST)

---

## 1. TỔNG HỢP SỐ LƯỢNG TESTCASE THEO HỆ THỐNG

| Hệ thống | Nghiệp vụ | Số lượng Testcase | Sheet chi tiết |
| :--- | :--- | :---: | :--- |
| Ebiz | NHS-Doanh nghiệp | 1 | EBiz_chuyen_tien_lô_Ngoai |
| Edigi | NHS-Cá nhân | 10 | EDigi _ TT vé máy bay SSMedia, EDigi _ Quên mật khẩu, EDigi_Đăng ký dịch vụ thẻ, EDigi_Mở tài khoản số đẹp |
| Teller App | Thẻ | 1 | Thẻ |
| Teller App | Tiền gửi | 8 | BPM |
| BPM | BPM | 1 | BPM |
| Fincore | Vay | 4 | ASSET, LIAB |
| **TỔNG CỘNG** | | **25** | |

---

## 2. HỆ THỐNG EBIZ: CHUYỂN TIỀN THEO LÔ NGOÀI HỆ THỐNG

### Testcase ID: EBIZ_1 - Chuyển tiền theo lô ngoài hệ thống trên EBIZ
- **Nội dung thực hiện:** Khởi tạo và duyệt lệnh cấp 1 ở IB, Duyệt lệnh cấp 2 ở MB iOS

| Bước | Mô tả thực hiện | Kết quả mong muốn |
| :---: | :--- | :--- |
| 1 | **User khởi tạo giao dịch : Đăng nhập vào hệ thống IB EBiz**<br>+ Truy cập link UAT EBiz : http://10.128.10.85:8080/KHDN/account/login-corp<br>+ Nhập "Tên đăng nhập"<br>+ Nhập "Mật khẩu"<br>+ Nhập mã Captcha (nếu có)<br>+ "Đăng nhập".<br>+ "Đồng ý" popup "Quý khách chưa kích hoạt phương thức xác thực Smart OTP..." | Đăng nhập thành công vào hệ thống IB EBiz. |
| 2 | **User khởi tạo giao dịch : Chọn menu Chuyển tiền / Chuyển tiền theo lô**<br>+ "Chuyển tiền"<br>+ "Chuyển tiền theo lô" | Hiển thị màn hình nhập thông tin Chuyển tiền theo lô. |
| 3 | **User khởi tạo giao dịch : Nhập thông tin chuyển tiền**<br>+ Thông tin tài khoản chuyển : chọn một tài khoản<br>+ Phạm vi chuyển : "Ngân hàng khác"<br>+ Tên file: chọn đường dẫn file cần thực hiện chuyển tiền<br>+ Diễn giải : nhập nội dung diễn giải<br>+ Phí được thanh toán bởi : chọn 1 nguồn thu phí<br>+ "Tiếp tục" | Dữ liệu được ghi nhận, chuyển sang bước kiểm tra thông tin. |
| 4 | **3.1 Trường hợp nội dung file đúng thông tin:**<br>+ Kiểm tra lại thông tin trên màn hình và thông tin đã nhập ở bước 3 và thông tin trong file đã upload<br>+ "Xác nhận"<br>+ "Nhật ký giao dịch" : kiểm tra cấp duyệt kế tiếp | Khởi tạo giao dịch thành công → xem được cấp duyệt kế tiếp để lấy user đăng nhập vào duyệt ở bước 4. |
| 5 | **3.2 Trường hợp nội dung file có thông tin không đúng:**<br>+ Popup màn hình Thông tin danh sách lỗi, yêu cầu Tải về, sau đó mới Tiếp tục.<br>+ "Tải về"<br>+ "Tiếp tục"<br>+ Sau đó "Xác nhận" và "Nhật ký giao dịch" để xem cấp duyệt kế tiếp như bước 3.1 | Khởi tạo giao dịch thành công → xem được cấp duyệt kế tiếp để lấy user đăng nhập vào duyệt ở bước 4. |
| 6 | **User duyệt cấp 1 : Tiến hành đăng nhập hệ thống để duyệt lệnh (như bước 1)**<br>+ "Giao dịch chờ duyệt"<br>+ Bấm vào hyperlink giao dịch cần duyệt<br>+ "Duyệt giao dịch" | Hiển thị chi tiết giao dịch chờ duyệt và form xác thực OTP. |
| 7 | **User duyệt cấp 1 : Xác thực giao dịch**<br>+ "Lấy OTP"<br>+ Nhập Mã xác thực (tùy theo đang áp dụng hệ thống OTP nào mà sẽ lấy mã ở hệ thống đó)<br>+ "Duyệt giao dịch"<br>+ "Nhật ký giao dịch" : kiểm tra thông tin cấp duyệt kế tiếp | Hệ thống báo duyệt lệnh thành công, vào Nhật ký giao dịch xem được cấp duyệt kế tiếp để lấy user đăng nhập vào duyệt ở bước 6. |
| 8 | **User duyệt cấp 2 : Tiến hành đăng nhập hệ thống MB để tìm duyệt lệnh**<br>+ Mở app Ebiz UAT trên iOS<br>+ "Đăng nhập"<br>+ Nhập "Tên đăng nhập"<br>+ Nhập "Mật khẩu"<br>+ "Đăng nhập".<br>+ "Đóng" popup "Quý khách chưa kích hoạt tính năng xác thực Smart OTP..." | Đăng nhập thành công vào app Ebiz UAT trên iOS. |
| 9 | **User duyệt cấp 2 : Tìm giao dịch cần duyệt và tiến hành duyệt lệnh**<br>+ "Giao dịch chờ duyệt"<br>+ Tìm lệnh cần duyệt: "Duyệt"<br>+ "Lấy OTP"<br>+ "Đóng"<br>+ Nhập Mã xác thực (tùy theo đang áp dụng hệ thống OTP nào mà sẽ lấy mã ở hệ thống đó)<br>+ Hệ thống tự động duyệt giao dịch khi nhập đúng mã xác thực<br>+ "Nhật ký" : kiểm tra lại trạng thái giao dịch đã được duyệt thành công với trạng thái Đã duyệt. | Giao dịch được duyệt thành công, trạng thái hiển thị "Đã duyệt". |

---

## 3. HỆ THỐNG EDIGI

### Testcase ID: EDIGI_2 - Quên mật khẩu trên EDigi
- **Chương trình thực hiện:** Kênh Internet Banking (IB)
- **Đầu vào / Kiểm tra Backend:** Gọi kiểm tra thông tin Khách hàng nhập tại Backend EDigi/ Khách hàng/Danh sách khách hàng
- **Các bước thực hiện:**
  1. Chọn "Quên mật khẩu" tại màn hình đăng nhập.
  2. Nhập các thông tin (Xác nhận thông tin):
     - Họ và tên
     - Tên đăng nhập
     - Nhận mật khẩu mới qua: Số điện thoại hoặc email (chọn 1 trong 2)
     - Số điện thoại
     - Phương thức xác thực: SMS, Smart OTP (chọn 1 trong 2)
     - Nhấn "Xác nhận".
  3. Nhập OTP và nhấn xác nhận.
  4. Kiểm tra email hoặc SMS để xác nhận thành công.
- **Kết quả mong muốn:** Nhận mật khẩu mới qua SMS hoặc Email.

---

### Testcase ID: EDIGI_1 - Thanh toán vé máy bay SSMedia trên EDigi
- **Chương trình thực hiện:** App EDigi Mobile
- **Điều kiện tiên quyết:** Máy đã thực hiện đăng nhập vào app trước đó để không phải thực hiện quét face những lần sau.

| Giai đoạn | Thao tác thực hiện | Kết quả mong muốn |
| :--- | :--- | :--- |
| **Truy cập dịch vụ** | + Truy cập app Edigi trên điện thoại<br>+ Chọn "Đăng nhập"<br>+ Nhập "Mật khẩu"<br>+ Chọn "Đăng nhập"<br>+ Chọn "Mua vé máy bay" | Vào được màn hình dịch vụ mua vé máy bay SSMedia. |
| **Khởi tạo giao dịch** | + Chọn mục "Điểm đi" → Hiển thị danh sách điểm đi → Chọn điểm đi bất kỳ<br>+ Chọn mục "Điểm đến" → Hiển thị danh sách điểm đến → Chọn điểm đến bất kỳ<br>+ Chọn "Tìm chuyến bay" → Hiển thị danh sách chuyến bay → Chọn chuyến bay bất kỳ<br>+ Hiển thị màn hình "Tóm tắt đặt chỗ" → Chọn "Tiếp tục"<br>+ Hiển thị màn Thông tin khách hàng → Kéo xuống dưới cùng và chọn "Tiếp tục"<br>+ Hiển thị màn Xác nhận thanh toán → Chọn "Thanh toán" | Hệ thống chuyển tiếp qua các màn hình và hiển thị đúng thông tin đặt chỗ. |
| **Thanh toán giao dịch** | + Hiển thị màn hình thông tin giao dịch<br>+ Chọn "Thanh toán"<br>+ Hiển thị màn hình xác nhận giao dịch<br>+ Nhập mã OTP "123456"<br>+ Chọn "Xác nhận"<br>+ Hiển thị màn hình xác nhận giao dịch thành công | Giao dịch thanh toán thành công, hiển thị biên lai/màn hình kết quả. |
| **Kiểm tra nhật ký giao dịch** | + Chọn "Tra cứu"<br>+ Chọn "Nhật ký giao dịch"<br>+ Hiển thị màn Nhật ký giao dịch<br>+ Chọn giao dịch đầu tiên (Trạng thái giao dịch thành công)<br>+ Hiển thị màn thông tin chi tiết giao dịch | Chi tiết giao dịch thể hiện trạng thái "Thành công". |
| **Kiểm tra báo cáo trên Backend** | + Truy cập BE UAT: `http://10.1.51.118:9090/auth/login?returnUrl=`<br>+ Nhập "Tên đăng nhập", "Mật khẩu" → "Đăng nhập"<br>+ Trên thanh menu chọn "Báo cáo" -> "Báo cáo dịch vụ tài chính" -> "Báo cáo giao dịch thanh toán dịch vụ SSMedia"<br>+ Nhập "Mã giao dịch core" → Chọn "Tìm kiếm"<br>+ Hiển thị giao dịch cần tìm | Kiểm tra trạng thái giao dịch ghi nhận "Hạch toán thành công". |

---

### Nhóm Testcase Đăng ký dịch vụ thẻ (EDIGI_3, EDIGI_4, EDIGI_5)
- **Môi trường thực hiện:** HĐH iOS, App MB
- **Các bước thực hiện chung:**
  1. Đăng nhập vào ứng dụng.
  2. Chọn chức năng thẻ trên màn hình chính.
  3. Chọn dịch vụ thẻ tương ứng cần đăng ký.
  4. Xác nhận thông tin và nhập mã OTP.

| Mã Testcase | Đối tượng kiểm thử | Đầu vào thẻ | Dịch vụ đăng ký | Kết quả mong muốn |
| :--- | :--- | :--- | :--- | :--- |
| **EDIGI_3** | Đăng ký dịch vụ thẻ trên EDigi | Thẻ quốc tế | Đăng ký dịch vụ SMS Alert | Đăng ký thành công dịch vụ SMS Alert cho thẻ quốc tế. |
| **EDIGI_4** | Đăng ký dịch vụ thẻ trên EDigi | Thẻ quốc tế | Đăng ký dịch vụ 3D Secure | Đăng ký thành công dịch vụ 3D Secure cho thẻ quốc tế. |
| **EDIGI_5** | Đăng ký dịch vụ thẻ trên EDigi | Thẻ nội địa | Đăng ký dịch vụ SMS Alert | Đăng ký thành công dịch vụ SMS Alert cho thẻ nội địa. |

---

### Nhóm Testcase Mở tài khoản số đẹp (EDIGI_6 đến EDIGI_10)
- **Môi trường thực hiện:** HĐH Android, App MB

#### EDIGI_6: Mở tài khoản số đẹp LUỒNG SỐ TỰ CHỌN
- **Các bước thực hiện:**
  1. Đăng nhập hệ thống: Mở App MB, nhập User ID & Password => Đăng nhập => Nhập OTP => Nhấn Xác nhận.
  2. Mở tài khoản: Chọn chức năng Tài khoản => Chọn Mở tài khoản số đẹp => Chọn độ dài của tài khoản (8, 9, 10, 12 số) => Nhấn Tiếp tục.
  3. Chọn loại tài khoản: Nhập số tự chọn muốn mở > Click Tìm kiếm => Chọn vào tài khoản => Click chọn điều kiện, điều khoản => Nhấn Tiếp tục => Nhập mã xác thực => Nhấn Xác nhận.
- **Kết quả mong muốn:** Mở tài khoản số đẹp thành công.

#### EDIGI_7: Mở tài khoản số đẹp LUỒNG TỪ KHO SỐ CỦA NGÂN HÀNG
- **Các bước thực hiện:**
  1. Đăng nhập hệ thống: Mở App MB, nhập User ID & Password => Đăng nhập => Nhập OTP => Nhấn Xác nhận.
  2. Mở tài khoản: Chọn chức năng Tài khoản => Chọn Mở tài khoản số đẹp => Chọn độ dài của tài khoản (8, 9, 10, 12 số) => Nhấn Tiếp tục.
  3. Chọn loại tài khoản: Chọn từ kho số đẹp có sẵn (số lặp, số soi gương, số hỗn hợp, số tiến) > Chọn số tài khoản muốn mở => Click chọn điều kiện, điều khoản => Nhấn Tiếp tục => Nhập mã xác thực => Nhấn Xác nhận.
- **Kết quả mong muốn:** Mở tài khoản số đẹp thành công.

#### EDIGI_8: Mở tài khoản số đẹp khi còn nợ phí số đẹp
- **Các bước thực hiện:**
  1. Đăng nhập hệ thống: Mở App MB, nhập User ID & Password => Đăng nhập => Nhập OTP => Nhấn Xác nhận.
  2. Mở tài khoản: Chọn chức năng Tài khoản => Chọn Mở tài khoản số đẹp.
- **Kết quả mong muốn:** Hệ thống hiển thị popup báo lỗi nợ phí, không cho phép tiếp tục giao dịch.

#### EDIGI_9: Mở tài khoản số đẹp khi chưa định danh tài khoản số đẹp đã mở trước đó
- **Các bước thực hiện:**
  1. Đăng nhập hệ thống: Mở App MB, nhập User ID & Password => Đăng nhập.
  2. Chọn chức năng Mở tài khoản số đẹp.
- **Kết quả mong muốn:** Hệ thống cảnh báo yêu cầu hoàn tất định danh tài khoản cũ trước khi mở thêm tài khoản mới.

#### EDIGI_10: Mở tài khoản số đẹp luồng số tự chọn, chọn số tài khoản đã tồn tại
- **Các bước thực hiện:**
  1. Đăng nhập hệ thống: Mở App MB, nhập User ID & Password => Đăng nhập.
  2. Chọn chức năng Mở tài khoản số đẹp > Luồng tự chọn.
  3. Nhập số tài khoản đã tồn tại trong hệ thống.
- **Kết quả mong muốn:** Hệ thống báo số tài khoản đã có người sở hữu / không khả dụng.

---

## 4. HỆ THỐNG TELLER APP: THẺ

### Testcase ID: CARD_TA_1 - Tạo mới hồ sơ thẻ Debit – thẻ chính
- **Môi trường:** Teller App UAT (`https://10.1.45.88:8444/web/guest/desktop`)
- **Quy trình thực hiện:**
  - **Maker thực hiện:**
    1. Đăng nhập hệ thống Teller App UAT.
    2. Chọn mục Quản lý thẻ -> Tạo mới hồ sơ thẻ -> Thẻ chính.
    3. Nhập mã khách hàng và chọn Thẻ Debit -> Nhập các thông tin bắt buộc (có dấu *).
    4. Gửi duyệt.
  - **Checker thực hiện:**
    1. Đăng nhập hệ thống Teller App.
    2. Chọn mục Quản lý thẻ -> Tìm kiếm hồ sơ thẻ -> Thẻ chính.
    3. Nhập mã khách hàng và chọn trạng thái Chờ duyệt.
    4. Thực hiện duyệt hồ sơ.
- **Kết quả mong muốn:** Tạo hồ sơ thẻ Debit thành công.

---

## 5. HỆ THỐNG BPM: LUỒNG CẤP PHÊ DUYỆT (CPD)

### Testcase ID: BPM_1 - Luồng CPD = Giám đốc ĐVKD
- **Môi trường:** BPM-UAT
- **Dữ liệu kiểm thử (Test Data):** CIF: `121347847`
- **Điều kiện tiên quyết:** Đăng nhập thành công, User đã được phân quyền đầy đủ, CIF Khách hàng hợp lệ.

| Bước | Vai trò | Mô tả chi tiết thực hiện | Kết quả mong muốn |
| :---: | :--- | :--- | :--- |
| 1 | **CB QHKH (RM_R)** | 1. Đăng nhập vào hệ thống.<br>2. Vào chức năng "Đề xuất cấp tín dụng".<br>3. Tìm kiếm khách hàng bằng số CIF: `121347847`.<br>4. Nhấn nút "Khởi tạo" trên dòng thông tin khách hàng. | 1. Đăng nhập thành công.<br>2. Hệ thống hiển thị màn hình tìm kiếm khách hàng.<br>3. Hệ thống trả về đúng thông tin khách hàng.<br>4. Hệ thống chuyển sang màn hình "Nhập thông tin nhanh". |
| 2 | **CB QHKH (RM_R)** | 1. Tại màn hình "Nhập thông tin nhanh", nhập các trường bắt buộc (Nhóm sản phẩm, Sản phẩm, Số tiền CTD,...).<br>2. Nhấn nút "Hoàn thành". | 1. Dữ liệu được nhập hợp lệ.<br>2. Hệ thống lưu thông tin, khởi tạo mã giao dịch và luân chuyển hồ sơ đến bước tiếp theo.<br>3. Hồ sơ xuất hiện trong "My Work" / "Công việc của tôi" của vai trò CB QHKH (RM_R). |
| 3 | **CB QHKH (RM_R)** | 1. Đăng nhập vào hệ thống.<br>2. Mở hồ sơ từ "Công việc của tôi".<br>3. Tại màn hình "Nhập thông tin chi tiết", điền đầy đủ thông tin vào các tab.<br>4. Nhấn nút "Hoàn thành". | 1. Mở được hồ sơ với đúng mã giao dịch.<br>2. Dữ liệu được nhập hợp lệ.<br>3. Hệ thống luân chuyển hồ sơ đến bước Thẩm định tại ĐKVD. |
| 4 | **CB Thẩm định tại ĐVKD (AMBRN_R)** | 1. Đăng nhập vào hệ thống.<br>2. Mở hồ sơ từ "Công việc của tôi".<br>3. Thực hiện thẩm định, điền các ý kiến, nhận xét vào các trường thông tin.<br>4. Nhấn nút "Hoàn thành". | 1. Mở được hồ sơ.<br>2. Hệ thống ghi nhận ý kiến thẩm định.<br>3. Hồ sơ được luân chuyển đến bước Phê duyệt. |
| 5 | **Giám đốc ĐVKD (CABRN1)** | 1. Đăng nhập vào hệ thống.<br>2. Mở hồ sơ từ "Công việc của tôi".<br>3. Kiểm tra toàn bộ thông tin hồ sơ, đề xuất, ý kiến thẩm định.<br>4. Phê duyệt cấp tín dụng: Thiết lập `Ý kiến phê duyệt = Đồng ý cấp tín dụng`, `Phê duyệt chi tiết = Đồng ý theo đề xuất của đơn vị Thẩm định`.<br>5. Nhấn nút "Đồng ý". | 1. Mở được hồ sơ, chọn phê duyệt = Đồng ý.<br>2. Hệ thống ghi nhận kết quả phê duyệt là "Đồng ý".<br>3. Hồ sơ được luân chuyển về cho CB QHKH với trạng thái "Đã phê duyệt". |
| 6 | **CB QHKH (RM_R)** | 1. Đăng nhập, mở hồ sơ từ "Công việc của tôi".<br>2. Nhận kết quả phê duyệt "Đồng ý".<br>3. Nhấn nút "Hoàn thành" để kết thúc quy trình. | 1. Mở được hồ sơ.<br>2. Trạng thái hồ sơ trên hệ thống được cập nhật thành "Hoàn thành". Quy trình kết thúc thành công. |

---

## 6. HỆ THỐNG FINCORE: ASSET (HỢP ĐỒNG TÍN DỤNG)

### TTHD_OTC_001: Tạo HĐTD cha
- **Bước 1 (Maker thực hiện):**
  - Đăng nhập hệ thống: Nhập User ID và Password của TTV.
  - Nhập dữ liệu đầu vào (Tab LNM Details):
    - Function: Chọn Add; Limit ID * => hiển thị số hợp đồng tín dụng và sol của CN/PGD; Bấm Go.
    - Description *: Nhập nội dung của HĐTD.
    - CCY *: Chọn đơn vị tiền tệ (VND, USD).
    - Limit Type ID: Nhập mã khách hàng (CIF).
    - Approval Limit *: Nhập số tiền được phê duyệt theo tờ trình cấp tín dụng.
    - Drawing Power Indicator *: Chọn Deriver (HĐ có tài sản thế chấp) hoặc Equal (HĐ không có/có một phần tài sản thế chấp).
    - Limit Approval Date *: Ngày nhập HĐTD; Contract Sign Date: Ngày ký HĐTD; Limit Expiry Extended up to: Ngày gia hạn; Limit Review Date: Ngày đánh giá lại; Limit Expiry Date: Ngày hết hiệu lực HĐTD.
    - Approval Level: Chọn cấp phê duyệt.
    - Loan Type: Non Revolving (HĐ từng lần) hoặc Revolving (HĐ hạn mức).
    - Term Type: `00001` (Fixed date), `00002` (After credit contract date), `00003` (After disbursement date); Month/Day: Thời hạn vay; Maturity Date: Ngày đáo hạn; Product Code: 01PF.
  - Tab Limit Categories: Currency, CIF ID, Product: 01PF (add thêm các hình thức vay, bảo lãnh...).
  - Tab Basel: Tích chọn => Bấm Submit (chuyển duyệt).
- **Bước 2 (Checker duyệt):** Đăng nhập Checker => Function: Verify, Limit ID * => Tích chọn HĐTD đã tạo => Bấm Go => Kiểm tra và bấm Submit duyệt.
  - *Kết quả mong muốn:* Tạo số HĐTD dạng `LAV...`
- **Bước 3 (Truy vấn giao dịch):** Đăng nhập TTV => Function: Inquiry => Nhập Limit ID * đã tạo và duyệt => Bấm Go => Kiểm tra lại đầy đủ thông tin hợp đồng.

### TTHD_OTC_003: Tạo HĐTD con và liên kết với HĐ cha
- **Bước 1 (Maker thực hiện):** Đăng nhập TTV => Tab LNM Details: Function: Add => Limit ID * => Bấm Go.
  - Parent Limit ID: Nhập HĐTD cha đã tạo ở trên.
  - Drawing Power Indicator *: PARENT; Drawing Power Pcnt: Tỷ lệ vốn trên tổng hạn mức cha.
  - Nhập các thông tin hạn mức, thời hạn, loại cấp tín dụng, mã sản phẩm.
  - Tab Basel: Tích chọn => Submit chuyển duyệt.
- **Bước 2 (Checker duyệt):** Đăng nhập Checker => Function: Verify => Ref No: Nhập số Ref đã tạo => Bấm Go => Kiểm tra và Submit duyệt.
  - *Kết quả mong muốn:* Tạo số HĐTD con thành công dạng `LAV...`

### TTHD_OTC_004: Modify HĐTD cha/con
- **Bước 1 (Maker thực hiện):** Đăng nhập TTV => Tab LNM Details: Function: Modify => Limit ID * => Nhập số HĐTD và sol hiện hữu => Bấm Go.
  - Điều chỉnh các thông tin cho phép (Description, Approval Limit, Drawing Power Indicator, Sign Date, Expiry Date, Loan Type, Maturity Date...).
  - Submit chuyển duyệt.
- **Bước 2 (Checker duyệt):** Đăng nhập Checker => Function: Verify => Chọn hợp đồng đã điều chỉnh => Bấm Go => Kiểm tra thông tin điều chỉnh => Submit duyệt.
  - *Kết quả mong muốn:* HĐTD hiển thị đầy đủ thông tin đã điều chỉnh.

### TTHD_OTC_005: HSCLM - Thực hiện link tài sản đảm bảo với HĐTD
- **Thực hiện:** Đăng nhập hệ thống => Vào menu HSCLM (Link Tài sản thế chấp với HĐTD).
  - Function *: Chọn Link.
  - Limit ID: Nhập số HĐTD và mã sol CN.
  - Collateral ID *: Mã tài sản thế chấp đã tạo trong menu HCLM.
  - Nhấn Accept => Nhập Collateral Value, Apportioned Value, Loan to Value Pcnt. => Submit.
  - *Kết quả mong muốn:* Liên kết tài sản đảm bảo vào HĐTD thành công.

---

## 7. HỆ THỐNG FINCORE: LIAB (TIỀN GỬI & DỊCH VỤ QUẦY)

### DV_CT_TC001: Chuyển tiền cùng loại tiền
1. **Thao tác thực hiện:** Mở Edge truy cập UAT Finacle, TTV hạch toán chuyển tiền.
2. **In Ủy nhiệm chi:** TTV vào menu `HPR` > Click Go > Chọn dòng dữ liệu theo User ID hạch toán, Report Name: `"VOUCHER-số bút toán"`, Ngày thực hiện > Nhấn Print Screen. *(Kết quả: Hiển thị đúng số tiền, người chuyển, người nhận, nội dung).*
3. **Duyệt chuyển tiền:** KSV đăng nhập > Vào menu `HXFER` > Function: `P-Post` > Transaction ID: Nhập số giao dịch > Click Go > Kiểm tra và Submit duyệt. *(Kết quả: Báo duyệt thành công).*
4. **In phiếu hạch toán:** TTV vào menu `HPR` > Chọn Report Name: `"ACCOUNTING_SLIP_Số bút toán"` > Nhấn Print Screen. *(Kết quả: Hiển thị phiếu hạch toán chuyển tiền).*

### DV_TP_TC001: Thu phí bằng tiền mặt
1. **Thao tác thực hiện:** TTV vào hệ thống thực hiện khởi tạo bút toán thu phí tiền mặt.
2. **Duyệt thu phí:** KSV đăng nhập > Menu `HGCHRG` > Function: `Verify` > Event ID: Nhập mã phí, User ID: User hạch toán > Nhấn Go > Chọn dòng giao dịch > Kiểm tra và Submit.
3. **In giấy nộp tiền:** TTV vào menu `HPR` > Report Name: `"VOUCHER_GCHRG"` > Nhấn Print Screen.
4. **In bảng kê:** TTV vào menu `HADVC` > Transaction ID: Nhập số bút toán > Print Denomination Slip: Chọn Only > Nhấn Print. Sau đó vào menu `HPR` > Report Name: `"DENOMINATION SLIP"` > Nhấn Print Screen.
5. **In phiếu hạch toán:** TTV vào menu `HPR` > Report Name: `"ACCOUNTING_SLIP_Số bút toán"` > Nhấn Print Screen.

### DV_RT_TC001: Rút tiền mặt trong hạn mức (Người rút khác chủ tài khoản)
1. **Thao tác thực hiện:** TTV khởi tạo giao dịch rút tiền mặt.
2. **In lệnh chi:** TTV vào menu `HPR` > Report Name: `"VOUCHER-số bút toán"` > Nhấn Print Screen.
3. **In bảng kê chi:** TTV vào menu `HPR` > Report Name: `"DENOMINATION SLIP"` > Nhấn Print Screen.
4. **Duyệt rút tiền:** KSV đăng nhập > Menu `HCASHWD` > Function: `P-Post` > Transaction ID: Nhập số giao dịch > Click Go kiểm tra > Click Submit duyệt.
5. **In phiếu hạch toán:** TTV vào menu `HPR` > Report Name: `"ACCOUNTING_SLIP_Số bút toán"` > Nhấn Print Screen.

### DV_NT_TC001: Nộp tiền mặt trong hạn mức (Người nộp khác chủ tài khoản)
1. **Thao tác thực hiện:** TTV khởi tạo giao dịch nộp tiền mặt.
2. **In phiếu nộp tiền:** TTV vào menu `HPR` > Report Name: `"VOUCHER-số bút toán"` > Nhấn Print Screen.
3. **In bảng kê thu:** TTV vào menu `HPR` > Report Name: `"DENOMINATION SLIP"` > Nhấn Print Screen.
4. **Duyệt nộp tiền:** KSV đăng nhập > Menu `HCASHDEP` > Function: `P-Post` > Transaction ID: Nhập số giao dịch > Click Go kiểm tra > Click Submit duyệt.
5. **In phiếu hạch toán:** TTV vào menu `HPR` > Report Name: `"ACCOUNTING_SLIP_Số bút toán"` > Nhấn Print Screen.

### DV_GT_TC001: Giải tỏa tài khoản cùng Sol
1. **Thao tác thực hiện:** TTV khởi tạo lệnh giải tỏa tài khoản.
2. **Duyệt giải tỏa:** KSV đăng nhập Finacle UAT > Vào chức năng duyệt giải tỏa > Nhập số tài khoản / mã yêu cầu > Kiểm tra và Submit.
- *Kết quả mong muốn:* Hệ thống hiện thông báo `"Record verified successfully"`.

### DV_PT_TC001: Phong tỏa tài khoản
1. **Thao tác thực hiện:** TTV khởi tạo lệnh phong tỏa tài khoản.
2. **Duyệt phong tỏa:** KSV đăng nhập > Thực hiện kiểm tra và duyệt lệnh phong tỏa. *(Kết quả: Duyệt lệnh phong tỏa thành công).*
3. **Hủy lệnh phong tỏa trước khi duyệt:** KSV hoặc TTV chọn chức năng Hủy/Reject yêu cầu phong tỏa khi chưa duyệt. *(Kết quả: Hủy lệnh thành công, tài khoản trở lại trạng thái bình thường).*

### TDA_LTRC_OP_001: Mở mới Tài khoản lãi trước bằng tiền mặt
1. **TTV đăng nhập hệ thống:** Microsoft Edge, Finacle Core.
2. **Mở mới tài khoản:** Menu Shortcut `HOAACSB` (hoặc mở tài khoản có kỳ hạn) => Nhập CIF ID => Tab Scheme: Chọn Scheme code thuộc sản phẩm lãi trước, nhập kỳ hạn (Deposit period) => Tạo số tài khoản thành công.
3. **KSV đăng nhập hệ thống:** Đăng nhập user KSV.
4. **Duyệt mở mới tài khoản:** Vào màn hình duyệt tài khoản có kỳ hạn => Kiểm tra các tab => Submit duyệt thành công.

### CAA_OP_002: Mở mới Tài khoản thanh toán Combo
1. **TTV đăng nhập hệ thống:** Microsoft Edge, Finacle Core.
2. **Mở mới tài khoản:** Menu Shortcut mở tài khoản không kỳ hạn => Nhập CIF ID => Tab General: Charge Level Code, Scheme code thuộc sản phẩm Combo => Tạo số tài khoản thành công.
3. **KSV đăng nhập hệ thống:** Đăng nhập user KSV.
4. **Duyệt mở mới tài khoản:** Vào màn hình phê duyệt tài khoản không kỳ hạn => Kiểm tra Tab General, Tab Interest... => Submit duyệt thành công.

### TDA_LTRC_OP_004: Mở mới Tiền gửi lãi trước xuất sổ tiết kiệm
1. **TTV đăng nhập hệ thống:** Khởi tạo mở tài khoản có kỳ hạn lãi trước.
2. **Mở mới tài khoản:** Nhập CIF, Scheme code lãi trước, kỳ hạn => Tạo số tài khoản.
3. **KSV đăng nhập & Duyệt:** KSV kiểm tra và duyệt mở tài khoản thành công.
4. **Xuất sổ tiết kiệm:** TTV vào menu Shortcut `CPBMNT` => Function: In/Xuất sổ tiết kiệm. *(Kết quả kiểm tra nghiệp vụ: Hệ thống thông báo "Customer is not a passbook holder" nếu chưa thiết lập).*
5. **Truy vấn giao dịch mở tài khoản:** TTV vào menu `HTI` => Function: Inquiry => Kiểm tra giao dịch đủ 4 vế và trạng thái Verified cả 4 vế.
