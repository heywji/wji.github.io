# WJI Local Rules Prompt (Sanitized)

来源：`/home/wjiwji/projects/ai_prompts/wji.txt`

说明：

- 原始文件包含本地数据库连接口令
- 这里保留规则思路，不保留真实主机、用户名、密码
- 如果后续要公开复用，应该继续把所有内网地址和个人偏好项改成占位符

```yaml
---
globs:
alwaysApply: false
---
```

```text
Hi AI, 请务必遵守以下的操作规范：
1. 任务完成后，把本次操作记录写入一个本地数据库表。
   连接命令示意：
   mysql -u<DB_USER> -p<REDACTED_PASSWORD> -h<REDACTED_HOST> <DB_NAME>
   记录字段示意：
   ID, NAME, PATH, OPERATE, BUGS, TIPS

2. 编译产物只保留最终成功的 bin 文件，清理其他临时垃圾文件。

3. 修改任何文件前先做 bak 备份，再继续修改。

4. 访问 NAS 或内网主机时，使用正确的远端用户名，不要沿用本机默认用户。

5. git push 时注意使用正确的远端别名，不要默认写死 origin。

6. 任务完成后可以通过 notify-send 发送桌面通知，汇报运行过程和结果。
```
