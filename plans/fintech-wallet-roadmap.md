# .NET Fintech Wallet Engineering Roadmap

**Status:** Plan only. No source-code modifications or tests are claimed as complete.  
**Original project:** [lychnellj/wallet-api](https://github.com/lychnellj/wallet-api) (ASP.NET Core 10, PostgreSQL, EF Core, JWT and Docker).  
**Planned fork:** `princly25/wallet-api`, or `princly25/fintech-wallet-lab` after renaming. The visible fork relationship should be retained.

## Attribution and usage

The upstream repository currently has no license file at the root. GitHub's fork feature preserves upstream attribution, but copying or relicensing source elsewhere must not be assumed to be permitted without a license. Keep our added work separate and accurately describe original versus new contributions. The public portfolio may link to the fork rather than reposting upstream files.

## Confirmed upstream features

- Wallets, users, authentication, PostgreSQL, Docker Compose, and deposit/withdraw/transfer endpoints are already present.
- The source has a tracked SQLite `wallet.db` file even though runtime configuration uses PostgreSQL.
- `POST /api/Transaction/transfer` **already exists**. Our addition should be reliable and idempotent transfers, *not* a claim to introduce transfers from scratch.
- Transaction controller methods currently have no `[Authorize]` attribute.
- `WalletService.TransferAsync` separately invokes repository `SaveChangesAsync` twice. Check data consistency, concurrency and transaction boundaries before deploying.
- Startup uses `EnsureCreated()`; migrations and production-grade initialization need a deliberate approach.
- Demo users/passwords and JWT key examples are currently present and must never be deployed as real credentials.
- Existing integration tests require a separately running API at `http://localhost:8080`; build a reproducible CI test harness.
- The source has no SQL Server EF Core provider wired in.

## Milestones and evidence

### M0 — Fork and baseline audit
- Use GitHub Fork, target owner `princly25`, retain source relationship.
- On branch `feat/baseline-security`, document original dependency versions and known behavior.
- Run `dotnet test` and local containers before claiming tests pass; record test output and exact commit.
- Ensure seeded demo credentials are development-only and JWT secrets come from environment/secrets stores.
- Deny unauthenticated transfer/deposit/withdraw actions; check ownership and roles on all wallet APIs.
- Replace tracked runtime database with safe migration handling, if authorized; investigate data sensitivity first.

### M1 — Robust, idempotent wallet transfers
- Design idempotency key scoped to authenticated principal + route, with persisted request hash, response metadata, unique constraint and conflict handling.
- Atomic debit/credit/ledger writes in one database transaction. Prevent double spending and duplicate requests under concurrent execution and retries.
- Add tests: same key/same payload retry, same key/different payload conflict, concurrent conflicting transfers, insufficient balance and crash/retry behavior.
- Define transaction isolation/concurrency behavior for both PostgreSQL and SQL Server before implementation.

### M2 — SQL Server / PostgreSQL provider switching
- Add EF Core SQL Server dependency and `DatabaseProvider` configuration switch.
- Separate provider-specific connection strings and migration assemblies / scripts as appropriate.
- Test identical behaviors using both PostgreSQL and SQL Server containers.
- Review SQL Server `rowversion` versus PostgreSQL concurrency-token semantics and existing migrations.

### M3 — AWS Lambda S3 statement ingestion
- Add a separate Lambda project for `ObjectCreated` events with strict prefix/bucket restrictions, idempotency, validation and safe failure handling.
- Provide synthetic sample statement objects and unit tests.
- Use IAM least privilege; never commit AWS credentials.
- Do not process real financial data or enable live cloud resources without separate approval.

### M4 — Container CI and deployment plans
- GitHub Actions build, test, dependency review and image build with no cloud secrets.
- Add AWS ECS Fargate + RDS deployment configuration using infrastructure-as-code.
- Prefer GitHub OIDC role assumption, restricted IAM permissions and manual production approvals.
- Make cloud deployment manual/opt-in to avoid unexpected costs.
- Tests must pass before an ECS release can proceed.

### M5 — Explain and demonstrate
- Document architecture, threat model, API examples, local setup, provider configurations and reproducible test results.
- Maintain an `UPSTREAM.md` attribution file and `CHANGES.md` tracing our commits to new features.
- Distinguish a code sample from actual production use; never claim 5+ years of work or team leadership solely from this exercise.

## Other project candidates

| Original repo | Verified licensing | Potential *new* work (not yet completed) |
| --- | --- | --- |
| [jrfrigat/PlaylistShare](https://github.com/jrfrigat/PlaylistShare) | MIT | Improve secure login/authorization, provider integration tests and CI; first inspect existing functionality |
| [aws-samples/use-github-and-terraform-to-deploy-web-applications](https://github.com/aws-samples/use-github-and-terraform-to-deploy-web-applications) | MIT No Attribution | Add least-privilege ECS deployment, OIDC policy, automated plan validation and cost guardrails |
| [a-patel/aws-dotnet-labs](https://github.com/a-patel/aws-dotnet-labs) | MIT | Modernize supported SDK/.NET versions and add safe Lambda/S3 test coverage |
| [abpframework/eShopOnAbp](https://github.com/abpframework/eShopOnAbp) | Needs review; archived | Security/performance test improvements only if project remains compatible and license permits |

## Commit discipline

Make genuinely incremental commits as work is implemented and tested. Do not backdate commits or artificially spread them over days. Prefer feature branches and PRs. Every README claim should have code, tests or a screenshot/log to support it.
