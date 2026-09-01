# SQLExecute<a name="ZH-CN_TOPIC_0242371446"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-22T03:46:59.076Z pushedAt=2026-07-22T03:48:41.484Z -->

## Function<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_sab94d410ef5f4a509e8d2881ec7373e7"></a>

If parameter markers exist in a statement, SQLExecute executes a prepared SQL statement using the current values of the parameter markers.

## Prototype<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_se7599e2d01c7423e9560a92d77e7f485"></a>

```c
SQLRETURN SQLExecute(SQLHSTMT    StatementHandle);
```

## Parameters<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_s2f725c8885a54047b1c851581a5cfb20"></a>

**Table 1** SQLExecute parameters

<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_t273b6621863d447d83d4abfa8022d0d8"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_r307958bad12a44e392db7fbb6af50301"><th class="cellrowborder" valign="top" width="23.580000000000002%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a2bf3d610f4ba41d3ba42d42367ef7169"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a2bf3d610f4ba41d3ba42d42367ef7169"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a2bf3d610f4ba41d3ba42d42367ef7169"></a><strong id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_af4475977916c484c958035d7e218a430"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_af4475977916c484c958035d7e218a430"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_af4475977916c484c958035d7e218a430"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.42%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_add57a1dc45d74987a5c7310f1484e1fd"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_add57a1dc45d74987a5c7310f1484e1fd"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_add57a1dc45d74987a5c7310f1484e1fd"></a><strong id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_zh-cn_topic_0058965164_b410532714330"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_zh-cn_topic_0058965164_b410532714330"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_zh-cn_topic_0058965164_b410532714330"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_r1710d254c4ea46a4bb7ff750906e746c"><td class="cellrowborder" valign="top" width="23.580000000000002%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a7c3ace89f78b43a49610f0469c371ee1"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a7c3ace89f78b43a49610f0469c371ee1"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a7c3ace89f78b43a49610f0469c371ee1"></a>StatementHandle</p></td>
<td class="cellrowborder" valign="top" width="76.42%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a92272c99dc784efebbe3125e4ecf3186"><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a92272c99dc784efebbe3125e4ecf3186"></a><a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_a92272c99dc784efebbe3125e4ecf3186"></a>Statement handle of the statement to be executed.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_s46aaf47ea8e24486b39259cd50dacdf8"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: The statement execution failed.

- SQL_NO_DATA: The SQL statement does not return a result set.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272892_zh-cn_topic_0237120422_zh-cn_topic_0059778182_s224798c3974741f3976659ed7d6de926"></a>

See: [Example](example.md)
