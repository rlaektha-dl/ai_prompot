# Security Specification & Invariants

## 1. Data Invariants
- Every document in `/prompts/{promptId}` MUST have a valid `promptId` matching `^[a-zA-Z0-9_-]+$` with length <= 128.
- Every prompt MUST contain exactly the 4 required fields: `title`, `category`, `content`, `createdAt` (no extra shadow fields).
- `title` must be a non-empty string (`1 <= size <= 200`).
- `category` must be one of `['사진 합성', '스타일 변환', '캐릭터·포스터']`.
- `content` must be a non-empty string (`1 <= size <= 20000`) so long prompts over 1,000 chars are supported while preventing resource exhaustion.
- `createdAt` must be a server timestamp (`request.time`) on creation and immutable on update.
- Public guests may read (`get`, `list`) prompts without signing in.
- Only the verified administrator (`dasomsome@gmail.com` with `email_verified == true`) may `create`, `update`, or `delete` prompts.

## 2. Adversarial Payloads Guarded Against
1. Unauthenticated or non-admin write (`create`, `update`, `delete`) attempts.
2. Admin email spoofing with `email_verified == false`.
3. Shadow field injection (`isAdmin: true`, `extraField: "hack"`).
4. Missing required fields (`title`, `category`, `content`, or `createdAt`).
5. Invalid `category` outside the 3 allowed categories.
6. Empty `title` or `content`.
7. Oversized `title` (> 200 chars) or `content` (> 20000 chars).
8. Wrong field types (e.g. number for `title`, list for `content`).
9. Spoofed `createdAt` client timestamp not equal to `request.time` on create.
10. Mutating `createdAt` during an update operation.
11. Invalid document ID containing special characters or exceeding 128 chars.
12. Arbitrary reads/writes to unknown collections (blocked by global default-deny rule).
