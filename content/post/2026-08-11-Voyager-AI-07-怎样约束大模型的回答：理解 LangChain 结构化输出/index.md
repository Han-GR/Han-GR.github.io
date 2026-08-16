---
title: 怎样约束大模型的回答：理解 LangChain 结构化输出
description:
slug: 2026-08-11-Voyager-AI-07-怎样约束大模型的回答：理解 LangChain 结构化输出
date: 2026-08-11
image: json-schema.jpg
categories:
  - AI&数据
  - 后端
tags:
  - LangChain
  - Pydantic
  - 结构化输出
  - Voyager-AI
---

## “请返回 JSON”还不够

在 Prompt 末尾写“请严格返回 JSON”，模型仍可能改字段名、漏字段，或者把数字写成带单位的文字。JSON 能解析，也不代表业务上可用。

Voyager-AI 把约束分成三层：Prompt 说明任务，Schema 规定数据形状，业务校验检查字段之间是否合理。

```mermaid
flowchart LR
    P["Prompt<br/>说明要做什么"] --> M["大模型"]
    M --> S["Schema<br/>检查字段和类型"]
    S --> V["业务校验<br/>检查数据关系"]
    V --> R["可使用的结果"]
```

## 用 Pydantic 编写 Schema

Schema 是数据结构约定。预算结果可以写成：

```python
class BudgetCategory(BaseModel):
    category: str
    amount_minor: int = Field(ge=0)


class BudgetResult(BaseModel):
    total_minor: int = Field(gt=0)
    categories: list[BudgetCategory]
```

Pydantic 会检查 `total_minor` 是正整数、分类金额不是负数、categories 是列表。`BaseModel` 还提供 `model_validate`，可以把真实数据放进去验证。

## LangChain 怎样绑定结构

```python
def invoke_structured(model, schema, payload, *, prompt):
    try:
        runnable = model.with_structured_output(schema)
        result = runnable.invoke(prompt)
    except (ValidationError, OutputParserException) as error:
        raise AgentOutputError() from error

    return schema.model_validate(result)
```

`with_structured_output(schema)` 返回一个 Runnable。Runnable 可以理解成“具有统一 invoke 调用方式的可执行组件”。LangChain 会根据模型能力请求结构化结果，再由本地 Pydantic 复查。

解析错误被转换为 `AgentOutputError`，模型连接失败则是 `AgentModelError`。区分两者后，排错时能知道是服务不可用，还是回答不符合结构。

## 类型正确仍然可能算错账

下面的数据完全符合 Schema：

```json
{
  "total_minor": 120000,
  "categories": [
    {"category": "transport", "amount_minor": 36000},
    {"category": "lodging", "amount_minor": 42000}
  ]
}
```

但分类合计只有 78000。Pydantic 只知道每个字段类型正确，不知道业务上必须对账。

因此最终计划还会执行：

```python
category_total = sum(
    item["amount_minor"]
    for item in budget["categories"]
)

if category_total != budget["total_minor"]:
    raise InvalidTravelPlanError()
```

同一层校验还要求 Reviewer 已批准、final_plan 有足够内容。结构化输出减少模型自由度，业务校验继续保护真正重要的规则。

## 总结

约束模型不能只靠一句更强硬的 Prompt。Prompt、Schema 和业务校验解决的是不同问题：一个表达意图，一个检查形状，一个判断数据关系。

LangChain 的结构化输出让不同模型以相近方式绑定 Schema，Pydantic 在本地提供可重复的验证。模型负责生成不确定内容，代码负责守住可以确定的边界。
