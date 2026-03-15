# 5-Step Smoke Test

Run these in your agent chat environment.

## 1) Search

Prompt:

```text
在收藏库里搜索我最近保存的文章，给我前 3 条标题、原文链接和记录ID。
```

Pass criteria:

- returns 1-3 records,
- each includes title + original URL + record id.

## 2) Fetch one record

Prompt:

```text
把第 1 条的详细信息给我：标题、标签、总结、原文链接、记录ID。
```

Pass criteria:

- details are readable,
- includes original link and record id.

## 3) Create test record

Prompt:

```text
保存这篇文章到稍后读：https://example.com/test-article
标签：测试, smoke-test
备注：这是冒烟测试数据
```

Pass criteria:

- new record created in MongoDB,
- returns record id and original link.

## 4) Update test record

Prompt:

```text
把刚才 smoke-test 那条记录状态改成 Done，并追加备注：更新成功。
```

Pass criteria:

- status and note both updated.

## 5) Query back

Prompt:

```text
刚才那篇测试文章主要说了什么？把原文链接和记录ID都给我。
```

Pass criteria:

- returns summary or fallback explanation,
- returns original link and record id.
