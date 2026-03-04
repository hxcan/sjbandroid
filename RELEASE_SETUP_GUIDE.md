# 📱 正式发布流程配置指南

本文档说明如何在 **sjbandroid** 项目中配置正式版的 Release Build 流程。

---

## 🔐 GitHub Secrets 配置

在 [Settings > Secrets and variables > Actions](https://github.com/hxcan/sjbandroid/settings/secrets/actions) 添加以下 Secrets：

| Secret Key | 描述 | 示例值 |
|------------|------|--------|
| `RELEASE_KEYSTORE_BASE64` | 正式签名证书的 Base64 编码内容 | `MIIB...==` |
| `RELEASE_STORE_PASSWORD` | 密钥库密码 | `your_store_pass` |
| `RELEASE_KEY_ALIAS` | 密钥别名 (alias) | `release_key` |
| `RELEASE_KEY_PASSWORD` | 密钥密码 | `your_key_pass` |
| `GITHUB_TOKEN` | (自动生成) 用于上传 Release 资产 | - |

---

## 🛠️ 生成正式证书的步骤

### 1. 使用 keytool 创建 keystore

```bash
keytool -genkeypair \
  -v \
  -keystore release-key.keystore \
  -storepass your_store_password \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias release_key \
  -keypass your_key_password \
  -dname "CN=YourName, OU=Dept, O=Organization, L=City, ST=State, C=Country"
```

### 2. 转换为 Base64

```bash
base64 -w 0 release-key.keystore > release-key.txt
cat release-key.txt
# 复制输出结果作为 RELEASE_KEYSTORE_BASE64 的值
```

### 3. 填入 GitHub Secrets

按照上文表格配置每个 Secret 的值。

---

## 🚀 触发发布的方式

### 方式 1：创建 Release（推荐）

1. 在 GitHub 页面点击 **"Releases"** → **"Draft a new release"**
2. 输入标签名（如 `v1.0.0`）
3. 选择分支 `master`
4. 填写版本说明
5. 点击 **"Publish release"**

### 方式 2：推送标签

```bash
git tag v1.0.0
git push origin v1.0.0
```

工作流程会自动检测到 `release` 事件并执行签名构建。

---

## ✅ 验证构建结果

构建成功后会看到：

1. **Actions** 页面显示成功运行的工作流
2. **Artifacts** 可下载 `sjbandroid-release-apk`
3. **Release** 页面自动附上签名的 APK 文件

---

## 🔒 安全建议

- ⚠️ **永远不要**将 `.keystore` 文件提交到代码仓库
- ⚠️ 定期轮换密钥密码和库密码
- ✅ 限制对 Secrets 的访问权限
- ✅ 使用 CI/CD 日志监控异常活动

---

## ❓ 常见问题

### Q: 为什么 Release Build 失败了？
A: 检查以下几点：
- Secrets 是否正确配置且值没有多余的空格
- 证书密码与 Alias 匹配
- Gradle 参数传递正确

### Q: 如何测试而不影响正式环境？
A: 可以暂时使用 `debug.keystore` 进行测试（见 CI 中的 fallback 逻辑），但不应用于生产。

---

## 📝 参考

- 主任务：[#4494](https://glzquuktdzuk.gzg.sealos.run/issues/4494)
- 关联任务：[#4488](https://glzquuktdzuk.gzg.sealos.run/issues/4488) (持续集成工作流开发)