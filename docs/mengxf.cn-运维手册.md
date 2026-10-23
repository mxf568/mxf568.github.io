# mengxf.cn 运维手册（上线版）

> 适用范围：mengxf.cn 个人网站（阿里云部署版）
> 整理：2026-10-23 · 本文档随仓库公开，**不要在此提交密码、证件号等敏感信息**

## 一、当前架构

```
浏览器
  │  https
  ▼
DNS：阿里云云解析（@ 与 www 均 A 记录 → 47.104.94.70，TTL 600）
  ▼
阿里云 ECS · 华北1（青岛）· 实例 i-m5ed04kw8bloje3y7c76（公网 IP 47.104.94.70）
  规格：2核2G / 3M固定带宽 / 40G ESSD Entry · 系统：Alibaba Cloud Linux 3
  ▼
nginx 1.24（systemd 服务名 nginx）→ 站点根目录 /usr/share/nginx/html/
```

- 本仓库（mxf568/mxf568.github.io）＝**网站内容备份与历史版本库**，不参与对外服务
- ICP 备案号：鲁ICP备2026032396号-1（页脚已悬挂，链接 beian.miit.gov.cn）

## 二、日常更新网站内容的流程

1. 修改本地文件（或直接在服务器上改）`/usr/share/nginx/html/index.html`
2. 静态文件**无需 reload nginx**，保存即生效
3. 浏览器 Ctrl+F5 强制刷新验证
4. 建议每次改动前备份：`cp -a /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.bak_$(date +%Y%m%d_%H%M%S)`
5. 有余力时把新版 index.html 提交回本仓库，保持备份同步

## 三、HTTPS 证书

- 签发：Let's Encrypt（certbot --nginx，覆盖 mengxf.cn + www.mengxf.cn）
- 当前证书到期：**2027-01-20**，certbot 已配置自动续期定时任务
- 手动检查续期：`certbot renew --dry-run`
- 续期后自动 reload nginx（certbot 部署时已配置）

## 四、服务器与端口

- SSH：root@47.104.94.70:22（密钥认证已配置；密码登录仍开启，建议后续关闭密码登录仅留密钥）
- 安全组（默认安全组 sg-n5elb0tqj3g8neqkzz）入方向：22 / 80 / 443 / ICMP / 3389
- nginx 配置：/etc/nginx/nginx.conf（server_name mengxf.cn www.mengxf.cn；80 强制 301 → 443）

## 五、合规与年检备忘

| 事项 | 要求 | 状态 |
|---|---|---|
| ICP 备案号悬挂 | 首页底部居中 + 链接 beian.miit.gov.cn | ✅ 已挂 |
| 域名解析 | 须持续解析到阿里云内地服务器（30 天规则） | ✅ 已指向 ECS |
| 公安联网备案 | 网站开通后 30 日内（数据码已使用） | 提交于 2026-10-22，待属地网安审核 |
| 公安备案号悬挂 | 过审后在页脚加公安备案标识 + beian.gov.cn 链接 | 待审核通过后办理 |
| 备案信息变更 | 主体/联系方式变化后 30 日内在阿里云备案系统提交变更 | 持续注意 |

## 六、常见操作速查

```bash
# 查看 nginx 状态
systemctl status nginx
# 重载配置（改 conf 后）
nginx -t && systemctl reload nginx
# 看实时日志
tail -f /var/log/nginx/access.log
# 回滚到某个备份
cp -a /usr/share/nginx/html/index.html.bak_20261023_152522 /usr/share/nginx/html/index.html
```

## 七、注意事项

1. 本仓库是**公开仓库**：任何密码、证件号、数据码一律不得提交
2. ECS 登录密码如已泄露请及时在控制台重置；推荐只保留密钥登录
3. 网站内容须与备案口径一致（个人、非经营性、无交互栏目），改动性质前先评估备案影响
