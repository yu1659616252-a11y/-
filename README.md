# requests + pytest + Allure API automation demo

这是一个可以直接接入 Jenkins 的 Python 接口自动化测试 demo。默认使用内置 mock 接口，方便本地和 Jenkins 在没有公网权限时也能跑通；真实接入时把 `API_BASE_URL` 改成你的测试环境地址即可。

## 目录结构

```text
.
├── api/
│   └── client.py              # requests 二次封装，自动写入 Allure 附件
├── tests/
│   ├── conftest.py            # pytest 参数、公共 fixture、内置 mock 服务
│   └── test_httpbin_api.py    # 示例接口测试
├── Jenkinsfile                # Jenkins Pipeline
├── pytest.ini                 # pytest 默认配置
└── requirements.txt           # Python 依赖
```

## 本地运行

1. 创建虚拟环境

```bash
python -m venv .venv
```

2. 激活虚拟环境

Windows:

```bat
.venv\Scripts\activate
```

macOS / Linux:

```bash
source .venv/bin/activate
```

3. 安装依赖

```bash
pip install -r requirements.txt
```

4. 运行测试并生成 Allure 原始结果

```bash
pytest
```

只运行冒烟用例：

```bash
pytest -m smoke
```

指定真实测试环境地址：

```bash
pytest --base-url=https://your-test-env.example.com
```

也可以用环境变量：

Windows:

```bat
set API_BASE_URL=https://your-test-env.example.com
pytest
```

macOS / Linux:

```bash
export API_BASE_URL=https://your-test-env.example.com
pytest
```

## 查看 Allure 报告

本地需要先安装 Allure Commandline，然后执行：

```bash
allure serve allure-results
```

如果只是生成静态报告：

```bash
allure generate allure-results -o allure-report --clean
```

## 接入真实项目

1. 保留 `api/client.py`，作为统一请求入口。
2. 在 `tests/conftest.py` 中增加登录、token、数据库连接等公共 fixture。
3. 在测试用例中把 `/get`、`/post` 替换成你的业务接口路径。
4. Jenkins 构建时通过参数 `API_BASE_URL` 控制测试环境，例如测试环境、预发环境。
5. 用 `PYTEST_MARK` 控制执行范围，例如 `smoke`、`api`、`regression`。

## Jenkins 准备工作

1. Jenkins 节点安装 Python 3。
2. Jenkins 安装 Allure 插件。
3. Jenkins 全局工具配置中增加 Allure Commandline，名称建议填 `Allure`。
4. 代码仓库根目录放入本项目的 `Jenkinsfile`。
5. 新建 Pipeline 任务，选择从 SCM 读取 `Jenkinsfile`。
6. 构建时填写：

```text
API_BASE_URL = https://your-test-env.example.com
PYTEST_MARK  = smoke
```

如果只是验证 demo，`API_BASE_URL` 保持默认值 `mock` 即可。构建完成后，Jenkins 页面会展示 Allure Report 入口。
