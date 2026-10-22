# Deploy (bản của NAUCode)

Cập nhật 22/10/2026. Mọi dữ kiện dưới đây đã kiểm ngày 21 và 22/10/2026, trừ chỗ ghi "chưa kiểm". File này nói về bản của NAUCode; tài liệu deploy của chính Paperclip nằm ở `doc/` và `docs/deploy/`.

Chuẩn chung: tài liệu "Hướng dẫn deploy và tổ chức repo trên GitHub" (Ops Portal, kệ Standards; nguồn `naucodehq/naucode-portal`, `content/md-docs/standards/huong-dan-deploy.md`).

## Repo

- Vị trí: `github.com/naucodehq/paperclip`, công khai, org `naucodehq`, nhánh mặc định `master`.
- Chuyển từ `2ngnhan/paperclip` ngày 20/10/2026; đường cũ (web và API) chuyển hướng về đây.
- Upstream: `github.com/paperclipai/paperclip`. GitHub không ghi repo này là fork của upstream, và tài khoản NAUCode không có quyền push lên upstream.
- Bản clone ở máy BOD: `~/dev/paperclip`, `origin` là `naucodehq/paperclip`, `upstream` là `paperclipai/paperclip`, `master` bám `origin/master`. `gh` chưa đặt repo mặc định: luôn gọi kèm `-R naucodehq/paperclip`.
- Vai trò: fork béo cũ phía sau n'clerk. Production không chạy repo này. App https://app.nclerk.com dựng từ repo `naucodehq/n-clerk`: `Dockerfile` kéo upstream `paperclipai/paperclip` theo `ARG PAPERCLIP_REF` (đang là `ad961227`) rồi áp overlay; Railway đang kéo code từ bản sao `technaucode/n-clerk` (xem DEPLOY.md của n-clerk). Kế hoạch thu gọn rồi archive repo này: `n-clerk/docs/fork-thinning.md`.

## Đích deploy

| Đích | Trạng thái 21/10/2026 |
| --- | --- |
| npm canary (`paperclipai`, `@paperclipai/*`) qua `.github/workflows/release.yml`, environment `npm-canary` | Không có bản nào từ repo này. Mọi push lên `master` chạy `verify_canary` rồi `publish_canary` (npm trusted publishing qua OIDC). Cả 8 lần chạy Release (22/03 tới 10/07/2026) đều đỏ: hai lần ngày 22/03/2026 đã qua verify và hỏng ở `publish_canary` (hai deployment `npm-canary` hỏng), sáu lần còn lại đỏ ở `verify_canary` (lần 10/07 ở bước test). Theo `doc/RELEASE-AUTOMATION-SETUP.md`, trusted publisher của npm chỉ khai cho `paperclipai/paperclip`, không cho repo này (chưa kiểm trên npm). Repo không có tag nào. |
| npm stable, environment `npm-stable` | Chỉ chạy tay (`workflow_dispatch`). Chưa từng chạy ở đây; environment này chưa tồn tại. |
| Cloudflare Pages `ppclip` | Không còn. Từng nối Git hồi tháng 3/2026 (check run cuối trên `f59ad00`); `ppclip.pages.dev` không phân giải nữa, commit cuối của `master` (10/07/2026) không có check run của Cloudflare. |
| Render | Không có. Commit `1a2c945` chỉ nhắc "Render-compatible"; repo không có `render.yaml`, không có status hay deployment của Render trên 30 commit gần nhất, app GitHub của Render không cài trên org `naucodehq`. |
| app.nclerk.com | Không deploy từ repo này (xem mục Repo). `/api/health` trả 200, phục vụ từ Railway sau Cloudflare. |

Không có webhook, không có GitHub Pages. App GitHub duy nhất cài trên org là "Cloudflare Workers and Pages" (mọi repo), nhưng không có project Cloudflare nào nối repo này: commit đầu nhánh PR ngày 20/10/2026 không có check run nào của Cloudflare.

## Một lần deploy diễn ra thế nào

Không có lần deploy nào mong muốn từ repo này. Việc một lần push kích hoạt hôm nay:

- Push lên `master`: `Release` (`verify_canary` rồi `publish_canary`) và `Refresh Lockfile` chạy. Test qua là `publish_canary` thử publish lên npm (đã xảy ra hai lần ngày 22/03/2026, hỏng ở bước publish). Không để chuyện đó xảy ra từ bản này.
- `Refresh Lockfile` có quyền ghi (`contents`, `pull-requests`): nếu lockfile lệch, nó force-push nhánh `chore/refresh-lockfile` rồi thử mở PR và bật auto-merge vào `master`. Với cài đặt hiện tại (Actions không được tạo hay duyệt PR: `can_approve_pull_request_reviews: false`; repo `allow_auto_merge: false`) bước mở PR sẽ hỏng, để lại nhánh lẻ và một lượt đỏ. Kiểm bằng `gh api repos/naucodehq/paperclip/actions/permissions/workflow` và `gh api repos/naucodehq/paperclip --jq .allow_auto_merge`. Kể cả khi bot merge được, merge do `GITHUB_TOKEN` làm không kích hoạt workflow, nên không chạy lại Release: PR #1 (github-actions tạo, merge thành `694a23e3` ngày 22/03/2026) không sinh lượt chạy nào.
- Pull request vào `master`: workflow `PR` gồm `policy`, `verify` (typecheck, test, build, chạy thử release canary) và `e2e` (Playwright). Không publish. Test đang đỏ ở `verify`.
- `e2e.yml` và `release-smoke.yml`: chỉ chạy tay.
- Commit có `[skip ci]` trong message không kích hoạt workflow nào ở trên. Commit chỉ sửa tài liệu lên `master` thì luôn ghi `[skip ci]`.

`master` không có branch protection, nên test đỏ không chặn merge.

## Deploy tay và dev ở máy

Không có deploy tay. Chạy code ở máy (Node 20 trở lên, `pnpm@9.15.4`):

```sh
pnpm install
pnpm dev        # API và UI cùng ở http://localhost:3100, DB nhúng khi DATABASE_URL trống
pnpm test:run
```

`pnpm dev` chạy server với `PAPERCLIP_MIGRATION_AUTO_APPLY=true`. Hướng dẫn chi tiết: AGENTS.md và `doc/DEVELOPING.md`.

Bẫy DB ở máy: `~/dev/paperclip/.env` (gitignore, không commit) đặt `DATABASE_URL` trỏ tới một Postgres trên Supabase, chưa kiểm là DB nào. `pnpm dev` chạy server với thư mục làm việc `server/` nên không nạp file này và vẫn dùng DB nhúng. Nhưng lệnh nào chạy server từ gốc repo (`server/src/config.ts` nạp `.env` của thư mục làm việc) hoặc shell nào đã export biến này sẽ nối vào DB đó, và server tự áp migration khi chạy không TTY (như Bash của Claude Code). Không chạy server hay `pnpm db:migrate` với biến này khi chưa xác định DB.

## Kiểm sau deploy

Không có deploy để kiểm. Sau mỗi lần merge vào `master`:

- `gh run list -R naucodehq/paperclip -L 5`: commit có `[skip ci]` thì không có lượt chạy mới; nếu Release có chạy, `publish_canary` phải không thành công.
- `npm view paperclipai dist-tags`: các bản ở đó là của upstream; không được có bản nào publish từ repo này.

## Rollback

Không có gì để rollback. Nếu một bản canary lỡ publish từ đây: tắt Release (`gh workflow disable Release -R naucodehq/paperclip`) và báo chủ gói npm (upstream Paperclip, NAUCode không phải chủ gói) để `npm deprecate` bản đó.

## Secret và biến (chỉ tên)

- Secret và biến Actions mức repo: không có. `e2e.yml` đọc `ANTHROPIC_API_KEY`, chưa đặt.
- Environment: chỉ có `npm-canary`, không secret (trusted publishing qua OIDC, không dùng token). `npm-stable` chưa tồn tại, sẽ tự tạo nếu ai chạy `publish_stable`.
- Biến chạy app là của upstream: `docs/deploy/environment-variables.md`.
- Ở máy: `.env` ở gốc bản clone (không commit) chỉ có `DATABASE_URL`.

## Khi chuyển repo sang org khác

Lần chuyển 2ngnhan sang naucodehq (20/10/2026) đã xong:

- [x] Không có chuỗi chủ cũ viết cứng trong code (`git grep -e 2ngnhan -e technaucode -e ncutils` chỉ ra CLAUDE.md và DEPLOY.md, nơi nhắc lịch sử).
- [x] `~/dev/paperclip`: `origin` trỏ `naucodehq/paperclip`.
- [x] `n-clerk/docs/fork-thinning.md` ghi `naucodehq/paperclip`.
- [x] Cloudflare Pages `ppclip`: không còn. Render: không có.

Lần sau (chuyển tiếp hoặc archive):

- Đưa tài liệu lên `master` TRƯỚC khi archive, vì archive xong repo chỉ còn đọc.
- Không có secret, webhook, Pages hay tích hợp nào phải nối lại. Kiểm lại `n-clerk/docs/fork-thinning.md` và memory dự án, nơi còn nhắc tên repo này.

## Việc còn lại

1. Chủ repo quyết: tắt `Release` và `Refresh Lockfile` (`gh workflow disable Release -R naucodehq/paperclip`, `gh workflow disable "Refresh Lockfile" -R naucodehq/paperclip`), hoặc archive repo theo `fork-thinning.md`. Chưa làm, đây là thay đổi cài đặt của chủ repo.
2. Xác định `DATABASE_URL` trong `~/dev/paperclip/.env` là DB nào; không còn dùng thì xoá hoặc đổi tên file.
3. Bản clone `~/dev/paperclip`: nhánh `claude/fervent-gould` có một commit chưa push ("centralize branding into BRAND_NAME"), có một stash trên `master`, và bốn nhánh `claude/*` khác không có gì mới. BOD quyết giữ hay bỏ commit đó; bốn nhánh còn lại xoá được.
