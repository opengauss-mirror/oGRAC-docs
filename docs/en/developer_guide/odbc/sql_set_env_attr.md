# SQLSetEnvAttr<a name="EN_TOPIC_0242371456"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-07-24T09:43:30.866Z pushedAt=2026-07-24T11:28:31.650Z -->

## Function<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_s93fc7574cf154704b8cff271a57b9601"></a>

Sets environment attributes.

## Prototype<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_sfbbc7fe8e3e34b51af9d3a43dab859fc"></a>

```c
SQLRETURN SQLSetEnvAttr(SQLHENV       EnvironmentHandle,
                        SQLINTEGER    Attribute,    
                        SQLPOINTER    ValuePtr,     
                        SQLINTEGER    StringLength);
```

## Parameters<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_s1c9b27937d964eaba00ae77fe1cd2c71"></a>

**Table 1**  SQLSetEnvAttr parameters

<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_t82b61d38241342ffa2c83b3e50393841"></a>

<table><thead align="left"><tr id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_r3ec068cec36347ccb83a7f18cf131215"><th class="cellrowborder" valign="top" width="23.27%" id="mcps1.2.3.1.1"><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a44a45da69b324aa4b5c1187191ec5c77"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a44a45da69b324aa4b5c1187191ec5c77"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a44a45da69b324aa4b5c1187191ec5c77"></a><strong id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a78fd62134c834d6ab90eace249f90f74"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a78fd62134c834d6ab90eace249f90f74"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a78fd62134c834d6ab90eace249f90f74"></a>Keyword</strong></p></th>
<th class="cellrowborder" valign="top" width="76.73%" id="mcps1.2.3.1.2"><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_aee2bc08a3b8f47bf81fb032ef089ba6d"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_aee2bc08a3b8f47bf81fb032ef089ba6d"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_aee2bc08a3b8f47bf81fb032ef089ba6d"></a><strong id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a51048b44452847fabe05c8633f0220cf"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a51048b44452847fabe05c8633f0220cf"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a51048b44452847fabe05c8633f0220cf"></a>Description</strong></p></th>
</tr>
</thead>
<tbody><tr id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_r89c7807f135840058d4a248137b3ca08"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5b881394ab5445c89e8b7a9cf6d8c93a"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5b881394ab5445c89e8b7a9cf6d8c93a"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5b881394ab5445c89e8b7a9cf6d8c93a"></a>EnvironmentHandle</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ae9e04af441044a6581179c8dac3884f3"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ae9e04af441044a6581179c8dac3884f3"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ae9e04af441044a6581179c8dac3884f3"></a>Environment handle.</p></td>
</tr>
<tr id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_r2088b45aa8374f988b5b381a7e85ae5b"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ace1e0a1bd94f482798c38666d51a57a7"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ace1e0a1bd94f482798c38666d51a57a7"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_ace1e0a1bd94f482798c38666d51a57a7"></a>Attribute</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5713dfef98384960a3106a2b7c9aa751"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5713dfef98384960a3106a2b7c9aa751"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a5713dfef98384960a3106a2b7c9aa751"></a>Environment attribute to be set. Currently, only SQL_ATTR_ODBC_VERSION is supported, which specifies the ODBC version. The default version is SQL_OV_ODBC3.</p></td>
</tr>
<tr id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_re3c53307a0b8488f86edbf902499dcd5"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_afaeaa8d63c164853a2b3aee69d5c4dda"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_afaeaa8d63c164853a2b3aee69d5c4dda"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_afaeaa8d63c164853a2b3aee69d5c4dda"></a>ValuePtr</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_af32d5845d0bc4860ae768bce9257560e"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_af32d5845d0bc4860ae768bce9257560e"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_af32d5845d0bc4860ae768bce9257560e"></a>Points to the value of the corresponding attribute. Depending on the attribute value, it can be a 32-bit integer value or points to a null-terminated string.</p></td>
</tr>
<tr id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_rc78d71a78a944585b1a9275d30efa604"><td class="cellrowborder" valign="top" width="23.27%" headers="mcps1.2.3.1.1 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a3b5e927cb76544a6b2b78ab3c7fcccd7"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a3b5e927cb76544a6b2b78ab3c7fcccd7"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a3b5e927cb76544a6b2b78ab3c7fcccd7"></a>StringLength</p></td>
<td class="cellrowborder" valign="top" width="76.73%" headers="mcps1.2.3.1.2 "><p id="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a99d1997f52594ef2b2b236b459350ccd"><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a99d1997f52594ef2b2b236b459350ccd"></a><a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_a99d1997f52594ef2b2b236b459350ccd"></a>If ValuePtr points to a string or a binary buffer, this parameter is the length of *ValuePtr. If ValuePtr points to an integer value, StringLength is ignored.</p></td>
</tr>
</tbody>
</table>

## Return Values<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_sd43c3bb519574fb68eae3b53fb1b652f"></a>

- SQL\_SUCCESS: The call is successful.

- SQL\_SUCCESS\_WITH\_INFO: Some warning information is present.

- SQL_ERROR: The environment attribute failed to be set.

- SQL\_INVALID\_HANDLE: The input handle is invalid.

## Example<a name="zh-cn_topic_0238272902_zh-cn_topic_0237120432_zh-cn_topic_0059778852_s54d045c03fcc414bab03c37ea6e4da08"></a>

See: [Example](example.md)
