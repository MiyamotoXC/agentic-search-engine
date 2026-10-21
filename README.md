# Agentic Search Engine（agentic-search-engine）

> 用 **LLM 自主工具调用（Agentic Search）替代传统 RAG** 的企业级多源搜索系统：
> 模型自己决定调用哪些检索工具、以什么顺序、何时停止，而不是把"向量召回 + 重排"写死成一条固定管线。
> 支持 **可观测的搜索轨迹**（`search_with_trace` / `search_stream`），便于排查「为什么搜不到 / 为什么搜到这个」。

> 作者：[@MiyamotoXC](https://github.com/MiyamotoXC) ｜ 许可证：MIT ｜ 欢迎 Star / Fork 二次开发。

## 一、为什么不用传统 RAG

传统 RAG 把检索写死成「向量召回 → 重排 → 拼上下文」，遇到结构化数据、代码、跨系统查询就力不从心。本项目把搜索拆成一组**可被 LLM 自由编排的工具**，由模型在运行时决定：

- 要查结构化数据（员工 / 订单 / 项目）→ 调数据库引擎
- 要查长篇文档语义 → 调向量库引擎
- 要查政策条款关键词 → 调关键词引擎
- 要查内部代码 → 调代码检索引擎
- 要调业务系统 → 调企业 SDK 引擎

## 二、架构

```
agentic-search-engine/
├── main.py                    # CLI 入口：单轮 / 带轨迹 / 流式
├── app.py                     # Streamlit 交互界面
├── config.py                  # 配置中心（环境变量，见下）
├── core/
│   ├── agent.py               # AgenticSearchAgent：工具注册 / 执行 / 系统提示 / 多轮搜索
│   ├── llm.py                 # LLM 调用封装
│   └── logger.py              # 轨迹与日志
├── engines/                   # 五类检索后端（均实现统一接口）
│   ├── vector_db.py           # Chroma 向量检索
│   ├── database.py            # SQLite 结构化查询
│   ├── keyword_search.py      # Whoosh 中文关键词（jieba 分词）
│   ├── code_search.py         # 代码语义检索
│   └── enterprise_sdk.py      # 企业系统 SDK 适配
├── data/
│   ├── code_repo/             # 被检索的示例代码库（FastAPI 项目，含完整 src/ 与测试）
│   └── db/                    # 运行时生成的 SQLite（由 seed_data 初始化）
├── seed_data.py / seed_data_large.py  # 种子数据（员工 / 政策 / 技术文章）
└── requirements.txt
```

## 三、快速开始

```bash
pip install -r requirements.txt

# 1) 准备环境变量（复制模板后填入你的 LLM Key）
cp .env.example .env        # 填写 DASHSCOPE_API_KEY 等

# 2) 初始化种子数据与索引
python seed_data.py         # 写入 data/db + 构建关键词/向量索引

# 3) 运行
python main.py                       # 交互式搜索
python app.py                        # 启动 Streamlit 界面
```

## 四、配置（环境变量，均在 `.env` 中，不入库）

| 变量 | 说明 |
|---|---|
| `DASHSCOPE_API_KEY` | LLM / Embedding 调用密钥（必需） |
| `LLM_BASE_URL` | 模型服务地址（可选，默认官方地址） |
| `LLM_MODEL` | 模型名（可选，默认 qwen 系列） |
| `MAX_SEARCH_ROUNDS` | 单轮最多工具调用轮数（默认 20） |

仓库内**不含任何真实密钥**；`.env.example` 仅为占位模板，真实 `.env` 已被 `.gitignore` 忽略。

## 五、核心能力

- **工具自编排**：模型根据问题自主选择并组合检索工具，无需硬编码检索顺序
- **多引擎协同**：向量 / SQL / 关键词 / 代码 / 企业 SDK 五类后端统一接口
- **可观测轨迹**：`search_with_trace` 返回每一步调用的工具、参数与结果
- **流式输出**：`search_stream` 边搜边答
- **中文友好**：关键词引擎内置 jieba 分词

## 六、能力边界（诚实说明）

- 检索质量上限取决于各后端的数据与索引；首次运行需先执行 `seed_data.py` 构建索引
- 工具执行失败有兜底日志，但不会无限重试（受 `MAX_SEARCH_ROUNDS` 约束）
- 企业 SDK 引擎以适配层形式提供，接入真实系统需补充凭证与接口

## 七、License

代码以 MIT 发布。
