# ar-image-assets

Kho ảnh cho app AR Drawing. Mỗi lần push, GitHub Action tạo `catalog.json` (danh sách ảnh + tên hiển thị) và deploy lên GitHub Pages:
https://phutran0703.github.io/ar-image-assets/

App tải `catalog.json` mỗi lần mở (1 request nhỏ), nên ảnh mới hiện ra mà không cần phát hành bản app mới.

## Cập nhật ảnh

### 1. Thêm ảnh

Chép ảnh vào `images/<Chủ đề>/`:

- **Chủ đề có sẵn:** chép vào thư mục đó, ví dụ `images/Christmas/`.
- **Chủ đề mới:** tạo thư mục mới, ví dụ `images/Dinosaur/`. Tên thư mục là tên chủ đề hiện trong app.
- **Banner màn Settings:** chép vào `images/Banner-Settings/`.

Chỉ dùng `.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`, đặt trực tiếp trong thư mục chủ đề (không tạo thư mục con).

### 2. Đặt tên hiển thị

Thêm một dòng cho mỗi ảnh vào `titles.json`:

```json
"images/Christmas/Screenshot 2026-10-26 at 15.16.00.png": "Santa Sleigh Reindeer",
```

- Các dòng ngăn cách bằng dấu phẩy; dòng cuối **không** có dấu phẩy.
- Có thể bỏ qua nếu tên file đã có nghĩa: `santa-claus.png` hiện là "Santa Claus".
- Nếu bỏ qua mà tên file vô nghĩa (ví dụ `Screenshot…`), app hiện tên dạng "Christmas 20".

### 3. Đặt loại mở khoá (tuỳ chọn)

Mỗi ảnh thuộc một trong 3 loại, khai báo trong `access.json`:

```json
"images/Christmas/santa.png": "premium",
```

- `free`: dùng ngay.
- `ad`: xem quảng cáo để mở vĩnh viễn (Premium mở luôn).
- `premium`: chỉ mở khi mua Premium.

Ảnh **không có** trong `access.json` được app tự chia theo vị trí trong chủ đề (40% free / 35% ad / 25% premium, xen kẽ, 2 ảnh đầu free). Giá trị sai hoặc path không tồn tại bị bỏ qua (xem log của Action). Cú pháp giống `titles.json`: dòng cuối không có dấu phẩy.

### 3b. Dịch tên hiển thị (tuỳ chọn)

Tên trong `titles.json` là tiếng Anh. Bản dịch nằm trong `title_translations/<mã ngôn ngữ>.json` (12 file: `vi`, `ja`, `ko`, `zh_Hans`, `zh_Hant`, `es`, `pt_BR`, `fr`, `de`, `id`, `th`, `ar`). Khoá là **tên tiếng Anh** (không phải path ảnh):

```json
"Santa Sleigh Reindeer": "Ông già Noel cưỡi xe tuần lộc",
```

- Thêm một dòng vào mỗi file cho mỗi tên mới. Cú pháp giống `titles.json`.
- Tên chưa dịch thì app hiện tiếng Anh. Log của Action in số tên còn thiếu ở từng ngôn ngữ.
- App chỉ tải file của ngôn ngữ đang dùng, và chỉ khi file đó thay đổi.

### 3c. Bộ sưu tập (sets/)

Mỗi bộ là một thư mục trong `sets/`. Người dùng vẽ xong **tất cả** ảnh của bộ thì nhận huy hiệu của bộ (hiện ở Home, Profile, kèm màn chúc mừng có hiệu ứng). Thêm hoặc sửa bộ chỉ cần push, không phải ra bản app mới.

```
sets/
  christmas-countdown/
    set.json
    badge.png        (tuỳ chọn)
  first-steps/
    set.json
```

`set.json`:

```json
{
  "order": 1,
  "name": { "en": "Christmas Countdown", "vi": "Đếm ngược Giáng sinh" },
  "description": { "en": "10 festive pictures to draw before Christmas.", "vi": "10 bức vẽ rộn ràng đón Giáng sinh." },
  "badge": { "emoji": "🎄", "shape": "star", "colors": ["#FF5A5F", "#C81D3A"] },
  "from": "2026-11-01",
  "until": "2027-01-10",
  "images": [
    "images/Christmas/....png",
    "images/Christmas/....png"
  ]
}
```

- **Tên thư mục** là id của bộ. Đã phát hành thì đừng đổi tên, vì tiến độ của người dùng gắn với id này.
- **images**: path ảnh đã có trong `images/` (copy đúng khoá trong `titles.json`). Ảnh có thể lấy từ nhiều chủ đề khác nhau, nên làm được bộ trộn như "First Steps". Nên để khoảng 10 ảnh. Path sai bị bỏ qua, xem log Action.
- **name / description**: theo mã ngôn ngữ (`en`, `vi`, `ja`, `es`, `pt_BR`, `zh_Hans`, `zh_Hant`, `ko`, `fr`, `de`, `id`, `th`, `ar`). Bắt buộc có `en`; ngôn ngữ thiếu thì app hiện tiếng Anh.
- **badge**: app tự vẽ huy hiệu.
  - `emoji`: hình ở giữa huy hiệu.
  - `shape`: một trong `circle`, `star`, `flower`, `heart`, `shield`, `hexagon`.
  - `colors`: 2 màu, sáng trước, đậm sau.
  - Muốn dùng ảnh riêng thì đặt `badge.png` (vuông, nền trong suốt) vào thư mục bộ.
- **from / until** (tuỳ chọn, `YYYY-MM-DD`): bộ theo mùa chỉ hiện trong khoảng này, ví dụ Giáng sinh, Tết, Valentine. Hết hạn thì bộ ẩn đi, nhưng ai đã đạt huy hiệu vẫn giữ.
- **order**: thứ tự hiện trên Home, số nhỏ lên trước.

### 4. Đẩy lên GitHub

```
cd ~/Desktop/IMAGE-AR
git add .
git commit -m "Thêm ảnh ..."
git push
```

Hoặc trên web: **Add file → Upload files**, rồi **Commit changes**.

### 5. Kiểm tra

Vào [Actions](https://github.com/PhuTran0703/ar-image-assets/actions) và đợi lần chạy mới nhất hiện ✅ (khoảng 1 phút).

Nếu hiện ❌, thường là do `titles.json`, `access.json`, một file trong `title_translations/` hoặc một `sets/*/set.json` sai cú pháp (thiếu dấu phẩy, hoặc thừa dấu phẩy ở dòng cuối). Sửa rồi push lại.

### 6. App tự cập nhật

Người dùng mở lại app (có mạng) là thấy ảnh và tên mới. Ảnh đã tải trước đó không tải lại; chỉ ảnh mới được tải về.

## Việc khác

- **Đổi tên ảnh đã có:** sửa tên trong `titles.json` (và khoá tương ứng trong `title_translations/*.json`), rồi làm bước 4.
- **Sửa bản dịch:** sửa file trong `title_translations/`, rồi làm bước 4.
- **Ghim "Topic of the Day":** ghi tên thư mục chủ đề vào `featured.json`, rồi làm bước 4. App sẽ hiện đúng chủ đề này thay vì random mỗi ngày (ví dụ dịp Halloween):

  ```json
  {
    "topic": "Spooky Halloween"
  }
  ```

  Tên phải khớp đúng tên thư mục trong `images/` (chữ hoa, khoảng trắng). Sai tên thì Action bỏ qua và in danh sách tên hợp lệ trong log. Muốn quay về random: để `"topic": ""`, rồi làm bước 4. Người dùng thấy thay đổi khi mở lại app (có mạng).
- **Đổi loại mở khoá:** sửa `access.json`, rồi làm bước 4.
- **Xoá ảnh:** xoá file trong `images/`, rồi làm bước 4.
- **Giới hạn GitHub Pages:** repo tối đa khoảng 1 GB (hiện khoảng 380 MB), khoảng 100 GB lượt tải mỗi tháng.
