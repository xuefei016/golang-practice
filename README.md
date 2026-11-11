# golang-homework

Course work in Go, organized as several independent modules. It covers basic algorithms, concurrency, a GORM blog data layer, a Gin + JWT blog HTTP API, and go-ethereum interaction with the Sepolia test network.

This is not a single runnable application. Each `backend/homework0N` directory is its own Go module. `lesson1/` is an earlier exercise under the root module. There is no implementation for homework 06 — only the assignment text.

The official assignment statements live in `backend/homework01.md` … `backend/homework06.md` and link to [MetaNode Academy](https://github.com/MetaNodeAcademy/go-homework1-template). Starter templates for the first two assignments are under `backend/templates/`.

## Modules

| Path | What it actually contains |
| --- | --- |
| `lesson1/` | Early practice in the root module (`golang-homework`, Go 1.26.5). `calculator.go` is a working variadic calculator (add / subtract / multiply / divide, history, divide-by-zero error) with a `main` demo. `homework.go` is the same eight algorithm problems as homework 01, but only `SingleNumber` and `IsPalindrome` are implemented; the rest are `TODO` stubs. `student.go` is assignment comments only (a student list and a bank-account exercise) with no code. |
| `backend/homework01/` | Eight LeetCode-style functions in `homework.go`: single number (XOR), palindrome number, valid parentheses, longest common prefix, plus one, in-place remove duplicates, merge intervals, and two sum. Unit tests and a GitHub Actions grading workflow are included. Stdlib only (module `homework01`, Go 1.20). |
| `backend/homework02/` | Pointers, goroutines, a small `Shape` interface (`Rectangle`, `Circle`), `Employee` embedding `Person`, unbuffered and buffered channels, a mutex counter, and an atomic counter. Implementations are in `homework.go`, with unit tests. Stdlib only (module `homework02`, Go 1.22). |
| `backend/homework03/` | A GORM blog data layer against **MySQL** (not SQLite). Models: `User` 1–N `Post` 1–N `Comment`, plus `PostCount` and `CommentStatus`. Hooks: `Post.AfterCreate` increments the user's `post_count`; `Comment.AfterDelete` sets the post's `comment_status` to `no_comments` when no comments remain. Queries: a user's posts with comments, the most-commented post, and comment deletion helpers. `main` auto-migrates, **deletes existing users/posts/comments**, seeds sample data, then runs those queries. (module `homework03`, Go 1.26.5; `gorm.io/gorm` + `gorm.io/driver/mysql`. A SQLite driver appears in `go.mod` / `go.sum` but is not imported by the code.) |
| `backend/homework04/` | A Gin + GORM **MySQL** blog HTTP API with bcrypt password hashing, JWT auth (`github.com/golang-jwt/jwt/v5`), and a unified JSON response helper. Public routes: `GET /health`, `POST /api/user/register`, `POST /api/user/login`, `GET /api/posts`, `GET /api/posts/:id`, `GET /api/posts/:id/comments`. Bearer-JWT routes: `POST /api/posts`, `PUT /api/posts/:id`, `DELETE /api/posts/:id`, `POST /api/posts/:id/comments` (create / update / delete of a post are restricted to the author). Logging uses Gin's default middleware (`gin.Default()`). No tests in this module. (module `homework04`, Go 1.26.5; Gin, GORM/MySQL, jwt/v5, `golang.org/x/crypto`.) |
| `backend/homework05/` | go-ethereum (`v1.17.4`) programs for an Ethereum JSON-RPC endpoint (intended for Sepolia). `main.go` connects with `ethclient`, prints block hash / number / timestamp / transaction count (latest, or `-blockNum`), then signs and sends a hardcoded 0.001 ETH (`1e15` wei) transfer. `cmd/counter` deploys the `Counter` contract, reads `getCount`, calls `increment`, and reads again. A helper to bind to an already-deployed address (`NewCounter`) exists but is commented out in `main`. `contracts/Counter.sol` is a minimal Solidity ^0.8.0 counter; `build/Counter.abi` and `build/Counter.bin` are compile artifacts; `contract/counter.go` is abigen-generated binding code. No tests. (module `homework05`, Go 1.26.5.) |
| `backend/homework06.md` | Assignment text only (a written Go-Ethereum / Geth architecture study). There is no `homework06/` code directory. |
| `backend/templates/` | Course starter for homework 01 and 02 (stub `homework.go`, tests, README, CI workflows, and `homework02.zip`). Not submissions. |

## Tech stack

Taken from each module's `go.mod` and the imports that the code actually uses:

- **Go** 1.20 (homework 01), 1.22 (homework 02), 1.26.5 (root module, homework 03–05)
- **Standard library only** in homework 01 and 02 (`sync`, `sync/atomic`, channels, etc.)
- **GORM** `gorm.io/gorm` with **MySQL** (`gorm.io/driver/mysql`) in homework 03 and 04
- **Gin** `github.com/gin-gonic/gin` in homework 04
- **JWT** `github.com/golang-jwt/jwt/v5` and **bcrypt** (`golang.org/x/crypto/bcrypt`) in homework 04
- **go-ethereum** `github.com/ethereum/go-ethereum v1.17.4` in homework 05
- **Solidity** `^0.8.0` for `Counter.sol` (homework 05)

## Build, test, and run

Requires a local Go toolchain. Commands below assume the repository root as the starting point. Homework 01 and 02 tests were written to be run with `go test` in their own module directories.

### lesson1 (calculator demo)

```bash
go run ./lesson1
```

This builds the root module and runs the calculator `main`. It does not exercise the unfinished algorithm stubs or the empty student file. There are no tests in `lesson1/`.

### Homework 01 (algorithms)

```bash
cd backend/homework01
go test -v
```

### Homework 02 (concurrency and language features)

```bash
cd backend/homework02
go test -v
```

`TestProducerConsumer` sleeps 1 second per item, so that test alone takes about 10 seconds.

### Homework 03 (GORM blog models)

Needs a reachable MySQL instance. The program **wipes** the `users`, `posts`, and `comments` tables, then reseeds them.

```bash
export BLOG_MYSQL_DSN   # MySQL DSN, required; the process exits if unset
cd backend/homework03
go run .
```

There is no `go test` target in this module.

### Homework 04 (Gin + JWT blog API)

Needs MySQL. JWT secret and port have defaults if unset (see below).

```bash
export BLOG_MYSQL_DSN     # MySQL DSN, required; the process exits if unset
export BLOG_JWT_SECRET    # optional
export PORT               # optional, listen port
cd backend/homework04
go run .
```

The server listens on `:$PORT` (default `8080`). Example: `GET /health` should return `{"status":"ok"}`. Send `Authorization: Bearer <token>` after `POST /api/user/login`. There is no `go test` target in this module.

### Homework 05 (Sepolia / go-ethereum)

Needs a funded testnet account and an RPC URL. These programs **send real transactions** and will spend testnet ETH.

Query a block and send the hardcoded transfer:

```bash
export ETH_RPC_URL        # JSON-RPC URL, required
export ETH_PRIVATE_KEY    # account private key hex, required (no 0x prefix in the parser used here)
cd backend/homework05
go run .                  # latest block, then a 0.001 ETH transfer
go run . -blockNum=123    # specific block number, then the same transfer
```

The recipient address and amount are hardcoded in `main.go`, not taken from flags or environment variables.

Deploy (by default) and increment the Counter contract:

```bash
export ETH_RPC_URL
export ETH_PRIVATE_KEY
cd backend/homework05
go run ./cmd/counter
```

To talk to an already-deployed Counter instead of deploying a new one, the source has `getCallContractInstance` / `NewCounter`; that call is currently commented out in `cmd/counter/main.go`.

There is no `go test` target in this module.

## Environment variables

Names only — do not commit values. `.env` is already in `.gitignore`.

| Variable | Used by | Required? |
| --- | --- | --- |
| `BLOG_MYSQL_DSN` | homework 03 and 04 | Yes. Both programs exit if it is empty. |
| `BLOG_JWT_SECRET` | homework 04 | No. If unset, the code falls back to a built-in default string. Set this in any real use. |
| `PORT` | homework 04 | No. Defaults to `8080`. |
| `ETH_RPC_URL` | homework 05 (`main` and `cmd/counter`) | Yes. |
| `ETH_PRIVATE_KEY` | homework 05 (`main` and `cmd/counter`) | Yes. Hex private key; the code passes it to `crypto.HexToECDSA`. |

## What I wrote vs. course-provided material

This repository is MetaNode Academy Go course homework. The split below is based on git history (`git log --stat`), the files kept under `backend/templates/`, the assignment markdown, leftover `TODO` / stub comments, and (for homework 05) the `// Code generated - DO NOT EDIT.` header on the contract binding.

### Course-provided (or clearly copied from the course)

- **Assignment text:** `backend/homework01.md` … `backend/homework06.md`. Homework 01 links to the official [go-homework1-template](https://github.com/MetaNodeAcademy/go-homework1-template) and the academy learning roadmap. These were added in commit `2cb6b5f` (`feat: 完成 homework01 与 homework02，搭建 homework03 脚手架`).
- **Starter templates:** `backend/templates/homework01/`, `backend/templates/homework02/`, and `backend/templates/homework02.zip`. These contain stub function bodies (`// TODO: implement`), unit tests, README instructions, and GitHub Actions workflows. They were added in the same commit.
- **Copied submission scaffolding for homework 01 and 02:** `backend/homework01/` and `backend/homework02/` keep the template README, `go.mod`, tests, and workflows. `homework01/homework_test.go` is identical to the template test file. `homework02/homework_test.go` differs only by dropping an unused `sync` import. Function signatures, problem comments, and the `Shape` / `Person` / `Employee` type scaffolding in `homework.go` match the templates.
- **Homework 04 assignment sample:** `backend/homework04.md` includes sample GORM models and Register/Login snippets (including a placeholder JWT signing key in the write-up). That sample is reference text, not a file that was compiled as-is.
- **`lesson1/homework.go` problem stubs:** the eight function signatures and comments match the homework 01 template. Six of the eight bodies are still `TODO` stubs. **`lesson1/student.go`** is only exercise comments (no implementation). These arrived in the first commit `8b1c5c6` (`init: golang homework lesson1`), before the `templates/` directory existed in this repo; they are still clearly assignment text.

### What I implemented

- **`backend/homework01/homework.go` function bodies** — all eight problems, filling in the template stubs. Evidence: `diff` against `backend/templates/homework01/homework.go` changes only the implementations (the XOR / stack / two-pointer logic, etc.).
- **`backend/homework02/homework.go` function bodies** — pointers, a channel-synchronized odd/even printer, a wait-group task scheduler with timing, `Shape` methods, `PrintInfo`, producer-consumer and buffered-channel programs, mutex and atomic counters. Evidence: the same kind of diff against `backend/templates/homework02/homework.go`. A few leftover `// TODO:` comments remain on methods that are already implemented, because those comments were in the template.
- **`lesson1/calculator.go`** — full calculator and `main`. There is no matching template in this repo; it was added in `8b1c5c6`. This is likely my implementation of a course exercise, but the prompt text is not checked in separately.
- **`lesson1/homework.go`** — only `SingleNumber` and `IsPalindrome`; the other six stay stubs.
- **Homework 03** — all application code (`models/`, `query/`, `main.go`) was added in `17030d3` (`feat: 完成 homework03 GORM 博客系统（模型/关联查询/钩子）`). The earlier `2cb6b5f` commit only dropped in `go.mod` / `go.sum` (the “scaffold”). There is no homework 03 template directory. This is my implementation of the assignment in `backend/homework03.md`.
- **Homework 04** — the full API (`config`, `models`, `handlers`, `middleware`, `utils`, `main.go`) was committed in `d6ce3e6` and refined in `ced473f` (unified response helpers, Gin’s default logger). There is no template tree for this assignment. The design follows the assignment (register / login / JWT / post CRUD / comments) but is a structured rewrite: separate handler packages, `jwt/v5` rather than the deprecated `dgrijalva/jwt-go` shown in the sample, and a shared JSON envelope. Likely inspired by the sample, not a paste of it.
- **Homework 05 application code** — `contracts/Counter.sol`, `main.go`, and `cmd/counter/main.go` were added in `5322881`, with a follow-up `19cecce` that added the (currently unused in `main`) path to attach to an already-deployed contract. The assignment asked for a simple counter-style contract; this Solidity is a short increment / `getCount` contract I committed. **`contract/counter.go` is generated** (`// Code generated - DO NOT EDIT.`) and belongs with the `build/` ABI and bytecode, not with hand-written logic.

### Not implemented

- Homework 06: assignment markdown only.
- The student-management / bank-account exercise described in `lesson1/student.go`.
- Six of the eight algorithm functions in `lesson1/homework.go`.

## Notes

- No `.env` or other secret files are committed. `.gitignore` ignores `.env` and `*.env`.
- Homework 05 reads the private key from `ETH_PRIVATE_KEY` only; do not put a key in source. The transfer recipient is a hardcoded address in `main.go`.
- Homework 04’s JWT code uses `BLOG_JWT_SECRET` when set; otherwise it falls back to a default string in `config/config.go`. Override that before any non-local use.
