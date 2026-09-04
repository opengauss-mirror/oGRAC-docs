# SQLSetStmtAttr<a name="ZH-CN_TOPIC_0242371457"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-24T11:18:07.841Z pushedAt=2026-07-24T11:27:29.406Z -->

## Function<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_s289df741daf0491095bd3a6eeb7c3397"></a>

Sets statement attributes.

## Prototype<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_s64dcb5b7dc82403a86dfb0b07729723d"></a>

```c
SQLRETURN SQLSetStmtAttr(SQLHSTMT      StatementHandle,
                         SQLINTEGER    Attribute,    
                         SQLPOINTER    ValuePtr,     
                         SQLINTEGER    StringLength);
```

## Parameters<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_s93d4d32e284e4e32930c5a9c033b6be3"></a>

**Table 1** SQLSetStmtAttr parameters

<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_tc7d6fc98e0094dfdb438ab59460d206d"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_rb323017d53364460aff9e3ccc9832912"><th class="cellrowborder" valign="top" width="23.27%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ab365a224aa1841d39743aef2954b4302"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ab365a224aa1841d39743aef2954b4302"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ab365a224aa1841d39743aef2954b4302"></a><strong id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a53341033a30144a38e6737be1d8cf505"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a53341033a30144a38e6737be1d8cf505"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a53341033a30144a38e6737be1d8cf505"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.73%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a2e4dd518264f4240a2ce290ed00efdf4"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a2e4dd518264f4240a2ce290ed00efdf4"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a2e4dd518264f4240a2ce290ed00efdf4"></a><strong id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_zh-cn_topic_0058965267_b552450314741"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_zh-cn_topic_0058965267_b552450314741"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_zh-cn_topic_0058965267_b552450314741"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_r13b05b59c65f4a8fb1f61880b8a41c0e"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ac3f3037f05bb4b6a98ea3fe0a2291a46"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ac3f3037f05bb4b6a98ea3fe0a2291a46"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_ac3f3037f05bb4b6a98ea3fe0a2291a46"></a>StatementHandle</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a6f3d479b15a64c6bbdf1fe3cc9c6a57c"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a6f3d479b15a64c6bbdf1fe3cc9c6a57c"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a6f3d479b15a64c6bbdf1fe3cc9c6a57c"></a>Statement handle.</p></td>
</tr>
<tr id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_r3191b8b205a145c9b4388d6bf3b1482e"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a7200a79355ae424c82fc839022a66d25"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a7200a79355ae424c82fc839022a66d25"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a7200a79355ae424c82fc839022a66d25"></a>Attribute</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a73f6bb39ae4140b4b1c3fe6f09a9d426"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a73f6bb39ae4140b4b1c3fe6f09a9d426"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a73f6bb39ae4140b4b1c3fe6f09a9d426"></a>Statement attribute to be set. It can be one of the following values:</p>
<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"></a><ul id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"><li>SQL_ATTR_QUERY_TIMEOUT: Specifies the SQL statement execution timeout.</li><li>SQL_ATTR_PARAMSET_SIZE: Specifies the number of parameter groups contained in a parameter set during batch execution.</li><li>SQL_ATTR_ROWS_FETCHED_PTR: Obtains the actual number of rows fetched.</li><li>SQL_ATTR_ROW_ARRAY_SIZE: Sets the maximum number of rows for batch fetching.</li></ul>
<p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a>If the attribute to be set is not within the above range, the driver returns SQL_ERROR.</p></td>
</tr>
<tr id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_r0c1b3c0fafbf4d7f8dcad213939dd18d"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a63582017c4844cdbb0164922a51fc64e"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a63582017c4844cdbb0164922a51fc64e"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a63582017c4844cdbb0164922a51fc64e"></a>ValuePtr</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_aa0bbb1c4b3ce4d938964e62c473ab843"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_aa0bbb1c4b3ce4d938964e62c473ab843"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_aa0bbb1c4b3ce4d938964e62c473ab843"></a>Points to the value of the corresponding attribute. Depending on the attribute value, ValuePtr can be a 32-bit unsigned integer value, or points to a null-terminated string, a binary buffer, or a driver-defined value.</p></td>
</tr>
<tr id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_rdc1f6c194f1d4e57bd88d21d62779354"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a0a98d98a9abb4a35970bd71fba0accec"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a0a98d98a9abb4a35970bd71fba0accec"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a0a98d98a9abb4a35970bd71fba0accec"></a>StringLength</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a10f4bbdff26b4974891e121722845a0b"><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a10f4bbdff26b4974891e121722845a0b"></a><a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_a10f4bbdff26b4974891e121722845a0b"></a>If ValuePtr points to a string or a binary buffer, this parameter is the length of *ValuePtr. If ValuePtr points to an integer value, StringLength is ignored.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_s25d37c96151c49ef8117dc53bda2bf2c"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: The statement attribute failed to be set.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272903_zh-cn_topic_0237120433_zh-cn_topic_0059777949_s14d206561091447bbb06bac48d8fee66"></a>

See: [Example](example.md)
