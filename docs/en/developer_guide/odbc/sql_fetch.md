# SQLFetch<a name="ZH-CN_TOPIC_0242371447"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-22T03:49:30.352Z pushedAt=2026-07-22T03:53:22.989Z -->

## Function<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_se050a9bc14bc47f1bb3b6d4650b37d67"></a>

Fetches the next rowset of data from a result set and returns the data of all bound columns.

## Prototype<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_s9c799d0d6e0a4ecb91a800e6acdd31dd"></a>

```c
SQLRETURN SQLFetch(SQLHSTMT    StatementHandle);
```

## Parameters<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_s19f2bee9a4f2459e9496155f5e29600b"></a>

**Table 1** SQLFetch parameters

<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_t41ad0c45de864fcc8fbc4c21618c94b6"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_r58f5f6f4c39542a6a3be76f92fef9c87"><th class="cellrowborder" valign="top" width="23.119999999999997%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a87934542f19744c4b9194eab9bc38b1e"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a87934542f19744c4b9194eab9bc38b1e"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a87934542f19744c4b9194eab9bc38b1e"></a><strong id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_ad7d886035a98477c9f64831910660d13"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_ad7d886035a98477c9f64831910660d13"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_ad7d886035a98477c9f64831910660d13"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.88000000000001%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_aa79b3721c81f461e8d8741820c91b222"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_aa79b3721c81f461e8d8741820c91b222"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_aa79b3721c81f461e8d8741820c91b222"></a><strong id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_zh-cn_topic_0058965190_b280121351440"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_zh-cn_topic_0058965190_b280121351440"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_zh-cn_topic_0058965190_b280121351440"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_ra4493268c6ea4e09918f7151b802440d"><td class="cellrowborder" valign="top" width="23.119999999999997%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a82994056abe64f8cbc98a3d2110ba6c8"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a82994056abe64f8cbc98a3d2110ba6c8"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a82994056abe64f8cbc98a3d2110ba6c8"></a>StatementHandle</p></td>
<td class="cellrowborder" valign="top" width="76.88000000000001%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a43c4cf7648bb45a2b64938ba3da60384"><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a43c4cf7648bb45a2b64938ba3da60384"></a><a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_a43c4cf7648bb45a2b64938ba3da60384"></a>Statement handle, obtained through SQLAllocHandle.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_s08aa46acb7f04acf807438d3b1f0df6f"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: Row data failed to be fetched from the result set.

- SQL_NO_DATA: The SQL statement does not return a result set.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272893_zh-cn_topic_0237120423_zh-cn_topic_0059777989_s297ce23d9b4d4e6ab0f5888390604f41"></a>

See: [Example](example.md)
