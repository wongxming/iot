# CoAP-based connections {#concept_gn3_kr5_wdb .concept}

## Overview {#section_sf1_mr5_wdb .section}

Constrained Application Protocol \(CoAP\) is applicable to low-power devices that have limited resources, such as Narrowband IoT \(NB-IoT\) devices. The process for connecting NB-IoT devices to IoT Platform based on CoAP is described in [Figure 1](#fig_irr_4r5_wdb). ￼

![](images/3114_en-US.png "CoAP-based connections")

The CoAP-based connection follows this process:

1.  The NB-IoT module integrates an SDK for accessing Alibaba Cloud IoT Platform. The manufacturer requests unique certificates in the console, including ProductKey, DeviceName, and DeviceSecret, and installs them to devices.
2.  The NB-IoT device accesses IoT Platform over the Internet service provider's \(ISP's\) cellular network. You need to contact the local ISP to make sure that the NB-IoT network has covered the region where the device is located.
3.  After the device is connected to IoT Platform, the ISP's machine-to-machine \(M2M\) platform manages service usage and billing for traffic generated by the NB-IoT device.
4.  You can report real-time data collected by the device to IoT Platform based on Constrained Application Protocol/User Datagram Protocol \(CoAP/UDP\). IoT Platform secures connections with more than 100 million NB-IoT devices and manages related data. The system connects with big data services, ApsaraDB, and report systems of Alibaba Cloud to achieve intelligent management.
5.  IoT Platform provides data sharing functions and message pushing services to forward data to related service instances and quickly integrate device assets and actual applications.

## Use the SDK to connect an NB-IoT device to IoT Platform {#section_uxd_vr5_wdb .section}

Follow these instructions to connect the device to IoT Platform using the C SDK. For more information, see [Download device SDKs](https://partners-intl.aliyun.com/help/doc-detail/42648.htm).

## CoAP-based connection {#section_owj_wr5_wdb .section}

The process of connecting devices using the CoAP protocol is described as follows:

1.  Use the CoAP endpoint address: `endpoint = ${ProductKey}.iot-as-coap.cn-shanghai.aliyuncs.com:5684`. Replace ProductKey with the product key that you have requested.
2.  Download the [root certificate](http://aliyun-iot.oss-cn-hangzhou.aliyuncs.com/cert_pub/root.crt?spm=5176.doc30539.2.1.1MRvV5&file=root.crt) using the Datagram Transport Layer Security \(DTLS\)-based secure session.
3.  Trigger device authentication to obtain the token for the device before the device sends data.
4.  The device includes this token in reported data. If the token has expired, you need to request a new token. The system caches the token locally for 48 hours.

## CoAP-based connection {#section_vlc_pdk_b2b .section}

1.  Authenticate the device. You can use this function to request the token before the device sends data. You only need to request a token once.

    ```
    
    POST /auth
    Host: ${productKey}.iot-as-coap.cn-shanghai.aliyuncs.com
    Port: 5684
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: {"productKey":"ZG1EvTEa7NN","deviceName":"NlwaSPXsCpTQuh8FxBGH","clientId":"mylight1000002","sign":"bccb3d2618afe74b3eab12b94042f87b"}
    ```

    Parameters are described as follows:

    -   Method: POST, Ony the POST method is supported.
    -   URL: /auth, URL address.
    -   Accept: the encoding format for receiving data. Currently, application/json and application/cbor are supported.
    -   Content-Format: encoding format of upstream data. Currently, application/json and application/cbor are supported.
    The payload parameters and JSON data formats are described as follows:

    |Field Name|Required|Description|
    |----------|--------|-----------|
    |productKey|Yes|productKey, requested from the IoT Platform console.|
    |deviceName|Required|deviceName, requested from the IoT Platform console.|
    |sign|Required|Signature, format: hmacmd5\(deviceSecret,content\). The content includes all parameter values that are submitted to the instance, except for version, sign, resources, and signmethod. These parameter values are sorted in alphabetical order and without concatenated symbols.|
    |signmethod|No|Algorithm type, hmacmd5 or hmacsha1.|
    |clientId|Required|Client identifier, a maximum of 64 characters.|
    |timestamp|No|Timestamp, not used in checks.|

    The response is as follows:

    ```
     response: {"token":"eyJ0b2tlbiI6IjBkNGUyNjkyZTNjZDQxOGU5MTA4Njg4ZDdhNWI3MjUxIiwiZXhwIjoxNDk4OTg1MTk1fQ.DeQLSwVX8iBjdazjzNHG5zcRECWcL49UoQfq1lXrJvI"}
    ```

    The return codes are described as follows:

    |Code|Description|Payload|Remarks|
    |----|-----------|-------|-------|
    |2.05|Content|Authentication passed: token object|The request is correct.|
    |4.00|Bad Request|Returned error message|The payload in the request is invalid.|
    |4.04|Not Found|404 not found|The requested path does not exist.|
    |4.05|Method Not Allowed|Supported method|The request method is not allowed.|
    |4.06|Not Acceptable|The required Accept parameter|The Accept parameter is not the specified type.|
    |4.15|Unsupported Content-Format|Supported content|The requested content is not the specified type.|
    |5.00|Internal Server Error|Error message|The authentication request is timed out or an error occurs on the authentication server.|

    The SDK provides IOT\_CoAP\_Init and IOT\_CoAP\_DeviceNameAuth for building CoAP-based authentication on IoT Platform

    Example code:

    ```
    
    iotx_coap_context_t *p_ctx = NULL;
    p_ctx = IOT_CoAP_Init(&config);
    if (NULL ! = p_ctx) {
      IOT_CoAP_DeviceNameAuth(p_ctx);
      do {
        count ++;
        if (count == 11) {
          count = 1;
        }
        IOT_CoAP_Yield(p_ctx);
      } while (m_coap_client_running);
      IOT_CoAP_Deinit(&p_ctx);
    } else {
      HAL_Printf("IoTx CoAP init failed\r\n");
    }
    ```

    The function statement is as follows:

    ```
    
    /**
    * @brief Initialize the CoAP client.
    * This function is used to initialize the data structure and network,
    * and create the DTLS session.
    *
    * @param [in] p_config: Specify the CoAP client parameter.
    *
    * @retval NULL: The initialization failed.
    * @retval NOT_NULL: The contex of CoAP client.
    * @see None.
    */
    iotx_coap_context_t *IOT_CoAP_Init(iotx_coap_config_t *p_config);
    
    /**
    * @brief Device name handle for authentication by the remote server.
    *
    * @param [in] p_context: Contex pointer to specify the CoAP client.
    *
    * @retval IOTX_SUCCESS: The authentication is passed.
    * @retval IOTX_ERR_SEND_MSG_FAILED: Sending the authentication message failed.
    * @retval IOTX_ERR_AUTH_FAILED: The authentication failed or timed out.
    * @see iotx_ret_code_t.
    */
    int IOT_CoAP_DeviceNameAuth(iotx_coap_context_t *p_context);
    ```

2.  Set upstream data \($\{endpoint\}/topic/$\{topic\}\).

    This is used to send data to a specified topic. To set $\{topic\}, choose**Products** \> **Notifications**in the IoT Platform console.

    To send data to topic`/productkey/${deviceName}/pub`, use URL address `${productKey}.iot-as-coap.cn-shanghai.aliyuncs.com:5684/topic/productkey/device/pub`to report data if the current device name is device. You can only use a topic that has the publishing permission to report data.

    Example code:

    ```
    
    POST /topic/${topic}
    Host: ${productKey}.iot-as-coap.cn-shanghai.aliyuncs.com
    Port: 5683
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: ${your_data}
    CustomOptions: number:61(token)
    ```

    Parameter description:

    -   Method: POST. The POST method is supported.
    -   URL: /topic/$\{topic\}. Replace $\{topic\} with the topic of the current device.
    -   Accept: Received data encoding methods. Currently, application/json and application/cbor are supported.
    -   Content-Format: Upstream data encoding format. The service does not check this format.
    -   CustomOptions: Indicates the token that the device has obtained after authentication. Option Number: 61.
3.  The SDK provides IOT\_CoAP\_SendMessage for sending data, and IOT\_CoAP\_GetMessagePayload and IOT\_CoAP\_GetMessageCode for receiving data.

    Example code:

    ```
    
    /* send data */
    static void iotx_post_data_to_server(void *param)
    {
      char path[IOTX_URI_MAX_LEN + 1] = {0};
      iotx_message_t message;
      iotx_deviceinfo_t devinfo;
      message.p_payload = (unsigned char *)"{\"name\":\"hello world\"}";
      message.payload_len = strlen("{\"name\":\"hello world\"}");
      message.resp_callback = iotx_response_handler;
      message.msg_type = IOTX_MESSAGE_CON;
      message.content_type = IOTX_CONTENT_TYPE_JSON;
      iotx_coap_context_t *p_ctx = (iotx_coap_context_t *)param;
      iotx_set_devinfo(&devinfo);
      snprintf(path, IOTX_URI_MAX_LEN, "/topic/%s/%s/update/", (char *)devinfo.product_key,
      (char *)devinfo.device_name);
      IOT_CoAP_SendMessage(p_ctx, path, &message);
    }
    
    /* receive data */
    static void iotx_response_handler(void *arg, void *p_response)
    {
      int len = 0;
      unsigned char *p_payload = NULL;
      iotx_coap_resp_code_t resp_code;
      IOT_CoAP_GetMessageCode(p_response, &resp_code);
      IOT_CoAP_GetMessagePayload(p_response, &p_payload, &len);
      Hal_printf ("[appl]: Message response code: 0x % x \ r \ n ", resp_code );
      Hal_printf ("[appl]: Len: % d, payload: % s, \ r \ n ", Len, Fargo payload );
    }
    ```

    ```
    
    /** 
    * @ Brief send a message using the specific path to the server.
    * The client must pass the authentication by the server before sending messages.
    *
    * @param [in] p_context: Contex pointer to specify the CoAP client.
    * @param [in] p_path: Specify the path name.
    * @param [in] p_message: The message to be sent.
    *
    * @retval IOTX_SUCCESS: The message has been sent.
    * @retval IOTX_ERR_MSG_TOO_LOOG: The message is too long.
    * @retval IOTX_ERR_NOT_AUTHED: The client has not passed the authentication by the server.
    * @see iotx_ret_code_t.
    */
    int IOT_CoAP_SendMessage(iotx_coap_context_t *p_context, char *p_path, iotx_message_t *p_message);
    
    /**
    * @brief Retrieves the length and payload pointer of the specified message.
    *
    * @param [in] p_message: The pointer to the message to get the payload. This should not be NULL.
    * @param [out] pp_payload: The pointer to the payload.
    * @param [out] p_len: The size of the payload.
    *
    * @retval IOTX_SUCCESS: The payload has been obtained.
    * @retval IOTX_ERR_INVALID_PARAM: The payload cannot be obtained due to invalid parameters.
    * @see iotx_ret_code_t.
    **/
    int IOT_CoAP_GetMessagePayload(void *p_message, unsigned char **pp_payload, int *p_len);
    
    /**
    * @brief Get the response code from a CoAP message.
    *
    * @param [in] p_message: The pointer to the message to add the address information to.
    * This should not be null.
    * @param [out] p_resp_code: The response code.
    *
    * @retval IOTX_SUCCESS: The response code to the message has been obtained.
    * @retval IOTX_ERR_INVALID_PARAM: The pointer to the message is NULL.
    * @see iotx_ret_code_t.
    **/
    int IOT_CoAP_GetMessageCode(void *p_message, iotx_coap_resp_code_t *p_resp_code);
    ```


## Restrictions {#section_ut1_xqj_zdb .section}

-   The topics follow the Message Queuing Telemetry Transport \(MQTT\) topic standard. The `coap://host:port/topic/${topic}` operation in CoAP can be used for all $\{topic\} topics and MQTT-based topics. You cannot specify parameters in the format of ``? query_String=xxx``.
-   The client locally caches the requested token that has been transmitted over DTLS and included in the response.
-   The transmitted data size depends on the maximum transmission unit \(MTU\).The MTU less than 1KB is recommended.

## Descriptions of other functions in the C SDK {#section_crf_drj_zdb .section}

-   Use IOT\_CoAP\_Yield to receive data.

    Call this function to receive data. You can run this function in a single thread if the system allows.

    ```
    
    /**
    * @brief Handle CoAP response packet from remote server,
    * and process timeout requests etc..
    *
    * @param [in] p_context: Contex pointer to specify the CoAP client.
    *
    * @return status.
    * @see iotx_ret_code_t.
    */
    int IOT_CoAP_Yield(iotx_coap_context_t *p_context);
    ```

-   Use IOT\_CoAP\_Deinit to free up the memory.

    ```
    
    /**
    * @brief Deinitialize the CoAP client.
    * This function is used to release the CoAP DTLS session
    * and release the related resource. *
    * @param [in] p_context: Contex pointer to specify the CoAP client.
    *
    * @return None.
    * @see None.
    */
    void IOT_CoAP_Deinit(iotx_coap_context_t **p_context);
    ```


For more information about how to use the C SDK, see \\sample\\coap\\coap-example.c.

