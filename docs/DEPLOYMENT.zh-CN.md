# 部署 Pi Native CRM

## 在线演示

[GitHub Pages 演示](https://jayfeng7.github.io/crm-agent-lab/)提供免登录的浏览器体验。`Publish interactive demo` 工作流先运行演示测试，再发布 `examples/chat-first-demo/dist`。派生仓库可以在 Settings → Pages 中选择 GitHub Actions 作为发布来源。

## 自托管完整 CRM

准备一台 Linux 主机、Docker Engine 与 Compose、指向主机的域名，以及独立的 Supabase 项目。Supabase 提供 Postgres、Auth、Storage 和 Realtime。

```bash
git clone https://github.com/jayfeng7/crm-agent-lab.git
cd crm-agent-lab
corepack pnpm install --frozen-lockfile
cp .env.hostgator.example .env
```

1. 参考 [环境变量说明](SETUP.md)，配置 Supabase URL、公开 anon key、服务端 service role key、数据库连接串、域名和随机生成的服务端密钥。结合 [应用模板](../.env.example)核对实际启用的集成，并将 `SIGNUP_MODE` 设为 `invite_only`。
2. 将 Supabase Auth 的 Site URL 和回调允许列表配置为部署地址。使用 `corepack pnpm db:migrate` 初始化空库，先确保连接指向独立的新库。已有数据库按照 [升级文档](ATUALIZANDO.md)走备份和迁移流程。
3. 按 [管理员初始化脚本](../scripts/bootstrap-owner.ts)的说明，用环境变量传入独立的管理员邮箱、密码和组织名称。从可信主机执行，完成后清理用于引导的密码变量。
4. 将域名 DNS 指向服务器，按生产模板配置 HTTPS、Redis 和 WAHA。使用生产 Compose 与本地构建覆盖文件，从同一份源码构建 Web、Worker 和 Scheduler。

```bash
docker compose -f docker-compose.prod.yml -f docker-compose.build.yml build app worker scheduler
docker compose -f docker-compose.prod.yml -f docker-compose.build.yml up -d
docker compose -f docker-compose.prod.yml -f docker-compose.build.yml ps
```

5. 从公网访问 `https://你的域名/login`，确认 HTTPS、登录、组织和客户页面可用。再在组织设置中绑定模型，选择合成客户运行一次只读任务，检查工具记录与运行状态。按启用的服务核对 `/api/v1/health` 和容器日志。

服务器上的 `.env`、数据库连接串、模型密钥和备份应保存在部署环境。详细运维步骤见 [自托管文档](deploy-selfhost/)与 [部署验收清单](DEPLOY-CHECKLIST.md)。
