# SQLSetConnectAttr<a name="ZH-CN_TOPIC_0242371455"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-24T06:20:58.635Z pushedAt=2026-07-24T11:27:52.680Z -->

## Function<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_se8dc3eed91144d248bfd9c9d9b144b15"></a>

Sets various connection attributes.

## Prototype<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_s2c4e3a37799b4f07a9b22c05f4166f27"></a>

```c
SQLRETURN SQLSetConnectAttr(SQLHDBC       ConnectionHandle,
                            SQLINTEGER    Attribute,    
                            SQLPOINTER    ValuePtr,     
                            SQLINTEGER    StringLength);
```

## Parameters<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_s7729ce590d3c4eef9cbc62b3973d4feb"></a>

**Table 1** SQLSetConnectAttr parameters

<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_tade40e51081242c39ed6b9052dd71671"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_r7eb093ebfd84455ca7313d34669fee99"><th class="cellrowborder" valign="top" width="23.27%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a667e7535363842138b12773dfea3acd1"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a667e7535363842138b12773dfea3acd1"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a667e7535363842138b12773dfea3acd1"></a><strong id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a98aaed01fefa44009722a371ea43cd72"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a98aaed01fefa44009722a371ea43cd72"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a98aaed01fefa44009722a371ea43cd72"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.73%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af54cf2cf0e134631b3bfc82100d783c3"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af54cf2cf0e134631b3bfc82100d783c3"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af54cf2cf0e134631b3bfc82100d783c3"></a><strong id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a20d1c40a467346c7b0f9da6d16d8e72e"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a20d1c40a467346c7b0f9da6d16d8e72e"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a20d1c40a467346c7b0f9da6d16d8e72e"></a>Parameter Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_rcbcb561bbde04c1abab3653b06f8af6f"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_ae791a6b139fd429687444e6b313a8376"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_ae791a6b139fd429687444e6b313a8376"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_ae791a6b139fd429687444e6b313a8376"></a>ConnectionHandle</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a46c8fea3a3e248f7b927fdcd54246daa"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a46c8fea3a3e248f7b927fdcd54246daa"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a46c8fea3a3e248f7b927fdcd54246daa"></a>Connection handle.</p></td>
</tr>
<tr id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_ref5fb95e1c1e4c70bdca3be2865ea6f6"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a35e34c8507cf49f1b8cf924d11a89ade"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a35e34c8507cf49f1b8cf924d11a89ade"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a35e34c8507cf49f1b8cf924d11a89ade"></a>Attribute</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a9ae74af514a5475eaf5ed65ed75043f0"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a9ae74af514a5475eaf5ed65ed75043f0"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a9ae74af514a5475eaf5ed65ed75043f0"></a>Connection attribute to be set. It can be one of the following values:</p>
<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"></a><ul id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_uefc416b07b1941be81ff69fc135a3a7f"><li>SQL_ATTR_AUTOCOMMIT: Enables or disables auto-commit. 1: Enabled; 2: Disabled.</li><li>SQL_ATTR_QUERY_TIMEOUT: Specifies the maximum wait time for SQL execution, in seconds.</li><li>SQL_ATTR_LOGIN_TIMEOUT: Specifies the maximum wait time for logging in to the database.</li><li>SQL_ATTR_CONNECTION_TIMEOUT: Specifies the database connection timeout.</li></ul>
<p id="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a><a name="zh-cn_topic_0238272884_zh-cn_topic_0237120414_zh-cn_topic_0059778058_a0214a0e9219d4856874070000e4365ba"></a>If the specified attribute is not within the above range, the driver returns SQL_ERROR.</p></td>
</tr>
<tr id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_r10e9ba39ddc74bbe97828a9fe966fbda"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_afec09da615ce49e9b3e7c0d84e1c55ac"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_afec09da615ce49e9b3e7c0d84e1c55ac"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_afec09da615ce49e9b3e7c0d84e1c55ac"></a>ValuePtr</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af02e14e5cd344e1da182edd3ec08a228"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af02e14e5cd344e1da182edd3ec08a228"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af02e14e5cd344e1da182edd3ec08a228"></a>Points to the value of the corresponding attribute. Depending on the attribute value, it can be a 32‑bit unsigned integer value or points to a null‑terminated string.</p></td>
</tr>
<tr id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_r3de5904d75fd4cfa9accc0752197579a"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af423e7625e0341eaaba023ac05edec20"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af423e7625e0341eaaba023ac05edec20"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_af423e7625e0341eaaba023ac05edec20"></a>StringLength</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a0582ff9af622483fa0dcebb56c6187cf"><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a0582ff9af622483fa0dcebb56c6187cf"></a><a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_a0582ff9af622483fa0dcebb56c6187cf"></a>If ValuePtr points to a string or a binary buffer, this parameter is the length of *ValuePtr. If ValuePtr points to an integer value, StringLength is ignored.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_s12ee56593ac441249a9f5f184e227202"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: The connection attribute failed to be set.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272901_zh-cn_topic_0237120431_zh-cn_topic_0059777827_s25786d32524c458786d43445baa48941"></a>

See: [Example](example.md)
