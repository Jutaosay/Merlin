# Merlin 项目维护说明（基础维护阶段）

> 项目：Merlin 380 平台 shadowsocks 插件（仅做功能维护，不改路由策略）

## 1. 维护边界（必须遵守）

- **不改默认路由表策略**（Merlin 与 OpenWrt 方案不同）。
- 本仓库仅做：
  - Web 配置页面字段维护
  - dbus 配置映射维护
  - 运行配置生成与启动脚本维护
- 二进制更新源沿用上游（当前脚本指向 `cary-sas/v2ray_bin` 等）。

## 2. 页面显示逻辑与执行逻辑（核心链路）

### 2.1 Web 端
- 文件：`shadowsocks/webs/Main_Ss_Content.asp`
- 关键函数：
  - `verifyFields()`：控制不同协议字段显隐
  - `save()`：保存主面板配置到 `ss_basic_*`，并同步到 `ssconf_basic_*_<node>`
  - `add_ss_node_conf(flag)`：新增节点写入 `ssconf_basic_*_<node>`

### 2.2 后端执行
- 文件：`shadowsocks/ss/ssconfig.sh`
- 关键函数：
  - `create_hy2_json()`：生成 `/koolshare/ss/hysteria.json`
  - `start_hy2`：按配置启动 Hysteria2

### 2.3 网页测速/连通测试
- 文件：`shadowsocks/scripts/ss_webtest.sh`
- 关键函数：
  - `create_hy2_json()`：生成 `/tmp/tmp_hysteria.json` 进行测试

## 3. Hysteria2 新增字段（本次维护）

新增 3 个输入项：
- `obfs`（混淆密码）
- `up`（上传限速，mbps）
- `down`（下载限速，mbps）

### 3.1 dbus 字段
- 主面板：
  - `ss_basic_hy2_obfs`
  - `ss_basic_hy2_up_mbps`
  - `ss_basic_hy2_down_mbps`
- 节点：
  - `ssconf_basic_hy2_obfs_<node>`
  - `ssconf_basic_hy2_up_mbps_<node>`
  - `ssconf_basic_hy2_down_mbps_<node>`

### 3.2 配置映射
- `hy2_obfs` 非空时生成 salamander obfs 块
- `hy2_up_mbps` / `hy2_down_mbps` 任一非空时生成 bandwidth 块
- 仅填一侧时默认补齐：`up=10`、`down=50`

## 4. 已提交记录
- Commit: `1133e1a`
- Message: `feat(hy2): add obfs/up/down fields in UI and hysteria2 config generation`
- Files:
  - `shadowsocks/webs/Main_Ss_Content.asp`
  - `shadowsocks/ss/ssconfig.sh`
  - `shadowsocks/scripts/ss_webtest.sh`

## 5. binary 与包目录整理策略
- `380_armv5/<component>/`：每组件保留最近 3 版
- `380_armv5_packge/`：保留最近 3 个发布目录

## 6. 路由器侧回归测试建议
1. Hysteria2 节点测试：obfs 空/非空，up/down 空/单侧/双侧
2. 检查 `/koolshare/ss/hysteria.json` 字段生成
3. 检查 Hysteria2 拉起与代理
4. Trojan/Trojan-Go/Hysteria2 切换显隐是否正常
5. 路由表行为与历史一致（本次未改路由逻辑）

## 7. 后续维护建议
每次协议字段变更按“四处同步”执行：
1) 页面字段
2) `verifyFields()` 显隐
3) `save()` / `add_ss_node_conf()` 的 dbus 映射
4) 运行配置 + webtest 配置生成
