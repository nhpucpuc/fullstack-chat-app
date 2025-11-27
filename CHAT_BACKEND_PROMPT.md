Dưới đây là một “development prompt” mẫu bạn có thể dùng để giao cho AI hoặc cộng tác viên hiện thực backend chat box cho hệ thống tìm kiếm mentor (Node.js + Express + TypeScript + SQL Server), bám theo cấu trúc thư mục bạn đã nêu. Lưu ý repo tham chiếu ở đây là ứng dụng chat realtime MERN có JWT, Socket.io và các tính năng realtime tương tự, nên hãy tận dụng các ý tưởng đó khi chuyển sang ngăn xếp Node/Express/TypeScript/SQL Server.

---

### Prompt gợi ý
> Bạn là kỹ sư backend xây dựng module chat box cho hệ thống tìm kiếm mentor. Stack: **Node.js + Express + TypeScript + SQL Server**. Thư mục nguồn nằm trong `server/src/` với các nhánh con: `config`, `constants`, `controllers`, `database` (chứa script SQL), `middlewares`, `routes`, `services`, `types`, `utils`, `validation`, `index.ts`.  
> 
> **Yêu cầu kiến trúc & chuẩn mã:**
> - Dùng TypeScript strict, cấu hình tsconfig hiện có. Không dùng `any`/`unknown` tùy tiện; khai báo `types`/`interfaces` trong `src/types`.
> - Kết nối SQL Server qua pool (ví dụ `mssql`). Tách cấu hình DB vào `src/config` (env: DB_HOST, DB_USER, DB_PASSWORD, DB_NAME, DB_PORT, DB_SSL?). Khởi tạo pool tại `src/database/index.ts` hoặc module tương đương và tái sử dụng qua DI/import.
> - Tổ chức logic theo layered approach: `routes` -> `controllers` -> `services` -> `database/repositories` (nếu tách). Controller chỉ parse input/response; service xử lý nghiệp vụ; tầng DB dùng parameterized queries/stored procedures.  
> - Áp dụng middlewares cho auth (JWT), validation (celebrate/zod/yup tùy sẵn có; đặt schema ở `src/validation`), error handling chuẩn Express (middleware 4 tham số) và logging.
> - Sử dụng WebSocket/Socket.io gateway để push realtime. Tách cấu hình Socket.io vào `src/config/socket.ts` hoặc file chuyên biệt, đăng ký namespace/rooms per conversation.
> 
> **Tính năng chat box cần có:**
> 1. **Khởi tạo cuộc trò chuyện (mentor/mentee):**
>    - Endpoint POST `/api/chats` nhận `mentorId`, `menteeId`, `topic?`.
>    - Kiểm tra trùng cuộc trò chuyện mở; nếu đã có thì trả về conversation hiện có.  
>    - Lưu vào SQL Server bảng `Conversations` (Id, MentorId, MenteeId, Topic, Status, CreatedAt, UpdatedAt).
> 2. **Gửi/nhận tin nhắn:**
>    - Endpoint POST `/api/chats/:conversationId/messages` nhận `{ content, attachments? }`.
>    - Lưu `Messages` (Id, ConversationId, SenderId, Content, AttachmentUrl?, CreatedAt, IsRead).
>    - Emit Socket.io event `message:new` tới room `conversation:{id}`; phía client join room sau khi mở chat.
> 3. **Lấy lịch sử chat với phân trang:**
>    - Endpoint GET `/api/chats/:conversationId/messages?cursor=...&limit=...` (hoặc page/limit). Trả về messages + `nextCursor`.
> 4. **Đánh dấu đã đọc & trạng thái online:**
>    - Endpoint PATCH `/api/chats/:conversationId/read` để mark messages của user kia là read; update `IsRead`.
>    - Socket.io event `user:online`, `user:offline`, `message:read`.
> 5. **Danh sách cuộc trò chuyện:**
>    - Endpoint GET `/api/chats?role=mentor|mentee` trả về danh sách conversations kèm last message, unread count.
> 6. **Upload/file attachment (nếu cần):**
>    - Middleware upload (Multer) + lưu URL; giới hạn định dạng/kích thước; quét virus nếu có.
> 
> **Bảo mật & kiểm thử:**
> - Bắt buộc JWT guard cho mọi endpoint chat; decode userId, role và dùng trong kiểm tra quyền (mentor/mentee phải thuộc conversation).
> - Chống SQL injection bằng parameterized queries; validate tất cả payloads qua schema ở `src/validation`.
> - Thêm rate limit cho endpoints chat và middleware chống spam socket (per-interval).
> - Viết unit/integration tests (Jest) cho controllers/services; mock DB layer. Nếu có CI, thêm scripts vào `package.json`.
> 
> **Đầu ra mong muốn:**
> - Các file/đường dẫn chính:
>   - `src/config/db.ts` (pool SQL Server), `src/config/socket.ts`.
>   - `src/types/chat.ts` (Conversation, Message, enums).
>   - `src/services/chat.service.ts` (nghiệp vụ conversation/message).
>   - `src/controllers/chat.controller.ts` (handlers Express).
>   - `src/routes/chat.routes.ts` (mount `/api/chats`).
>   - `src/middlewares/auth.ts`, `src/middlewares/error.ts`, `src/middlewares/rate-limit.ts`.
>   - `src/validation/chat.schema.ts`.
> - Scripts SQL mẫu trong `src/database/` để tạo bảng/stored procedures.  
> - Hướng dẫn env (DB creds, JWT_SECRET, SOCKET_PATH/PORT), lệnh chạy dev/prod, và mô tả sự kiện Socket.io.  
> 
> **Chất lượng & log:**
> - Log có cấu trúc (JSON) cho lỗi/requests; không log secrets.
> - Trả HTTP status code đúng chuẩn; body theo dạng `{ success, data, error }`.
> - Đảm bảo code format (Prettier/ESLint) và guard types cho mọi response DTO.

Bạn có thể chỉnh sửa prompt này để khớp với conventions cụ thể trong codebase của bạn (ví dụ style đặt tên biến, cách tổ chức `services`/`repositories`, hay format logger).
