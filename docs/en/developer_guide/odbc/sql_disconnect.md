# SQLDisconnect<a name="ZH-CN_TOPIC_0242371444"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-22T03:37:20.609Z pushedAt=2026-07-22T03:38:16.541Z -->

## Function<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_sf1f063f9cda741e3bddd12ffff9982ea"></a>

Closes the connection associated with a specific connection handle.

## Prototype<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_sa94159f087b8453d91742bd037a4dd23"></a>

```c
SQLRETURN SQLDisconnect(SQLHDBC    ConnectionHandle);
```

## Parameters<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_sed5e3f43dc554fcf87bcc0319da72ef0"></a>

**Table 1** SQLDisconnect parameters

<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_tf92662a862a94bb8a2ea829cbf5f90af"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_r28fb5700eaa54698a4340424276b87f4"><th class="cellrowborder" valign="top" width="24.5%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_ac9627e06109f453bac21c00c96534d27"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_ac9627e06109f453bac21c00c96534d27"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_ac9627e06109f453bac21c00c96534d27"></a><strong id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a9e3d1cc49cc849e6a9dbfa5f277cc791"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a9e3d1cc49cc849e6a9dbfa5f277cc791"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a9e3d1cc49cc849e6a9dbfa5f277cc791"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="75.5%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a514195d887524d04b3ecee1f113764cb"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a514195d887524d04b3ecee1f113764cb"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a514195d887524d04b3ecee1f113764cb"></a><strong id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_zh-cn_topic_0058965244_b447479391436"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_zh-cn_topic_0058965244_b447479391436"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_zh-cn_topic_0058965244_b447479391436"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_r0a0928672b2646d0a261d3ef7e26b596"><td class="cellrowborder" valign="top" width="24.5%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a80896ed0e4cc4f3fb5351a168f738731"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a80896ed0e4cc4f3fb5351a168f738731"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a80896ed0e4cc4f3fb5351a168f738731"></a>ConnectionHandle</p></td>
<td class="cellrowborder" valign="top" width="75.5%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a701be46576314b7491c2cde0e48ada5b"><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a701be46576314b7491c2cde0e48ada5b"></a><a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_a701be46576314b7491c2cde0e48ada5b"></a>Connection handle, obtained through SQLAllocHandle.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_sec718cedec224638b6f21c68a66c165d"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR：The connection failed to be released.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272890_zh-cn_topic_0237120420_zh-cn_topic_0059778556_sb7797f4e64534d1f85c319d5433804d4"></a>

See: [Example](example.md)
