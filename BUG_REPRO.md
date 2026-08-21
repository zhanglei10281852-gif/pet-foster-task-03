# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

运营把一个客户账号改成禁用后，这个客户浏览器里原来已经签发的 token 仍然能访问宠物和订单接口，只有重新登录才会被拒绝。本次只做诊断，不改代码；生产代码、测试和配置均保留现状。请复现并解释账号状态为何没有作用到既有会话，指出认证链放行的位置和完整因果过程。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-03
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-03.git
- parent SHA：4e84073566e55287ff330f6e7ffe817b2631e0bc

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-03.git bug-repro
cd bug-repro
git checkout --detach 4e84073566e55287ff330f6e7ffe817b2631e0bc
go test ./internal/pet -run ^TestAnnotationDisabledAccountSessionRejected$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationDisabledAccountSessionRejected$ -count=1
--- FAIL: TestAnnotationDisabledAccountSessionRejected (0.25s)
    annotation_pet_behavior_test.go:105: disabled user token error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.250s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationDisabledAccountSessionRejected$ -count=1
--- FAIL: TestAnnotationDisabledAccountSessionRejected (1.07s)
    annotation_pet_behavior_test.go:105: disabled user token error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	1.384s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

诊断必须定位 internal/pet/domain.go 的 User.AllowsSession，并说明该谓词以角色非空代替 Status 判断后，Service.UpdateUser 不撤销旧 token、Service.Authenticate 又继续放行的完整因果链；需以禁用前签发的 token 在禁用后仍能访问接口的定向复现作为证据，调查结束时生产代码、测试和配置保持零改动。
