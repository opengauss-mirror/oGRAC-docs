# SQLExecDirect<a name="ZH-CN_TOPIC_0242371445"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-22T03:42:49.531Z pushedAt=2026-07-22T03:45:58.766Z -->

## Function<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_sa444edc74fad433d8d9beb6eec51f680"></a>

Executes a prepared statement using the current parameter values. For executing a single SQL statement at a time, SQLExecDirect is the fastest execution method.

## Prototype<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_s25b19223d63f497cafffb0216d09a598"></a>

```c
SQLRETURN SQLExecDirect(SQLHSTMT        StatementHandle,
                        SQLCHAR         *StatementText,     
                        SQLINTEGER      TextLength);
```

## Parameters<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_s2f6abd60356142879f48870a38d40612"></a>

**Table 1** SQLExecDirect parameters

<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_t5203fb233a4b4f318bf1c119b929f9d6"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_r0e574f15a22c4800b3941984bafe861d"><th class="cellrowborder" valign="top" width="23.580000000000002%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a475cea2ebe134ead864d9b7d618138ad"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a475cea2ebe134ead864d9b7d618138ad"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a475cea2ebe134ead864d9b7d618138ad"></a><strong id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_adb9e72ead0c04ab08160b5e1f0f808c0"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_adb9e72ead0c04ab08160b5e1f0f808c0"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_adb9e72ead0c04ab08160b5e1f0f808c0"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.42%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_ac729ff85f7cf4d5b9ad03d225349e107"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_ac729ff85f7cf4d5b9ad03d225349e107"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_ac729ff85f7cf4d5b9ad03d225349e107"></a><strong id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a0c4d88ad07e04214aae84d532f665bb0"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a0c4d88ad07e04214aae84d532f665bb0"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a0c4d88ad07e04214aae84d532f665bb0"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_ra661b71731b3473ba5d1aec30971e67d"><td class="cellrowborder" valign="top" width="23.580000000000002%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a86d67aa437f94695b8dc0c0ad5b4d638"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a86d67aa437f94695b8dc0c0ad5b4d638"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a86d67aa437f94695b8dc0c0ad5b4d638"></a>StatementHandle</p></td>
<td class="cellrowborder" valign="top" width="76.42%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a08bc8a467df94afcb2fcccb05616e315"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a08bc8a467df94afcb2fcccb05616e315"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a08bc8a467df94afcb2fcccb05616e315"></a>Statement handle, obtained through SQLAllocHandle.</p></td>
</tr>
<tr id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_r8008d4d61d8d4765bb547320d85a2490"><td class="cellrowborder" valign="top" width="23.580000000000002%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a5ea2be078c354f8ebee10efbf6e270ec"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a5ea2be078c354f8ebee10efbf6e270ec"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a5ea2be078c354f8ebee10efbf6e270ec"></a>StatementText</p></td>
<td class="cellrowborder" valign="top" width="76.42%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a4734d8dd5edd4f78847ab03746074fd0"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a4734d8dd5edd4f78847ab03746074fd0"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a4734d8dd5edd4f78847ab03746074fd0"></a>SQL statement to be executed. Executing multiple statements at once is not supported.</p></td>
</tr>
<tr id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_rec7c2728bd8c4906b45c5fe2fb9880bb"><td class="cellrowborder" valign="top" width="23.580000000000002%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a6810d872caac4ff188126615119b337f"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a6810d872caac4ff188126615119b337f"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a6810d872caac4ff188126615119b337f"></a>TextLength</p></td>
<td class="cellrowborder" valign="top" width="76.42%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a8f930493fe4b425ea391e14c30e0ac9a"><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a8f930493fe4b425ea391e14c30e0ac9a"></a><a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_a8f930493fe4b425ea391e14c30e0ac9a"></a>Length of StatementText.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_s43b67f6e7de04935897f38b7e40a090f"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: The statement execution failed.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

- SQL_NO_DATA: The SQL statement does not return a result set.

## Example<a name="zh-cn_topic_0238272891_zh-cn_topic_0237120421_zh-cn_topic_0059778538_s8fbcb12db6c5494e98bac4b4820ecc3c"></a>

See: [Example](example.md)
