# paperclip (bản của NAUCode)

Deploy: đọc DEPLOY.md trước khi deploy hoặc chuyển repo.

## Repo này là gì

- Bản fork cũ ("fork béo") của Paperclip, mã nguồn mở của `paperclipai/paperclip`: control plane cho công ty vận hành bằng AI agent. NAUCode dùng Paperclip làm engine của app https://app.nclerk.com, cho NAUCode và các hợp đồng mã hoá phương pháp của khách.
- Production KHÔNG chạy repo này. Image của app.nclerk.com dựng từ `Dockerfile` của n-clerk: kéo upstream `paperclipai/paperclip` theo `ARG PAPERCLIP_REF` rồi áp overlay. Railway hiện kéo code từ bản sao `technaucode/n-clerk`, chưa từ `naucodehq/n-clerk`, nên push lên `naucodehq/n-clerk` chưa lên production. Sửa code ở đây không lên production. Muốn đổi app.nclerk.com thì làm trong `~/dev/n-clerk` theo DEPLOY.md của repo đó (có mục chuyển nguồn Railway). Ở `~/dev/n-clerk`, `origin` là bản sao technaucode, không push lên đó; chỉ push lên remote `naucodehq`.
- Kế hoạch là thu gọn rồi archive repo này: `n-clerk/docs/fork-thinning.md`.
- Repo: `github.com/naucodehq/paperclip`, công khai, org `naucodehq` (NAUCode), nhánh mặc định `master`. Chuyển từ `2ngnhan/paperclip` ngày 20/10/2026, đường cũ chuyển hướng về đây.

## Cảnh báo

- Push hay merge vào `master` là chạy workflow `Release` (verify rồi thử publish npm canary `paperclipai`, `@paperclipai/*`) và `Refresh Lockfile` (có quyền ghi: lockfile lệch thì nó force-push nhánh `chore/refresh-lockfile` rồi thử mở PR và bật auto-merge; với cài đặt hiện tại, Actions không được tạo PR và auto-merge tắt, bước mở PR sẽ hỏng và để lại nhánh lẻ cùng một lượt đỏ). Release đang bật. Commit chỉ sửa tài liệu thì ghi `[skip ci]` trong message; không merge code vào `master` khi Release chưa tắt.
- Chỉ push lên `origin` (`naucodehq/paperclip`). Remote `upstream` là `paperclipai/paperclip` của bên thứ ba, không có quyền push. Repo này không phải fork trên GitHub, nên PR lên upstream phải đi từ một fork thật.
- `gh` ở `~/dev/paperclip` chưa đặt repo mặc định và có remote `upstream`: luôn gọi kèm `-R naucodehq/paperclip`.
- Repo công khai: không commit secret, `.env`, template hay dữ liệu của khách.
- `~/dev/paperclip/.env` (không commit) đặt `DATABASE_URL` trỏ tới một Postgres trên Supabase, chưa kiểm là DB nào. Server tự áp migration khi chạy không TTY: không chạy server hay `pnpm db:migrate` với biến đó. Chi tiết ở DEPLOY.md, mục dev ở máy.

## Làm việc

- Hướng dẫn dev của Paperclip nằm ở AGENTS.md, nạp ngay dưới. Chạy ở máy: `pnpm install`, `pnpm dev`, để trống `DATABASE_URL`.
- Copy tiếng Việt không dùng em dash; dùng dấu chấm, hai chấm, phẩy, "·".
- Chuẩn chung về org và deploy: `~/dev/naucode-portal/content/md-docs/standards/huong-dan-deploy.md` (trên web: ops.naucode.com, kệ Standards).

@AGENTS.md
