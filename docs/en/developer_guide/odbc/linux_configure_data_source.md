# Configuring a Data Source on Linux<a name="ZH-CN_TOPIC_0289900737"></a>

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-11T09:23:16.052Z pushedAt=2026-08-11T09:23:55.993Z -->

The ODBC driver (`libogodbc.so`) provided by oGRAC can be used once it is configured into a data source. Configuring a data source requires setting up two configuration files, `odbc.ini` and `odbcinst.ini` (generated when compiling and installing unixODBC and placed in the `/usr/local/etc` directory by default), and performing configuration on the server.

## Procedure<a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_s0b78a5ac02d949ada15c29b00b5c734d"></a>

1. Obtain the unixODBC source package.

    The unixODBC version must be 2.3.6 or later. Taking unixODBC-2.3.6 as an example, you can download it directly: [unixODBC-2.3.6.tar.gz](http://www.unixodbc.org/unixODBC-2.3.6.tar.gz).

2. Install unixODBC. If an earlier version of unixODBC is already installed on the machine, uninstall it first or directly overwrite the existing installation.

    Taking unixODBC-2.3.6 as an example, run the following commands on the client to install unixODBC. By default, it is installed to the `/usr/local` directory, the data source files are generated in the `/usr/local/etc` directory, and the library files are generated in the `/usr/local/lib` directory.

    ```shell
    tar zxvf unixODBC-2.3.6.tar.gz
    cd unixODBC-2.3.6
    ./configure --enable-gui=no
    make
    # The installation may require root privileges.
    make install
    ```

3. Replace the client ODBC driver.

    Copy the ODBC driver (libogodbc.so) provided by oGRAC to the `/usr/local/lib` directory, and verify its integrity.

    ```shell
    ldd /usr/local/lib/libogodbc.so
    ```

4. Configure the data source.

    1. Configure the ODBC driver file.

        Append the following content to the `/usr/local/etc/odbcinst.ini` file.

        ```shell
        [OgracMPP]
        Driver64=/usr/local/lib/libogodbc.so
        setup=/usr/local/lib/libogodbc.so
        ```

        [Table 1](#zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_td564f21e7c8e458bbd741b09896f5d91) describes the configuration parameters in the `odbcinst.ini` file.

        **Table 1** Configuration parameters in the `odbcinst.ini` file

        <a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_td564f21e7c8e458bbd741b09896f5d91"></a>

        <table><thead align="left"><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_rb271d7f472844a738954ee76c1e32380"><th class="cellrowborder" valign="top" width="26.340000000000003%" id="mcps1.2.4.1.1"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a28c78abf21ca48ddaee6fd3c34f3397f"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a28c78abf21ca48ddaee6fd3c34f3397f"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a28c78abf21ca48ddaee6fd3c34f3397f"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8250ebf049654830b8253567e694de2e"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8250ebf049654830b8253567e694de2e"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8250ebf049654830b8253567e694de2e"></a>Parameter</strong></p>
        </th>
        <th class="cellrowborder" valign="top" width="38.89%" id="mcps1.2.4.1.2"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a87fb9ed3f1b140119593e71843494a31"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a87fb9ed3f1b140119593e71843494a31"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a87fb9ed3f1b140119593e71843494a31"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ace84b030832948cdafb08ca46b7053d4"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ace84b030832948cdafb08ca46b7053d4"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ace84b030832948cdafb08ca46b7053d4"></a>Description</strong></p>
        </th>
        <th class="cellrowborder" valign="top" width="34.77%" id="mcps1.2.4.1.3"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad3bb948669de4429be3b566882116d70"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad3bb948669de4429be3b566882116d70"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad3bb948669de4429be3b566882116d70"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b696083205917"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b696083205917"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b696083205917"></a>Example</strong></p>
        </th>
        </tr>
        </thead>
        <tbody><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r6bdd431c96eb4f7c85664375766de400"><td class="cellrowborder" valign="top" width="26.340000000000003%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8e9446f0aa7646fe9538d94cde8e74a3"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8e9446f0aa7646fe9538d94cde8e74a3"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a8e9446f0aa7646fe9538d94cde8e74a3"></a>[DriverName]</p>
        </td>
        <td class="cellrowborder" valign="top" width="38.89%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_accc06fd075c040e0884c74e19c4239b2"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_accc06fd075c040e0884c74e19c4239b2"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_accc06fd075c040e0884c74e19c4239b2"></a>Driver name, which corresponds to the driver name in the data source DSN.</p>
        </td>
        <td class="cellrowborder" valign="top" width="34.77%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad04d23410c114609aec4f993de148ce3"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad04d23410c114609aec4f993de148ce3"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ad04d23410c114609aec4f993de148ce3"></a>[OgracMPP]</p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r1fbbae158f634be28568c80376a63292"><td class="cellrowborder" valign="top" width="26.340000000000003%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a67dcf15c971843c0a09bcb75af898ed8"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a67dcf15c971843c0a09bcb75af898ed8"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a67dcf15c971843c0a09bcb75af898ed8"></a>Driver64</p>
        </td>
        <td class="cellrowborder" valign="top" width="38.89%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ac1271d08ff5242849aefc93aed5b0038"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ac1271d08ff5242849aefc93aed5b0038"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ac1271d08ff5242849aefc93aed5b0038"></a>Path to the driver dynamic library.</p>
        </td>
        <td class="cellrowborder" valign="top" width="34.77%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a9a7de0f221414c0f97d14fbbf74eb104"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a9a7de0f221414c0f97d14fbbf74eb104"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a9a7de0f221414c0f97d14fbbf74eb104"></a>Driver64=/usr/local/lib/libogodbc.so</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r4e8ad74343a849f8a7224716cf20ab57"><td class="cellrowborder" valign="top" width="26.340000000000003%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a18a35a0225c14a2e9aecdc72084f1418"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a18a35a0225c14a2e9aecdc72084f1418"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a18a35a0225c14a2e9aecdc72084f1418"></a>setup</p></td>
        <td class="cellrowborder" valign="top" width="38.89%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6eecb8f4cf064448a14a46be8e243daa"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6eecb8f4cf064448a14a46be8e243daa"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6eecb8f4cf064448a14a46be8e243daa"></a>Driver installation path, which must be the dynamic library path specified in Driver64.</p></td>
        <td class="cellrowborder" valign="top" width="34.77%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7f838d6444df4566853f51533341401b"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7f838d6444df4566853f51533341401b"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7f838d6444df4566853f51533341401b"></a>setup=/usr/local/lib/libogodbc.so</p></td>
        </tr>
        </tbody>
        </table>

    2. Configure the data source file.

        Append the following content to the `/usr/local/etc/odbc.ini` file.

        ```shell
        [OgracDB]
        Driver=OgracMPP
        Servername=127.0.0.1
        Port=1611
        Username=test
        Password=test123
        ```

        [Table 2](#zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_t55845a6555f2454297b64ce47ad3d648) describes the `odbc.ini` file configuration parameters.

        <b>Table 2</b>  `odbc.ini` file configuration parameters

        <a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_t55845a6555f2454297b64ce47ad3d648"></a>

        <table><thead align="left"><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r670b13dddd7d4cf1803d4fb238347e4f"><th class="cellrowborder" valign="top" width="30.570000000000004%" id="mcps1.2.4.1.1"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acaf03818449f45a3baf67b847f258487"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acaf03818449f45a3baf67b847f258487"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acaf03818449f45a3baf67b847f258487"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_aa54a5590250c4c4ea04282a93d622454"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_aa54a5590250c4c4ea04282a93d622454"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_aa54a5590250c4c4ea04282a93d622454"></a>Parameter</strong></p>
        </th>
        <th class="cellrowborder" valign="top" width="33.37%" id="mcps1.2.4.1.2"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a2996382cc39847f79d6e271605212118"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a2996382cc39847f79d6e271605212118"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a2996382cc39847f79d6e271605212118"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b522969821213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b522969821213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_b522969821213"></a>Description</strong></p>
        </th>
        <th class="cellrowborder" valign="top" width="36.059999999999995%" id="mcps1.2.4.1.3"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5962444038ea45f4a049a6659667dbf3"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5962444038ea45f4a049a6659667dbf3"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5962444038ea45f4a049a6659667dbf3"></a><strong id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_abdb053af064f4fd39fba24fd58949b21"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_abdb053af064f4fd39fba24fd58949b21"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_abdb053af064f4fd39fba24fd58949b21"></a>Example</strong></p>
        </th>
        </tr>
        </thead>
        <tbody><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r0fb190e64b064cd5bdc4ba6a85b7f5b5"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5a22b1b502ce470daf0e150a5a22478d"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5a22b1b502ce470daf0e150a5a22478d"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5a22b1b502ce470daf0e150a5a22478d"></a>[DSN]</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4d0e13e5a4974e95ba02b65f0880769a"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4d0e13e5a4974e95ba02b65f0880769a"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4d0e13e5a4974e95ba02b65f0880769a"></a>Data source name.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p574463221213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p574463221213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p574463221213"></a>[OgracDB]</p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r6594d9c7f185417cbb0362b69ff20436"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a11f2651489984caaa3ba6c5a69b3bb4a"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a11f2651489984caaa3ba6c5a69b3bb4a"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a11f2651489984caaa3ba6c5a69b3bb4a"></a>Driver</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a45cdb983d7eb4357bfe4363048fff41b"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a45cdb983d7eb4357bfe4363048fff41b"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a45cdb983d7eb4357bfe4363048fff41b"></a>Driver name, which corresponds to DriverName in odbcinst.ini.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p587420421213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p587420421213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p587420421213"></a>Driver=OgracMPP</p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ra2915c8c5cb64c31bca0b7cc97b3cb94"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acda586c1c3074516b55eca79e45c96b9"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acda586c1c3074516b55eca79e45c96b9"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_acda586c1c3074516b55eca79e45c96b9"></a>Servername</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af9844951ac5f42cf95b1e70a86da25ed"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af9844951ac5f42cf95b1e70a86da25ed"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af9844951ac5f42cf95b1e70a86da25ed"></a>Server IP address.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p705535321213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p705535321213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p705535321213"></a>Servername=127.0.0.1</p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r478d8852e4294c01bb48cbaaf087df82"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p237028621213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p237028621213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p237028621213"></a>Username</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6d4497d8c6134705a3971992b73eb9a8"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6d4497d8c6134705a3971992b73eb9a8"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a6d4497d8c6134705a3971992b73eb9a8"></a>Database user name.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4c244a359e2341a9a25a634898bef6f8"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4c244a359e2341a9a25a634898bef6f8"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a4c244a359e2341a9a25a634898bef6f8"></a>Username=<span id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text542984584919"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text542984584919"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text542984584919"></a>test</span></p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r373830e4fd9c4f8997fcdde5596099f4"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a72415586711142e39d29a2d8b664ff2b"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a72415586711142e39d29a2d8b664ff2b"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a72415586711142e39d29a2d8b664ff2b"></a>Password</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab0a86a7c047f420081b1c5d1c57ca78f"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab0a86a7c047f420081b1c5d1c57ca78f"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab0a86a7c047f420081b1c5d1c57ca78f"></a>Database user password.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7803bcaba1324e56913a1a44d46f692e"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7803bcaba1324e56913a1a44d46f692e"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a7803bcaba1324e56913a1a44d46f692e"></a>Password=test123</p>
        <div class="note" id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_note66001755162217"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_note66001755162217"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_note66001755162217"></a><span class="notetitle"> Note: </span><div class="notebody"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p57144884162217"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p57144884162217"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p57144884162217"></a>The ODBC driver itself has already cleared the password from memory to ensure that the user password is not retained in memory after the connection is established.</p>
        </div></div>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_r99417f5e349c4f7b89331e2a2d56200d"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af23b15858fa341b6b849ff850e10a6cb"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af23b15858fa341b6b849ff850e10a6cb"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_af23b15858fa341b6b849ff850e10a6cb"></a>Port</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a86b34375cfcd4cdd84b070cc42d847c3"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a86b34375cfcd4cdd84b070cc42d847c3"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a86b34375cfcd4cdd84b070cc42d847c3"></a>Server port number.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab47ede062bc6450097f576df5aa63442"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab47ede062bc6450097f576df5aa63442"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_ab47ede062bc6450097f576df5aa63442"></a>Port=<span id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text64689595438"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text64689595438"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_text64689595438"></a>1611</span></p>
        </td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_rd27aa89b945d4ac6b33000b6df5c916e"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5f7c150a5b5f471fa60adba4fd913dd4"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5f7c150a5b5f471fa60adba4fd913dd4"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a5f7c150a5b5f471fa60adba4fd913dd4"></a>sslmode</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p462272821213"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p462272821213"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_zh-cn_topic_0058965230_p462272821213"></a>Enables SSL mode.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a53d9561f3715460d9f7b8a022292214c"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a53d9561f3715460d9f7b8a022292214c"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_a53d9561f3715460d9f7b8a022292214c"></a>sslmode=VERIFY_CA</p>
        </td>
        </tr>
        <tr id="row115681541336"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="p65681140317"><a name="p65681140317"></a><a name="p65681140317"></a>sslca</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="p195681543312"><a name="p195681543312"></a><a name="p195681543312"></a>Root certificate file that issues certificates for ODBC. The root certificate is used to verify the validity of server certificates.</p></td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="p205681419316"><a name="p205681419316"></a><a name="p205681419316"></a>sslca=/usr/test/certificate/ca.crt</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_row3180734193610"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p91811034123617"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p91811034123617"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p91811034123617"></a>sslcert</p></td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p31811134143616"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p31811134143616"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p31811134143616"></a>ODBC-side certificate file, which contains the client's public key.</p></td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p318163493616"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p318163493616"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p318163493616"></a>sslcert=/usr/test/certificate/client.crt</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_row18741310143913"><td class="cellrowborder" valign="top" width="30.570000000000004%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p674191083916"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p674191083916"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p674191083916"></a>sslkey</p>
        </td>
        <td class="cellrowborder" valign="top" width="33.37%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p67427101397"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p67427101397"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p67427101397"></a>ODBC-side private key file, which is used for digital signing and for decrypting data encrypted with the public key.</p>
        </td>
        <td class="cellrowborder" valign="top" width="36.059999999999995%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p7742101083917"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p7742101083917"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_p7742101083917"></a>sslkey=/usr/test/certificate/client.key</p>
        </td>
        </tr>

        </tbody>
        </table>

        For details about the allowed values of the `sslmode` options, see the following table:

        **Table 3**  `sslmode` options

        <a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_table22136585143846"></a>

        <table><thead align="left"><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row7964795143846"><th class="cellrowborder" valign="top" width="14.87%" id="mcps1.2.4.1.1"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p44475740143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p44475740143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p44475740143856"></a>sslmode</p>
        </th>
        <th class="cellrowborder" valign="top" width="18.87%" id="mcps1.2.4.1.2"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p45765196143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p45765196143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p45765196143856"></a>SSL Encryption Enabled</p></th>
        <th class="cellrowborder" valign="top" width="66.25999999999999%" id="mcps1.2.4.1.3"><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p15993376143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p15993376143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p15993376143856"></a>Description</p></th>
        </tr>
        </thead>
        <tbody><tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row4599577143846"><td class="cellrowborder" valign="top" width="14.87%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p41388182143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p41388182143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p41388182143856"></a>VERIFY_CA</p></td>
        <td class="cellrowborder" valign="top" width="18.87%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p64108423143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p64108423143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p64108423143856"></a>Yes</p></td>
        <td class="cellrowborder" valign="top" width="66.25999999999999%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p25399806143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p25399806143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p25399806143856"></a>An SSL secure connection must be used, and it must be verified that the database has a certificate issued by a trusted certificate authority.</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row10527338143846"><td class="cellrowborder" valign="top" width="14.87%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p61520998143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p61520998143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p61520998143856"></a>DISABLED</p></td>
        <td class="cellrowborder" valign="top" width="18.87%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p17144918143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p17144918143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p17144918143856"></a>No</p></td>
        <td class="cellrowborder" valign="top" width="66.25999999999999%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p46561127143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p46561127143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p46561127143856"></a>No SSL secure connection is used.</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row19876694143846"><td class="cellrowborder" valign="top" width="14.87%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p53085640143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p53085640143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p53085640143856"></a>PREFERRED</p></td>
        <td class="cellrowborder" valign="top" width="18.87%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p4969569143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p4969569143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p4969569143856"></a>Possible</p></td>
        <td class="cellrowborder" valign="top" width="66.25999999999999%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p66990807143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p66990807143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p66990807143856"></a>If supported by the database, an SSL secure encrypted connection is recommended, but the authenticity of the database server is not verified.</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row43737844143846"><td class="cellrowborder" valign="top" width="14.87%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p48154267143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p48154267143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p48154267143856"></a>REQUIRED</p></td>
        <td class="cellrowborder" valign="top" width="18.87%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p8181559143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p8181559143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p8181559143856"></a>Yes</p></td>
        <td class="cellrowborder" valign="top" width="66.25999999999999%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p58726539143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p58726539143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p58726539143856"></a>An SSL secure connection must be used, but only data encryption is performed, without verifying the authenticity of the database server.</p></td>
        </tr>
        <tr id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_row17092427143846"><td class="cellrowborder" valign="top" width="14.87%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p21152121143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p21152121143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p21152121143856"></a>VERIFY_FULL</p></td>
        <td class="cellrowborder" valign="top" width="18.87%" headers="mcps1.2.4.1.2 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p35600232143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p35600232143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p35600232143856"></a>Yes</p></td>
        <td class="cellrowborder" valign="top" width="66.25999999999999%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p65046568143856"><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p65046568143856"></a><a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_p65046568143856"></a>An SSL secure connection must be used. In addition to the verification scope of verify-ca, it also verifies whether the host name of the database host matches the certificate content.<span id="text18227139123314"><a name="text18227139123314"></a><a name="text18227139123314"></a>oGRAC</span> does not support this mode.</p></td>
        </tr>
        </tbody>
        </table>

5. Use SSL mode.

    1. Generate the server certificate.

       Perform the following operations on the server where the database is installed:

       ```shell
       # Generate the CA certificate ca.crt and private key ca.key on the host.
       openssl req -newkey rsa:3072 -passout pass:12345678 -keyout ca.key -x509 -days 365 -out ca.crt -subj "/C=CN/ST=BJ/O=huawei/OU=huawei/CN=CA/emailAddress=123456@xxx.com"
       # Generate the private key mes.key and certificate request mes.csr on the host.
       openssl req -newkey rsa:3072 -nodes -keyout mes.key -out mes.csr -subj "/C=CN/ST=BJ/L=BJ/O=huawei/OU=huawei/CN=Server/emailAddress=123456@xxx.com"
       # Generate the certificate mes.crt on the host.
       openssl x509 -req -days 365 -in mes.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mes.crt
       ```

    2. Configure server parameters.

       Create a directory `/opt/ograc/data` on the server where the database is installed, place the generated certificates `ca.crt`, `ca.key`, `ca.srl`, `mes.crt`, `mes.csr`, and `mes.key` into this directory, and modify the permissions on the folder and certificates:

       ```shell
       # Modify the folder permissions, where ogracdba is the database installation user.
       chmod 700 /opt/ograc/data
       chown ogracdba:ogracdba /opt/ograc/data
       
       # Modify the certificate permissions.
       chmod 400 /opt/ograc/data/*
       chown ogracdba:ogracdba /opt/ograc/data/*
       ```

    3. Add SSL parameters to enable SSL mode for the database.

       ```shell
       # Log in to the database.
       ogsql test/test123@127.0.0.1:1611 -q
       # Execute the following SQL statements to add parameters.
       alter system set SSL_CA = '/opt/ograc/data/ca.crt';
       alter system set SSL_CERT = '/opt/ograc/data/mes.crt';
       alter system set SSL_KEY='/opt/ograc/data/mes.key';
       
       # Restart the database.
       cms res -stop db
       cms res -start db
       ```

       After the restart, execute the SQL command: `show parameter ssl`. If the query result shows that the value of `HAVE_SSL` is `TRUE`, SSL has been successfully enabled.

    4. Configure an SSL connection on the ODBC client.

       Create an arbitrary directory `/usr/test/certificate` on the client where ODBC is located, and use scp to transfer the `ca.crt` file generated on the server to the client where ODBC is located. Then perform the following operations:

       ```shell
       mkdir -p /usr/test/certificate
       cd /usr/test/certificate
       scp root@xxx.xxx.xx.xx:/opt/ograc/data/ca.crt .
       ```

       Generate the private key `client.key` and the certificate request `client.csr` on the client:

       ```shell
       openssl req -newkey rsa:3072 -nodes -keyout client.key -out client.csr -subj "/C=CN/ST=BJ/L=BJ/O=huawei/OU=huawei/CN=Server/emailAddress=123456@xxx.com"
       ```

       Use scp to transfer `client.csr` to the `/opt/ograc/data` directory on the server where the database is located:

       ```shell
       scp client.csr root@xxx.xxx.xx.xx:/opt/ograc/data/
       ```

    5. Generate the client certificate.

       Generate the certificate `client.crt` in the `/opt/ograc/data` directory on the server where the database is located, and transfer it to the client:

       ```shell
       cd /opt/ograc/data
       openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt
       ```

    6. Transfer the certificate to the client.

       Transfer the generated certificate `client.crt` to the client:

       ```shell
       cd /usr/test/certificate
       scp root@xxx.xxx.xx.xx:/opt/ograc/data/client.crt .
       ```

    7. Configure ODBC parameters on the client.

       Grant permissions to the certificate:

       ```shell
       chmod 400 /usr/test/certificate/*
       ```

       Add the following parameters in the `odbc.ini` file:

       ```shell
       sslca=/usr/test/certificate/ca.crt
       sslcert=/usr/test/certificate/client.crt
       sslkey=/usr/test/certificate/client.key
       sslmode = VERIFY_CA
       ```

6. Configure environment variables on the client.

    ```shell
    vim ~/.bashrc
    ```

    Append the following content to the configuration file:

    ```shell
    export PATH=/usr/local/bin:$PATH
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
    ```

    Run the following command to apply the settings:

    ```shell
    source ~/.bashrc
    ```

7. Register the driver.

    ```shell
    odbcinst -i -d -f /usr/local/etc/odbcinst.ini
    ```

## Testing the Data Source Configuration<a name="zh-cn_topic_0283136654_zh-cn_topic_0237120407_zh-cn_topic_0059778464_scfaeeaa70a9448889e6fdd7e37d172c5"></a>

After installation, run the `isql` command to verify the connection, that is, `isql OgracDB -v test test123`.

- If the following information is displayed, the configuration is correct and the connection is successful.

    ```shell
    +---------------------------------------+
    | Connected!                            |
    |                                       |
    | sql-statement                         |
    | help [tablename]                      |
    | quit                                  |
    |                                       |
    +---------------------------------------+
    SQL> 
    ```

- If `ERROR` information is displayed, a configuration error exists. Check whether the above configuration is correct.
