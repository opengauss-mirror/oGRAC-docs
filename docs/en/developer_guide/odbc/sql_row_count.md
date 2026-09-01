# SQLRowCount<a name="ZH-CN_TOPIC_0242371452"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-24T03:53:34.965Z pushedAt=2026-07-24T03:54:48.588Z -->

## Function<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_s0d00013620f7443fb1bb6deca04314c8"></a>

Obtains the number of rows affected by the most recently executed SQL statement.

## Prototype<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_s195bb50934b04608b371fd8329ab099b"></a>

```c
SQLRETURN SQLRowCount(SQLHSTMT      StatementHandle,    
                      SQLLEN        *RowCountPtr);
```

## Parameters<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_s258e6631a7e74be6a4c3afe4dcc3bbc0"></a>

**Table 1** SQLRowCount parameters

<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_t33ff90a7a6b74897ab86f9751d4c5275"></a>
<table><thead align="left"><tr id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_rc5971abfc7ad454b82d4ec1fa4a5a3d2"><th class="cellrowborder" valign="top" width="23.27%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a99b3a615915a4e6fadf9098f3ca827f3"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a99b3a615915a4e6fadf9098f3ca827f3"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a99b3a615915a4e6fadf9098f3ca827f3"></a><strong id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a6a69571d09d44f4f925ba0681ef0ee34"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a6a69571d09d44f4f925ba0681ef0ee34"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a6a69571d09d44f4f925ba0681ef0ee34"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.73%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a76ec10478d09482189849545c24624e7"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a76ec10478d09482189849545c24624e7"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a76ec10478d09482189849545c24624e7"></a><strong id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ac70d7a582bc74080bf82d7b324923371"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ac70d7a582bc74080bf82d7b324923371"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ac70d7a582bc74080bf82d7b324923371"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_r1efc1d51b2d142d4822ebd4a97c02217"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a9e4c5c491a90471bacc88a38be62bde1"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a9e4c5c491a90471bacc88a38be62bde1"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a9e4c5c491a90471bacc88a38be62bde1"></a>StatementHandle</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ab23b5b1aa19a4cd69b0fa82ee6098ac3"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ab23b5b1aa19a4cd69b0fa82ee6098ac3"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ab23b5b1aa19a4cd69b0fa82ee6098ac3"></a>Statement handle.</p></td>
</tr>
<tr id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_rf6322af53ba14f53adb5cc47a1243cb6"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a1f11f77d7f3141aca8078f349dcdcfab"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a1f11f77d7f3141aca8078f349dcdcfab"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_a1f11f77d7f3141aca8078f349dcdcfab"></a>RowCountPtr</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ad725bfcd68d64d5287f3e274ae3de05a"><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ad725bfcd68d64d5287f3e274ae3de05a"></a><a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_ad725bfcd68d64d5287f3e274ae3de05a"></a>Pointer to a buffer. Upon successful execution, the number of affected rows is written to the address pointed to by this buffer.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_s5a0422bedc334a4f88a22c60cb4ba293"></a>

- SQL\_SUCCESS: The call is successful.
- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.
- SQL_ERROR: The execution failed.
- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272898_zh-cn_topic_0237120428_zh-cn_topic_0059779125_sbf766eb45a5e442094a561b3036a4205"></a>

See: [Example](example.md)
