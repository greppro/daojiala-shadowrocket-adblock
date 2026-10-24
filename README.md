# 到家啦（智能门锁）开屏广告拦截｜Shadowrocket

使用 iPhone 上的 Shadowrocket，通过域名 `REJECT` 规则尝试拦截「到家啦」App 的开屏广告。本项目来自实际连接日志排查与个人测试，不是官方规则，**不保证所有版本、设备或网络环境均有效**。

> **当前测试反馈**：启用 V2 规则当天开屏广告仍存在，第二天观察到广告消失。**可能**与本地广告缓存到期有关，但没有通过受控实验确认因果关系；也不能确定 23 条规则均为必要条件。

## 下载

| 文件 | 下载 / 导入 URL |
| --- | --- |
| 模块（推荐） | [daojiala_adblock.sgmodule](https://raw.githubusercontent.com/greppro/daojiala-shadowrocket-adblock/main/config/daojiala_adblock.sgmodule) |
| 独立配置 | [daojiala_adblock.conf](https://raw.githubusercontent.com/greppro/daojiala-shadowrocket-adblock/main/config/daojiala_adblock.conf) |

导入时使用上表链接指向的原始文件 URL，不要使用 GitHub 文件展示页 URL。

- [`config/daojiala_adblock.sgmodule`](config/daojiala_adblock.sgmodule)：**推荐**，规则模块，适合保留现有代理配置。
- [`config/daojiala_adblock.conf`](config/daojiala_adblock.conf)：独立 Shadowrocket 配置，兜底策略为 `FINAL,DIRECT`；**不会继承你原来的代理节点及分流**。

## 使用方法一：模块（推荐）

1. 在 GitHub 下载 `.sgmodule` 文件，保存到 iPhone「文件」App，或使用文件的 Raw 链接。
2. 打开 Shadowrocket → **配置** → **模块**（不同版本的入口名称可能不同）。
3. 通过导入本地文件或添加以下模块 URL 的方式添加，启用该模块。

   ```text
   https://raw.githubusercontent.com/greppro/daojiala-shadowrocket-adblock/main/config/daojiala_adblock.sgmodule
   ```

4. 保持原代理配置启用，确认运行在配置/规则模式；不要同时启用旧版重复规则。
5. 完全退出「到家啦」，重新启动，检查开屏广告与开门功能。
6. 在 Shadowrocket「数据」中核对对应域名是否命中 `REJECT`。

如果你的版本不支持本地模块导入，可直接使用上述模块 URL，或改用下述 CONF 规则合并方式。

## 使用方法二：CONF

### 方案 A：独立导入

下载 [`daojiala_adblock.conf`](config/daojiala_adblock.conf)，在 Shadowrocket → **配置**中导入并选择该配置。它使用 `FINAL,DIRECT`，即未命中的流量直连，**会改变原有代理路由行为**。适合临时验证，不推荐替换日常使用的订阅配置。

```text
https://raw.githubusercontent.com/greppro/daojiala-shadowrocket-adblock/main/config/daojiala_adblock.conf
```

### 方案 B：合并到现有配置（更适合长期使用）

1. 备份当前正在使用的 `.conf` 配置；订阅配置可能在更新时覆盖手动修改。
2. 打开 [`daojiala_adblock.conf`](config/daojiala_adblock.conf)，复制 `[Rule]` 后的全部 `DOMAIN,...,REJECT` 行（**不要复制 `FINAL,DIRECT`**）。
3. 将这些规则插入原配置的 `[Rule]` 区域，放在 `FINAL` / `MATCH` 等兜底规则之前；注意其他优先匹配规则。
4. 保存、启用修改后的配置，重启 App 验证。

## 为什么刚加规则仍然有广告？

广告可能预加载或缓存到本地：阻断新的广告请求不一定立刻清除已经下载的图片/视频，也不一定消除 App 内置的广告等待页。本次测试中**当天仍有广告，次日未出现**，符合缓存失效的可能性，但也可能受广告投放策略、网络状态、请求路径或时间影响。

建议在开门功能正常的前提下持续观察 2–5 天。**不要为了清缓存直接卸载门锁 App**，以免影响登录、钥匙授权或设备配置。断网测试时，远程开门可能不可用，请确保有其他可靠的开门方式。

## 更新与停用

- 发布新规则后，在 Shadowrocket 中更新已添加的远程模块；手动合并的规则需要自行同步。
- 停用模块方式：在模块列表关闭本模块，重新连接 Shadowrocket，再重启 App。
- 停用独立配置方式：切回原来的配置；手动合并方式则恢复备份，或删除此次添加的规则。
- 停用规则不会立即改变 App 已缓存的广告或数据，仍需区分网络请求和本地缓存。

## 排查流程

1. 在 Shadowrocket「数据」中确认广告相关域名的策略为 `REJECT`，不是 `DIRECT`。
2. 记录打开 App 的精确时间；清空旧连接日志，重新启动 App，再导出启动前后约 30 秒日志。
3. 检查是否仍有广告 SDK 或素材请求 `DIRECT`，但不要仅凭域名名称认定它一定属于该 App。
4. 如果广告仍在，观察是完整素材、空白等待还是直接进入首页；不同结果对应不同问题。
5. 需要 HTTPS 解密时只针对已确认的广告接口，避免收集、上传门锁凭证、Cookie、令牌等敏感数据。

## 安全与兼容性

- 规则为**域名级全局拦截**，并非仅作用于「到家啦」；其他使用相同广告 SDK 的 App 也可能受影响。
- 不拦截 `kingflying.cn` 及其子域名，不对整个共享 CDN 后缀做屏蔽。
- 本项目不修改 DNS 服务器、不要求越狱或 HTTPS 解密。
- 请分别验证 App 登录、蓝牙/本地开门、远程开门等关键功能。出现异常时立即停用模块或恢复配置备份。
- 23 条规则是排查过程中逐步汇总的**候选规则集合**，并非逐条经过独立有效性验证的最小集合。

## 规则说明

V2 包含 23 条 `DOMAIN` 规则，覆盖排查中出现的 AnyThink、Pangle/GroMore、Sigmob、FancyDSP、腾讯广告及其他广告 SDK 相关域名。具体清单见 [`config/daojiala_adblock.sgmodule`](config/daojiala_adblock.sgmodule)。不包含门锁业务域名，也没有通配屏蔽整个平台或 CDN。

## 免责声明

本项目为个人经验分享，与「到家啦」、Shadowrocket 或相关广告服务商无关联。请遵守当地法律及软件使用条款，自行评估门锁业务可用性。**不要提交含有个人身份信息、设备 ID、鉴权凭证、门锁地址或访问令牌的抓包数据库。**
