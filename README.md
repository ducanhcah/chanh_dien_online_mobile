# Chánh Điện Tịnh Tâm

Chánh điện online 2.5D cho TikTok LIVE. Mỗi người xem bình luận là bước vào chánh điện, ngồi xuống bồ đoàn tụng kinh. Ai tặng quà sẽ được xướng tên hồi hướng bằng giọng đọc tiếng Việt. Có nhạc thiền, chuông, mõ và đọc kinh định kỳ.

Mã nguồn chia sẻ miễn phí (MIT). Có hai phần:

| Thư mục | Là gì |
|---|---|
| `live/overlay.html`, `assets/`, `live-config.js`, `vercel.json` | Giao diện xem-only kiểu overlay, triển khai trên Vercel |
| `live/` | Server Node cho WebSocket TikTok LIVE, overlay và bảng điều khiển |

## Đưa bản web lên điện thoại

Bản web dùng hai dịch vụ: Vercel phục vụ giao diện overlay xem-only, còn Render chạy WebSocket để kết nối TikTok LIVE và đồng bộ người xem/bình luận. Mở domain Vercel sẽ tự vào overlay; không cần nhập pháp danh.

1. Tạo một **Render Blueprint** từ repository, branch `main`. `render.yaml` chọn gói **Free**, thư mục `live`, lệnh build `npm install`, lệnh chạy `npm start` và tối đa 5 ID TikTok khác nhau cùng lúc.
2. `render.yaml` cấu hình `HOST=0.0.0.0` và `ALLOWED_ORIGINS=https://live-kappa-ruby.vercel.app`. Nếu dùng domain Vercel riêng, cập nhật `ALLOWED_ORIGINS` trước khi deploy.
3. Sau khi Render tạo hostname, mở `live-config.js` và đặt `window.CHANH_DIEN_LIVE_SERVER` thành địa chỉ HTTPS của service, ví dụ `https://ten-service.onrender.com`.
4. Trên Vercel, đặt **Root Directory** trống (thư mục gốc có `index.html`), **Framework Preset** là **Other**, **Build Command** để trống và **Output Directory** là `.`. Triển khai lại.
5. Mở domain Vercel trên điện thoại, nhập ID hoặc liên kết của bất kỳ kênh TikTok công khai đang LIVE, rồi bấm **Kết nối LIVE**. Kết nối thành công thì màn hình nhập ID tự ẩn và overlay chỉ hiển thị người xem/bình luận/quà LIVE; không cần nhập pháp danh. Dùng cụm điều khiển góc phải dưới để tắt/bật nhạc, tăng/giảm âm lượng và bật/thoát toàn màn hình; lần chạm đầu tiên cũng mở quyền phát âm thanh cho trình duyệt. Nếu Safari không hỗ trợ toàn màn hình trong tab, dùng **Chia sẻ → Thêm vào Màn hình chính**.

Trên iPhone, để mở không có thanh địa chỉ, mở domain bằng Safari, chọn **Chia sẻ → Thêm vào Màn hình chính**, rồi mở bằng icon vừa tạo. Cốc Cốc trên iOS cũng không thể bật Fullscreen API trong tab thường.

Trang phải dùng HTTPS để trình duyệt cho phép âm thanh. Tên người tặng quà được đọc bằng giọng TTS tiếng Việt khi âm thanh đã được mở trên trình duyệt. Mỗi ID TikTok có room/sự kiện riêng; các tab chọn ID khác nhau không nhận sự kiện chéo. Nhiều tab cùng ID chia sẻ một kết nối upstream để tiết kiệm tài nguyên, nhưng mỗi tab giữ canvas riêng. Backend mặc định giới hạn 5 ID khác nhau đồng thời và một lần mở room mới mỗi 10 giây. Render Free có thể ngủ sau 15 phút không có truy cập và cần khoảng một phút để khởi động lại; chờ rồi kết nối lại. Muốn nhiều kênh đồng thời hơn cần tăng `MAX_PUBLIC_CHANNELS` và dùng máy chủ đủ tài nguyên. Nút quà trên web chỉ mô phỏng, không xử lý thanh toán; quà thật từ TikTok được nhận từ sự kiện LIVE.

Nếu TikTok connector báo `rate_limit_account_day`, hãy tạo API key theo hướng dẫn tại [EulerStream](https://www.eulerstream.com/pricing), rồi thêm `EULERSTREAM_API_KEY` trong **Render → service → Environment** và deploy lại. Key chỉ nằm ở backend; không đưa key vào `live-config.js`, `render.yaml` hay Git.

## Chạy bộ LIVE trên máy phát

Cần Node.js 20 trở lên.

```bash
cd live
npm install
npm start
```

1. Mở `http://localhost:3000/live/control.html` (bảng điều khiển).
2. Bấm **Mở màn hình overlay 9:16**. Để bật tiếng, nhấp hoặc nhấn phím bất kỳ một lần trong cửa sổ overlay (trình duyệt chặn tự phát tiếng).
3. Trong TikTok LIVE Studio: *Add source → Window capture* → chọn cửa sổ overlay. Với OBS thì dùng *Browser source* trỏ tới `http://localhost:3000/live/overlay.html`, kích thước 1080×1920.
4. Bật LIVE trên TikTok, nhập `@username` vào bảng điều khiển, bấm **Kết nối**.

Chưa live vẫn thử được: dùng **Thử tương tác**, **Đổ người mẫu 15/36/64/100** hoặc **Bot mô phỏng**.
Nhạc nền mặc định tắt; có thể bật lại bằng ô **Nhạc nền** trên bảng điều khiển.

### Người xem làm được gì

- **Bình luận bất kỳ**: được vào ngồi (tối đa 100 chỗ, chỗ ngồi tự mở rộng 24 → 36 → 64 → 100). Đầy chỗ thì người im lặng lâu nhất mà chưa tặng quà sẽ nhường chỗ.
- **Bình luận đúng một từ** `nam`, `nữ`, `sư`, `ni`, `tiểu`, `bà`, `cao tăng`: đổi hình nhân vật. Câu "Nam mô A Di Đà Phật" không bị hiểu nhầm thành "nam".
- **Tặng quà**: banner xướng tên, giọng đọc hồi hướng (lấy lời nguyện từ bình luận gần nhất của người đó nếu có), hiệu ứng theo giá trị quà, cộng điểm vào bảng Công Đức.
- **Thả tim**: tim bay trên đầu nhân vật.
- **Đổ người mẫu 15/36/64/100**: thêm người cho đến tổng số chỗ tương ứng; **Làm trống** cũng dừng bot đang chạy.

## Bảo mật

Dự án được viết để ai tải về chạy cũng an toàn mặc định:

- **Không dùng tài khoản TikTok.** Server chỉ đọc sự kiện công khai của phòng live theo username. Không cần, không hỏi và không lưu mật khẩu, cookie hay session. Không gửi tin nhắn thay bạn.
- **Local mặc định.** Server lắng nghe `127.0.0.1`. Chỉ cấu hình `HOST=0.0.0.0` khi triển khai backend công khai.
- **Giới hạn backend public.** Public mode chỉ phục vụ `/health`, file TTS và WebSocket; kiểm tra `Host`/`Origin`, chỉ role web với các lệnh kết nối được chấp nhận. Mỗi TikTok ID có room riêng; các tab của cùng ID dùng chung nguồn sự kiện. Tối đa 5 ID khác nhau hoạt động mặc định, và yêu cầu mở room mới có cooldown 10 giây. Bảng điều khiển nội bộ không được mở trên public mode.
- **File tĩnh có danh sách trắng.** Chỉ phục vụ `/assets` và `/live`. Chặn file ẩn (`.git`, `.env`), `node_modules`, mã server, `package.json` và mọi đường dẫn `../`.
- **Không tin dữ liệu vào.** Tên người xem, bình luận và tên quà đều bị cắt độ dài. Cài đặt chỉ nhận đúng khóa, đúng kiểu, trong giới hạn. Văn bản đưa vào giọng đọc được escape XML. Mọi chữ trên màn hình vẽ bằng `textContent` hoặc canvas, không chèn HTML, nên bình luận chứa mã độc chỉ hiện ra như chữ thường.
- **Chống spam.** Hàng đợi giọng đọc tối đa 15 câu. Bình luận thường không đọc thành tiếng.
- **Phụ thuộc ghim phiên bản** trong `live/package.json`: `ws`, `msedge-tts`, `tiktok-live-connector`. Không có khóa API nào trong mã.

Các lớp trên đã được thử: đường dẫn `../`, `%2e%2e`, `.git/config`, `server.mjs`, `package.json` đều trả 404. Host giả trả 421. WebSocket từ Origin lạ bị từ chối. Lệnh gửi từ overlay bị bỏ qua. Cài đặt sai kiểu bị kẹp về giới hạn.

### Lưu ý

- `tiktok-live-connector` là thư viện cộng đồng, không phải API chính thức của TikTok. Nó có thể ngừng chạy khi TikTok đổi hệ thống. Kiểm tra điều khoản TikTok trước khi dùng cho kênh của bạn.
- Giọng đọc dùng dịch vụ Read Aloud của Microsoft Edge qua mạng (giọng Nam Minh, Hoài My). Máy phát cần có Internet.
- Hình ảnh trong `assets/` được tạo bằng Google Flow, nhạc nền tạo bằng Gemini. Kiểm tra điều khoản của các dịch vụ đó trước khi dùng thương mại, hoặc thay bằng tài nguyên của bạn.

## Miễn trừ trách nhiệm

Dự án được chia sẻ **miễn phí, với mục đích học tập và tham khảo lập trình**. Phần mềm được cung cấp "nguyên trạng" (AS IS), không kèm bất kỳ bảo đảm nào, rõ ràng hay ngụ ý.

Khi tải về, sử dụng, chỉnh sửa hoặc phân phối lại mã nguồn này, **bạn tự chịu hoàn toàn trách nhiệm** về:

- Mọi nội dung bạn phát trên livestream hoặc đăng tải, bao gồm hình ảnh, âm thanh, giọng đọc, lời nguyện, bình luận được hiển thị.
- Việc nhận quà, donation hoặc bất kỳ khoản thu nào từ người xem.
- Việc tuân thủ điều khoản sử dụng của TikTok và các nền tảng, dịch vụ bên thứ ba khác (thư viện đọc sự kiện TikTok là thư viện cộng đồng, không chính thức).
- Việc tuân thủ pháp luật nơi bạn sinh sống và hoạt động, kể cả quy định về hoạt động tôn giáo, quảng cáo, thuế và bản quyền.
- Mọi rủi ro và thiệt hại phát sinh, trực tiếp hay gián tiếp, như bị khóa tài khoản, bị dừng live, mất dữ liệu, tranh chấp hay khiếu nại.

Dự án không đại diện cho bất kỳ chùa, tu viện hay tổ chức tôn giáo nào. Khi dùng nội dung liên quan đến Phật giáo, xin giữ sự tôn kính và không dùng vào mục đích lừa đảo, trục lợi hay xúc phạm niềm tin của người khác.

Việc sử dụng mã nguồn đồng nghĩa với việc bạn đã đọc và đồng ý với các điều trên.

> **Disclaimer (EN):** This project is shared free of charge for educational purposes only and is provided "AS IS", without warranty of any kind. Anyone who uses, modifies or redistributes it is solely responsible for their own content, streams, donations, compliance with TikTok and other platforms' terms, and applicable laws. The project is not affiliated with or endorsed by any religious organization. Users are responsible for any direct or indirect damages arising from their use of the project.

## Giấy phép

MIT, xem `LICENSE`.
