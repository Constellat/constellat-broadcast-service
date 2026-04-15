# Python 包打包并上传到 PyPI 完整流程

## 1. 项目结构

```
my_package/
├── my_package/
│   ├── __init__.py
│   └── _core.py      # 核心实现，通过 __all__ 控制导出
├── setup.py          # 或 pyproject.toml（现代方式）
├── README.md
├── LICENSE
└── MANIFEST.in       # 可选，控制哪些文件被打包
```

### 推荐的代码组织方式

将核心逻辑写在 `_core.py` 中，通过 `__all__` 声明对外导出的名称：

```python
# _core.py
class YourClass:
    pass

singleton_instance = YourClass()
__all__ = ['singleton_instance']
```

在 `__init__.py` 中统一导入，让用户可以直接 `from my_package import singleton_instance`：

```python
# __init__.py
from ._core import *
```

> 好处：将实现细节隐藏在 `_core.py`，用户无需关心内部模块结构，导入路径更简洁。

---

## 2. 编写 `setup.py`

```python
from setuptools import setup, find_packages

setup(
    name='your-package-name',          # PyPI 上的包名，全局唯一
    version='0.1.0',
    author='Your Name',
    author_email='your@email.com',
    description='A short description',
    long_description=open('README.md').read(),
    long_description_content_type='text/markdown',
    url='https://github.com/yourname/your-repo',
    packages=find_packages(),
    python_requires='>=3.6',
    install_requires=[
        # 填写依赖，如 'requests>=2.0'
    ],
    classifiers=[
        'Programming Language :: Python :: 3',
        'License :: OSI Approved :: MIT License',
        'Operating System :: OS Independent',
    ],
)
```

> 现代项目推荐使用 `pyproject.toml` + `hatchling` / `flit` / `poetry` 替代 `setup.py`。

---

## 3. 安装打包工具

```bash
pip install build twine
```

- **build**：生成源码包（sdist）和 wheel 包
- **twine**：安全地上传包到 PyPI

---

## 4. 构建包

```bash
# 同时生成 sdist（.tar.gz）和 wheel（.whl）
python -m build

# 旧方式（不推荐）
python setup.py sdist bdist_wheel
```

构建产物位于 `dist/` 目录：

```
dist/
├── your_package-0.1.0.tar.gz
└── your_package-0.1.0-py3-none-any.whl
```

---

## 5. 注册 PyPI 账号

1. 前往 [https://pypi.org/account/register/](https://pypi.org/account/register/) 注册账号
2. 在账号设置中生成 **API Token**（推荐，比用户名/密码更安全）
3. 在 `~/.pypirc` 中配置认证信息：

```ini
[pypi]
username = __token__
password = pypi-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## 6. 先用 TestPyPI 验证（推荐）

```bash
# 上传到测试环境
twine upload --repository testpypi dist/*

# 从测试环境安装验证
pip install --index-url https://test.pypi.org/simple/ your-package-name
```

---

## 7. 上传到正式 PyPI

```bash
# 上传全部产物
twine upload dist/*

# 或上传指定文件
twine upload dist/your_package-0.1.0.tar.gz
```

---

## 8. 验证安装

```bash
pip install your-package-name
```

---

## 9. 更新版本流程

1. 修改 `setup.py` 中的 `version` 字段（遵循语义化版本 SemVer：`MAJOR.MINOR.PATCH`）
2. 重新构建：`python -m build`
3. 上传新版本：`twine upload dist/*`

> PyPI 不允许覆盖已上传的同名同版本文件，每次上传必须更新版本号。

---

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| 包名已被占用 | 在 PyPI 搜索确认后换一个唯一名称 |
| `twine check` 报错 | 检查 README.md 格式，确保 `long_description_content_type` 正确 |
| 上传后找不到包 | PyPI 索引有延迟，等几分钟再搜索 |
| 依赖安装失败 | 检查 `install_requires` 中的版本约束 |
