# SQLAllocHandle<a name="ZH-CN_TOPIC_0242371438"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-22T01:55:32.346Z pushedAt=2026-07-22T02:01:10.632Z -->

## Function<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_sa9bd6e837fd4458199f7b677e876f4a9"></a>

Allocates an environment, connection, or statement handle, and can replace SQLAllocEnv, SQLAllocConnect, and SQLAllocStmt.

## Prototype<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_s311ac101dcb74b3abb5b3d2ee9781c2c"></a>

```c
SQLRETURN SQLAllocHandle(SQLSMALLINT   HandleType,    
                         SQLHANDLE     InputHandle,     
                         SQLHANDLE     *OutputHandlePtr);
```

## Parameters<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_s3488a4902c544a86ae94d239d63e21a9"></a>

**Table 1**  SQLAllocHandle parameters

<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_t753453111a55495490879cf0a4b36cf6"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_r91a64b4bccf441acbcb74615b079d506"><th class="cellrowborder" valign="top" width="23.27%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a1cfb85f5e73d458d816d5f5b5e9538cd"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a1cfb85f5e73d458d816d5f5b5e9538cd"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a1cfb85f5e73d458d816d5f5b5e9538cd"></a><strong id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a471452b4ce36463d9fbbad2b44ea3b2d"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a471452b4ce36463d9fbbad2b44ea3b2d"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a471452b4ce36463d9fbbad2b44ea3b2d"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.73%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a644b26f36c9f43c7976f5726d8459b5c"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a644b26f36c9f43c7976f5726d8459b5c"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a644b26f36c9f43c7976f5726d8459b5c"></a><strong id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0eca439d6c7b46918ec38642b1d660c0"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0eca439d6c7b46918ec38642b1d660c0"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0eca439d6c7b46918ec38642b1d660c0"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_r95533e861d6f43a6a61550c0fae09557"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a9d687882356144d0aedf0791c0fb3b9e"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a9d687882356144d0aedf0791c0fb3b9e"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a9d687882356144d0aedf0791c0fb3b9e"></a>HandleType</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_af3af585891f340b49049298be113dce4"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_af3af585891f340b49049298be113dce4"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_af3af585891f340b49049298be113dce4"></a>Type of handles allocated by SQLAllocHandle. It must be one of the following values:</p>
<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u90955fc4279f4f8481eba83586fa8eb8"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u90955fc4279f4f8481eba83586fa8eb8"></a><ul id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u90955fc4279f4f8481eba83586fa8eb8"><li>SQL_HANDLE_ENV (environment handle)</li><li>SQL_HANDLE_DBC (connection handle)</li><li>SQL_HANDLE_STMT (statement handle)</li></ul>
<p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a>Handles must be allocated in the following order: first the environment handle, then the connection handle, and finally the statement handle. Each subsequently allocated handle depends on the handle allocated before it.</p></td>
</tr>
<tr id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_race60e037af742c4a011f7c18a80d268"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a2dfff073d7ad4d95b80262a49b73153b"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a2dfff073d7ad4d95b80262a49b73153b"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a2dfff073d7ad4d95b80262a49b73153b"></a>InputHandle</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a45d5b00a851546bc9460a2501fba8759"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a45d5b00a851546bc9460a2501fba8759"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a45d5b00a851546bc9460a2501fba8759"></a>Type of the new handle to be allocated.</p>
<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u6d4579039a794155ac40f065ad1002e0"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u6d4579039a794155ac40f065ad1002e0"></a><ul id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_u6d4579039a794155ac40f065ad1002e0"><li>If HandleType is SQL_HANDLE_ENV, the value is SQL_NULL_HANDLE.</li><li>If HandleType is SQL_HANDLE_DBC, the value must be an environment handle.</li><li>If HandleType is SQL_HANDLE_STMT, the value must be a connection handle.</li></ul></td>
</tr>
<tr id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_r0fe9aba8e62e47b6a77a6512e40e19bc"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a870eea2acc264875b6f3ca487610bd48"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a870eea2acc264875b6f3ca487610bd48"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a870eea2acc264875b6f3ca487610bd48"></a>OutputHandlePtr</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a22ab6d451e7c41c791ba2bb971ea8d65"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a22ab6d451e7c41c791ba2bb971ea8d65"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a22ab6d451e7c41c791ba2bb971ea8d65"></a><strong id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0e83142c89c34323840cf5534c9b124f"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0e83142c89c34323840cf5534c9b124f"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0e83142c89c34323840cf5534c9b124f"></a>Output parameter</strong>: A pointer to a buffer in which the returned handle is stored in the newly allocated data structure.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_s8136c512ac4e4481a73d8640d73d26d2"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL\_ERROR: The corresponding handle failed to be allocated.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_sb7bc1f5b242e441583cf7237bb33199b"></a>

See: [Example](example.md)
