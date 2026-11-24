---
name: pdf-flipbook-reader
description: Dựng trang đọc PDF dạng sách lật (flipbook) có hiệu ứng giấy uốn cong, bóng gáy sách, zoom, chế độ tràn viền và lazy load trang, bằng StPageFlip (page-flip) + PDF.js + Tailwind CDN. Dùng skill này mỗi khi Miku nhắc tới lật sách, lật trang, flipbook, page flip, StPageFlip, "chế độ đọc sách", UniDocs, đọc PDF kiểu sách, hiệu ứng giấy cong, hoặc muốn tích hợp trang đọc sách vào dự án mới hoặc dự án có sẵn (reader.html, set.html) - kể cả khi không nói rõ "dùng skill" hay không nhắc tên thư viện. Cũng dùng khi sửa lỗi, tối ưu bộ nhớ, tăng độ nét khi zoom hay đổi giao diện của trang đọc sách này.
---

# PDF Flipbook Reader

Trang đọc PDF kiểu sách thật: kéo góc trang để lật, giấy uốn cong, đổ bóng theo nếp gấp. Bản gốc do Antigravity code cho dự án UniDocs, lưu nguyên văn ở `assets/reader-template.html`.

## Khi nào dùng

- Dự án mới cần "xem PDF như cuốn sách" (tài liệu học tập, giáo trình, catalogue, truyện tranh dạng PDF).
- Dự án có sẵn cần thêm một trang đọc sách riêng, mở từ trang danh sách (vd `set.html?id=...` bấm vào tài liệu thì mở `reader.html?file=...`).
- Sửa/tối ưu chính trang này (nét hơn khi zoom, đổi giao diện, đổi hiệu năng).

## Cách hoạt động (đọc kỹ trước khi sửa)

Hai thư viện làm hai việc khác nhau, code của mình là cái keo dán:

| Thư viện | Việc |
|---|---|
| PDF.js 3.11.174 | Vẽ nội dung từng trang PDF ra `<canvas>` |
| StPageFlip (`St.PageFlip`) | Lật trang: kéo góc, uốn giấy, đổ bóng, tự chuyển 1 trang / 2 trang theo màn hình |

Luồng `initReader()`:

1. Lấy đường dẫn PDF từ query `?file=...` (thiếu thì báo lỗi rồi `closeReader()`).
2. `pdfjsLib.getDocument()` lấy tổng số trang.
3. **Tạo trước toàn bộ `div.page` rỗng** (chỉ có spinner) cho mọi trang. Trang chẵn là `page-left`, trang lẻ là `page-right`; trang đầu và cuối là `data-density="hard"` (bìa cứng), còn lại `soft`.
4. Lấy trang 1 ở scale 1.5 để đo kích thước chuẩn của sách.
5. `initPageFlip(width, height)` rồi `loadFromHTML(document.querySelectorAll('.page'))`.
6. Render trước 4 trang đầu, tắt loader, hiện thanh điều khiển.

Lazy load:

- `renderPage(n)`: vẽ trang n vào canvas, tập `renderedPages` chống vẽ lại.
- `managePageBuffer(index)`: gọi sau mỗi sự kiện `flip`. Vẽ trước ±5 trang quanh trang hiện tại. Khi đã vẽ hơn 30 trang thì trả spinner cho trang cách xa hơn 15 trang để nhả RAM.

Chỉ thay nội dung **bên trong** `.page-content`, không đụng vào chính `div.page`, vì StPageFlip đang giữ tham chiếu tới các `div.page`.

## Cấu hình StPageFlip

```js
new St.PageFlip(bookContainer, {
    width, height,           // từ viewport trang 1 (scale 1.5)
    size: "stretch",         // co giãn theo khung chứa
    minWidth: 315, maxWidth: 1000,
    minHeight: 420, maxHeight: 1350,
    maxShadowOpacity: 0.5,
    showCover: true,         // trang 1 là bìa đơn
    mobileScrollSupport: false
});
```

## Tích hợp vào dự án

1. Copy `assets/reader-template.html` vào dự án (vd `reader.html`).
2. Nơi mở sách truyền đường dẫn qua query: `reader.html?file=<url-pdf>&id=<subjectId>`.
3. `closeReader()` thử `window.close()` (nếu mở bằng tab mới), rồi `history.back()`, cuối cùng về `set.html?id=<id>` hoặc `index.html`. Đổi hai tên trang này cho khớp dự án.
4. File PDF phải truy cập được từ trình duyệt: cùng origin hoặc server bật CORS, nếu không PDF.js báo lỗi và hiện alert.
5. Các CDN (Tailwind, Phosphor, PDF.js, StPageFlip) chạy bằng script tag. Khi đưa vào dự án thật, nên **ghim version cụ thể** cho StPageFlip (bản gốc dùng unpkg không ghim) và cân nhắc tải về self-host nếu dự án chạy offline hoặc mạng công ty chặn CDN.

Nếu dự án dùng framework (React/Vue/Svelte), giữ nguyên logic: tạo danh sách `.page` trước, gọi `loadFromHTML` sau khi DOM sẵn sàng, và render canvas vào `.page-content` từng trang.

## Điểm tinh chỉnh hay gặp

- **Số trang hiển thị**: ở chế độ 2 trang hiện `3-4`, ở chế độ 1 trang hiện `3`, tính trong handler `flip` bằng `getOrientation()`.
- **Bóng gáy**: `.page-left::after` và `.page-right::after`, gradient đen mờ rộng 60px sát gáy. Chỉnh `width` và độ đậm cuối gradient để đổi độ sâu.
- **Zoom**: `transform: scale()` trên `#bookContainer` (0.5 đến 2), zoom lớn hơn 1.1 thì khung cha bật `overflow-auto`.
- **Chế độ tràn viền**: bật/tắt class `wide-mode` trên `#mainArea` để giảm padding.

## Vấn đề đã biết (có thể cải thiện khi được nhờ)

- Khối CSS `.flip-book` khai báo hai lần, bản sau (có `transition: transform`) đè bản đầu. Xóa bản đầu được.
- Zoom bằng `scale` chỉ phóng to ảnh canvas đã vẽ ở scale 1.5 nên zoom 200% bị mờ. Muốn nét: render lại trang hiện tại ở scale cao hơn khi zoom thay đổi (nhớ nhân thêm `devicePixelRatio`), hoặc tăng scale render mặc định đổi lấy RAM.
- Sách có số trang chẵn thì trang cuối là bìa sau và xử lý hiển thị số trang ở chế độ 2 trang có nhánh đặc biệt; kiểm tra lại nếu đổi `showCover`.

## Quy tắc khi sửa code của Miku

Giữ nguyên mọi comment có sẵn trong template (đặc biệt comment ghi "CẤM XÓA"), không xóa hay viết lại. Khi thêm code mới thì thêm comment mới bên cạnh, không thay comment cũ.
