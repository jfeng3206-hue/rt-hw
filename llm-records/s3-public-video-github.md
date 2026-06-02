# 把 S3 视频公开链接放到 GitHub 上 —— 对话整理

> 注：本文档已移除敏感信息。AWS 账号 ID 已用 `XXXXXXXXXXXX` 替代，S3 报错中的 RequestId / HostId 已删除。
> bucket 名称中出现的 `rt-mock-XXXXXXXXXXXX-us-west-2-an` 即为脱敏后的形式。

---

## 1. 问题：presigned URL 触发了 GitHub 密钥泄露告警

**Q:** 把 S3 bucket 里的视频对象链接放到 GitHub 上，最好的方式是什么？我生成了 presigned URL，结果 GitHub 提示 "Publicly leaked secret"。

**A 要点：**

- 告警是对的。presigned URL 的查询字符串里嵌了 AWS **access key ID**（`X-Amz-Credential=AKIA...`），GitHub 的 secret scanning 会抓到，所以提交它等于泄露了真实凭证。
- 另一个问题：presigned URL 会**过期**。SigV4 presigned URL 用 IAM 用户凭证签发时最多 7 天，用角色/STS 签发时往往更短。就算骗过 GitHub，链接几天后也会失效。

正确做法取决于视频是公开还是保密：

**如果视频是公开的（比如 README 里的 demo）** —— 不要签名，给对象稳定的公开访问：
- 加一条 bucket policy，对该对象（或 `demos/*` 这样的前缀）授予 `s3:GetObject` 给所有人。注意现代默认配置（Object Ownership = "Bucket owner enforced"）下 ACL 是关闭的，所以用 bucket policy，不用 per-object ACL。
- 然后链接普通 URL：`https://your-bucket.s3.your-region.amazonaws.com/path/video.mp4`。无凭证、不过期、不告警。
- 更进一步用 **CloudFront**：bucket 保持私有，挂一个带 Origin Access Control (OAC) 的 CloudFront 分发，链接 CloudFront URL。HTTPS + 缓存 + bucket 锁死，是生产标准做法。

**如果视频必须保密** —— 那就不该把可用链接提交进仓库。需要时用脚本/后端按需生成 presigned URL，仓库里只写明如何申请访问。

**额外选项：** 如果只是个 demo 片段，直接把视频拖进 GitHub issue / PR / Release，GitHub 会托管它并给一个稳定的 `githubusercontent.com` 链接，连 S3 都不用。

---

## 2. 如何加 bucket policy（公开读）

**关键坑：** 即使 policy 写对了，AWS 仍会拦截公开访问，除非先关掉 **Block Public Access**（新 bucket 默认开启）。所以是两步：先解锁，再加 policy。

### 第一步 —— 关掉 "Block Public Access"（只关够用的）

控制台路径：S3 → 你的 bucket → **Permissions** 标签 → **Block public access (bucket settings)** → **Edit**。

不需要四个框全取消。允许公开 *policy* 只需取消这两个：

- "Block public access to buckets and objects granted through *new* public bucket policies"
- "Block public access to buckets and objects granted through *any* public bucket policies"

两个 **ACL** 相关的框保持勾选 —— 你用的是 policy 不是 ACL，没理由放开它们。保存，提示时输入 `confirm`。

### 第二步 —— 加 bucket policy

同一 **Permissions** 标签 → **Bucket policy** → **Edit** → 粘贴。替换 `YOUR-BUCKET-NAME` 和路径。

单个对象：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadDemoVideo",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/demos/video.mp4"
    }
  ]
}
```

整个前缀（注意结尾的 `/*`）：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadDemosPrefix",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/demos/*"
    }
  ]
}
```

字段含义：

| 字段 | 含义 |
|---|---|
| `Principal: "*"` | "互联网上任何人" —— 这是让它公开的关键 |
| `Action: s3:GetObject` | 只读/下载对象。不含 list、write、delete。 |
| `Resource` 以对象名结尾 | 只有那一个文件 |
| `Resource` 以 `/*` 结尾 | 该前缀下的每个文件 |

**易错点：** `Resource` ARN 里**没有** `:region:` 或 `:account-id:` —— S3 这些冒号字段留空（`arn:aws:s3:::bucket/key`）。误填进去 policy 会静默失配。

### 第三步 —— 测试链接

```
https://YOUR-BUCKET-NAME.s3.YOUR-REGION.amazonaws.com/demos/video.mp4
```

用 incognito 窗口打开（避免以自己身份登录）。能播放/下载就成功。`AccessDenied` 通常是：Block Public Access 没关、ARN 路径对不上真实 key、或 URL 里 region 错了。

### CLI 版本

```bash
# 第一步：放开两个 policy 相关的 block，保留 ACL block
aws s3api put-public-access-block \
  --bucket YOUR-BUCKET-NAME \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=false,RestrictPublicBuckets=false

# 第二步：应用 policy（先把上面的 JSON 存为 policy.json）
aws s3api put-bucket-policy \
  --bucket YOUR-BUCKET-NAME \
  --policy file://policy.json
```

**提醒：** 这会让对象对*任何有链接的人永久可读*，且 S3 会按下载流量向你收费。小 demo 没问题。文件大或流量高，用 CloudFront + OAC 更省更安全。

---

## 3. 能用 bare bucket ARN 吗？

**Q:** 能用 `arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an` 吗？

**A:** 对 `s3:GetObject` 不行 —— 这正是 bare bucket ARN 失效的场景。

规则：`arn:aws:s3:::bucket`（无斜杠）指 **bucket 本身**，`arn:aws:s3:::bucket/...` 指 **里面的对象**。`s3:GetObject` 作用于对象，所以 `Resource` 必须在 bucket 名后带 `/key` 或 `/*`。用 bare bucket ARN，policy 语法有效但静默匹配不到任何东西，会 `AccessDenied`。

正确写法：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadDemos",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an/demos/*"
    }
  ]
}
```

bare bucket ARN 适合 *bucket 级* 操作如 `s3:ListBucket` —— 但这里不需要，因为 list 会让任何人枚举整个 bucket。只服务视频就用 `s3:GetObject` + `/*` 路径。

**小注：** bucket 名里带了 region（`us-west-2`）只是命名习惯，不改变 ARN 格式或 URL。公开链接仍是 `https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/demos/your-file.mp4`。

---

## 4. 我的 bucket 名到底是什么？

整串就是 bucket 名：

```
rt-mock-XXXXXXXXXXXX-us-west-2-an
```

虽然含 `us-west-2`，那只是名字的一部分。region 没有从名字里被剥离，它就是名字里的字符。

各处的写法：

- **Bucket ARN:** `arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an`
- **Object ARN (policy 用):** `arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an/demos/*`
- **公开 URL:** `https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/demos/your-file.mov`

URL 里 `us-west-2` 出现*两次* —— 一次在 bucket 名里，一次是 `.s3.us-west-2.amazonaws.com` 的真实 region。看着冗余但正确，二者独立、只是碰巧文本相同。

确认确切名字与真实 region：

```bash
aws s3 ls
aws s3api get-bucket-location --bucket rt-mock-XXXXXXXXXXXX-us-west-2-an
```

---

## 5. 这个 policy 能用于公开访问吗？

**Q:** 下面这个能用吗？

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadDemos",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an/demos/*"
    }
  ]
}
```

**A:** 能用，但有个用词更正 —— 这是 **bucket policy** 不是 ACL。两者是不同机制，你（正确地）用的是 policy 这个。ACL 是更老的 per-object 权限系统，现代 bucket 默认关闭，所以才用 policy。

它会对 `demos/` 前缀下任何东西授予公开读。生效前查两点：

1. **Block Public Access** 必须放开，否则 AWS 覆盖 policy，仍 `AccessDenied`。需要 `BlockPublicPolicy` 和 `RestrictPublicBuckets` 关闭。
2. 视频文件确实在 `demos/` 前缀下，即 key 是 `demos/something.mov`。在根目录或别的文件夹则匹配不到。检查：

```bash
aws s3 ls s3://rt-mock-XXXXXXXXXXXX-us-west-2-an/ --recursive
```

---

## 6. 如何测试

### 第一步 —— 拿到确切 URL

```bash
aws s3 ls s3://rt-mock-XXXXXXXXXXXX-us-west-2-an/demos/ --recursive
```

### 第二步 —— 快速测试：incognito 浏览器

把 URL 粘进 **隐私/无痕窗口**。普通窗口可能已登录 AWS，会掩盖匿名访问是否真的可用。无痕 = 无凭证 = 真实测试。

### 第三步 —— 精确测试：curl

```bash
# -I = 只看 headers（HEAD 请求），快，不下载视频
curl -I "https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/demos/demo.mp4"
```

期望看到 `HTTP/1.1 200 OK`。要看完整错误体就去掉 `-I`。

**重要：** 用普通 `curl`，**不要**用 `aws s3 cp` 或 `aws s3api get-object` —— 那些会用你的 AWS 凭证，即使公开访问坏了也会成功，给出假阳性。

### 第四步 —— AccessDenied 的三个常见原因

| 看到的 `<Code>` | 原因 | 修复 |
|---|---|---|
| `AccessDenied` | Block Public Access 没关 | 关 `BlockPublicPolicy` + `RestrictPublicBuckets` |
| `AccessDenied` | key 不在 `demos/` 下 | 重查 `aws s3 ls` 输出，key 要匹配 policy 的 `Resource` |
| `NoSuchKey` | URL 路径打错 | 从 `ls` 输出精确复制 key |
| `PermanentRedirect` | URL 里 region 错 | `aws s3api get-bucket-location --bucket ...` 确认 |

查 Block Public Access 状态：

```bash
aws s3api get-public-access-block --bucket rt-mock-XXXXXXXXXXXX-us-west-2-an
```

`BlockPublicPolicy` 和 `RestrictPublicBuckets` 都应为 `false`。

### 第五步 —— 用到 GitHub

直接把普通 URL 粘进 README。嵌入播放器徽章示例：

```markdown
[![Demo video](https://img.shields.io/badge/▶-Watch%20Demo-blue)](https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/demos/demo.mp4)
```

---

## 7. 命令在哪里输入？

`aws s3 ls ...` 不是链接，是命令，输入到**终端**（命令行）：

- **Mac:** Terminal 应用（Cmd+Space 搜 "Terminal"）
- **Windows:** Command Prompt / PowerShell / Windows Terminal
- 或代码编辑器里的终端（IntelliJ、VS Code 底部的 "Terminal" 标签）

先确认 AWS CLI 装好了：

```bash
aws --version
```

有版本号就 OK；"command not found" 说明没装。命令抱怨凭证时用 `aws configure` 配置。

两类字符串别混：

- `aws s3 ls ...` → **终端**（命令）
- `https://...amazonaws.com/demos/demo.mp4` → **浏览器** 或 **README**（链接）
- `s3://...` 形式只用于 AWS CLI 命令；`https://...` 才是给人和浏览器用的。

---

## 8. 报错：InvalidAccessKeyId

**现象：**

```
aws s3 ls s3://rt-mock-XXXXXXXXXXXX-us-west-2-an/demos/ --recursive

An error occurred (InvalidAccessKeyId) when calling the ListObjectsV2 operation:
The AWS Access Key Id you provided does not exist in our records.
```

这是**凭证**问题，不是 bucket 或命令问题。CLI 发的 access key 被 AWS 认为不存在。常见原因（按概率）：

**1. 过期/已删的 access key**（最常见）。key 被轮换、删除或停用。检查：

```bash
aws configure list
```

看 `access_key` 行（末几位带掩码），去 IAM 控制台确认该 key 还存在且 active。

**2. 存的 key 有拼写错误/多余空格。** 重跑配置：

```bash
aws configure
```

仔细粘贴（别手敲）access key ID 和 secret。

**3. 环境变量覆盖了配置文件**（隐蔽）。若 shell 里设了 `AWS_ACCESS_KEY_ID`，它会静默盖过 `~/.aws/credentials`。检查：

```bash
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
```

打印出东西（而非空）就是元凶，清掉：

```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
```

**4. 用错 profile。** 查看：

```bash
cat ~/.aws/credentials
```

加 `--profile yourprofile` 或 `export AWS_PROFILE=yourprofile`。

确认身份：

```bash
aws sts get-caller-identity
```

返回账号/用户 ARN 说明凭证有效。同时确认 `Account` 字段与 bucket 名里的账号一致；若不同，说明 CLI 指向了错误的 AWS 账号。

---

## 9. 回到目标：怎么把链接放到 GitHub 让别人检查作业

走 S3 公开这条链路对"交作业"偏重（要修凭证 + 关 Block Public Access + 配 policy）。

**关键澄清的需求：**
- 视频**公开就行**，谁有链接都能看无所谓
- 文件大小 **100MB ~ 2GB 之间**

### 方案 A：GitHub Release（推荐，最省事，不碰 AWS）

GitHub Release 允许直接上传文件作为附件，单文件最大 2GB —— 你的视频正好在范围内。上传后得到永久、公开链接，不需要 AWS、不需要凭证、不触发密钥告警。

步骤：

1. 仓库 → 右侧 **Releases** → **Create a new release**（或 **Draft a new release**）
2. 填 tag，比如 `hw1-demo`，标题随意
3. 把视频文件**拖进** "Attach binaries by dropping them here"，等上传完
4. 点 **Publish release**
5. 右键附件链接复制，形如：
   ```
   https://github.com/你的用户名/仓库名/releases/download/hw1-demo/demo.mp4
   ```
6. 贴进 README 或作业说明。批改的人点开即可下载/观看。

（2GB 是单文件上限，接近 2GB 上传偏慢但不会被拒。）

### 方案 B：继续走 S3 公开（如果课程要求用 S3）

**整套 S3 公开配置都可在 AWS 控制台（网页）点完成，不用 CLI** —— 控制台用浏览器登录，跟坏掉的 access key 无关：

- S3 → bucket → **Permissions** → 关掉 Block Public Access 的两个 policy 相关开关
- 同页 → **Bucket policy** → 贴 policy
- **Objects** 标签页点文件 → 看到 **Object URL**，即公开链接

（CLI 凭证报错迟早得修，但单为这次作业，方案 A 更快。）

---

## 10. 实操后仍 AccessDenied —— 前缀对不上

**现象：** 已关 Block Public Access 两个开关、已贴 policy，incognito 打开
`https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/list_set_map.mov`
仍报 `AccessDenied`。

**原因：** 文件叫 **`list_set_map.mov`，直接在 bucket 根目录**，但 policy 只对 `demos/*` 前缀生效。两者对不上，所以被拒。

policy 里写的是 `.../demos/*`，意思是"只放开 `demos/` 文件夹"。而 `list_set_map.mov` 在根目录，不在 `demos/` 下，匹配不到。

### 办法 1：改 policy 匹配该文件（最快）

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadDemos",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an/list_set_map.mov"
    }
  ]
}
```

`Resource` 结尾从 `/demos/*` 改成 `/list_set_map.mov`。存盘后用 incognito 刷新。

想让所有根目录文件公开可用 `/*`（范围更大，自行掂量）：
```
"Resource": "arn:aws:s3:::rt-mock-XXXXXXXXXXXX-us-west-2-an/*"
```

### 办法 2：把文件挪进 demos/（policy 不动）

S3 控制台 → Objects → 选中 `list_set_map.mov` → **Actions** → **Move**，目标 `demos/`。之后链接：
```
https://rt-mock-XXXXXXXXXXXX-us-west-2-an.s3.us-west-2.amazonaws.com/demos/list_set_map.mov
```

### 改完记得

1. S3 → Permissions → Bucket policy 存盘
2. **新开 incognito 窗口**刷新（旧窗口可能缓存了 AccessDenied）

**小提醒：** `.mov` 在浏览器里不一定能直接播放（取决于浏览器对 QuickTime 的支持），但**下载肯定没问题**。想点开就在线播放，转成 `.mp4` 兼容性最好。交作业的话 `.mov` 完全够用。
