---
title: "如何从 Twilio (Java) 发起电话呼叫 | Microsoft Docs"
description: "了解如何在 Azure 的 Java 应用程序中使用 Twilio 从网页发起电话呼叫。"
services: 
documentationcenter: java
author: devinrader
manager: twilio
editor: mollybos
ms.assetid: 0381789e-e775-41a0-a784-294275192b1d
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 11/25/2014
ms.author: microsofthelp@twilio.com
ms.openlocfilehash: 04ecb80a2a9e15b549b47138caf71c7e64bda500
ms.sourcegitcommit: f537befafb079256fba0529ee554c034d73f36b0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/11/2017
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-java-application-on-azure"></a>如何在 Azure 的 Java 应用程序中使用 Twilio 发起电话呼叫
以下示例演示了如何使用 Twilio 从 Azure 中托管的网页发起呼叫。 生成的应用程序会提示用户输入电话呼叫值，如以下屏幕截图中所示。

![使用 Twilio 和 Java 的 Azure 呼叫窗体][twilio_java]

需要执行以下操作来使用本主题中的代码：

1. 获取 Twilio 帐户和身份验证令牌。 若要开始使用 Twilio，请在 [http://www.twilio.com/pricing][twilio_pricing] 上评估定价。 你可以在注册[https://www.twilio.com/try-twilio][try_twilio]。 有关 Twilio 提供的 API 的信息，请参阅[http://www.twilio.com/api][twilio_api]。
2. 获取 Twilio JAR。 在 [https://github.com/twilio/twilio-java][twilio_java_github] 上，可以下载 GitHub 源并创建自己的 JAR，或者下载预建的 JAR（带有或不带依赖项）。
   本主题中的代码使用预建的 TwilioJava-3.3.8-with-dependencies JAR 编写。
3. 将 JAR 添加到 Java 生成路径。
4. 如果使用 Eclipse 创建此 Java 应用程序，请使用 Eclipse 的部署程序集功能将 Twilio JAR 包含在应用程序部署文件 (WAR) 中。 如果不使用 Eclipse 创建此 Java 应用程序，请确保将 Twilio JAR 包含在与 Java 应用程序相同的 Azure 角色中，并将其添加到应用程序的类路径下。
5. 确保 cacerts 密钥库包含带有 MD5 指纹 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4（序列号为 35:DE:F4:CF，SHA1 指纹为 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A）的 Equifax 安全证书颁发机构证书。 这是 [https://api.twilio.com][twilio_api_service] 服务的证书颁发机构 (CA) 证书，在使用 Twilio API 时调用。 有关将此 CA 证书添加到 JDK 的 cacert 存储的信息，请参阅[将证书添加到 Java CA 证书存储][add_ca_cert]。

此外，熟悉的信息[使用创建 Hello World 应用程序使用 Azure Toolkit for Eclipse][azure_java_eclipse_hello_world]，或如果你未使用 Eclipse，则托管在 Azure 中的 Java 应用程序的其他方法，强烈建议。

## <a name="create-a-web-form-for-making-a-call"></a>创建用于发起呼叫的 Web 窗体
以下代码演示了如何创建 Web 窗体来检索用于发起呼叫的用户数据。 在本示例中，将创建名为 **TwilioCloud** 的新的动态 Web 项目，并添加 **callform.jsp** 作为 JSP 文件。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Automated call form</title>
    </head>
    <body>
     <p>Fill in all fields and click <b>Make this call</b>.</p>
     <br/>
      <form action="makecall.jsp" method="post">
       <table>
         <tr>
           <td>To:</td>
           <td><input type="text" size=50 name="callTo" value="" />
           </td>
         </tr>
         <tr>
           <td>From:</td>
           <td><input type="text" size=50 name="callFrom" value="" />
           </td>
         </tr>
         <tr>
           <td>Call message:</td>
           <td><input type="text" size=400 name="callText" value="Hello. This is the call text. Good bye." />
           </td>
         </tr>
         <tr>
           <td colspan=2><input type="submit" value="Make this call" />
           </td>
         </tr>
       </table>
     </form>
     <br/>
    </body>
    </html>

## <a name="create-the-code-to-make-the-call"></a>创建用于发起呼叫的代码
以下代码在用户填写 callform.jsp 显示的窗体后调用，用于创建呼叫消息并生成呼叫。 在本示例中，JSP 文件被命名为 **makecall.jsp** 并添加到 **TwilioCloud** 项目。 （使用 Twilio 帐户和身份验证令牌，而不是分配给以下代码中的 **accountSID** 和 **authToken** 的占位符值。）

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    import="java.util.*"
    import="com.twilio.*"
    import="com.twilio.sdk.*"
    import="com.twilio.sdk.resource.factory.*"
    import="com.twilio.sdk.resource.instance.*"
    pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Call processing happens here</title>
    </head>
    <body>
        <b>This is my make call page.</b><p/>
     <%
    try 
    {
         // Use your account SID and authentication token instead
         // of the placeholders shown here.
         String accountSID = "your_twilio_account";
         String authToken = "your_twilio_authentication_token";

         // Instantiate an instance of the Twilio client.     
         TwilioRestClient client;
         client = new TwilioRestClient(accountSID, authToken);

         // Retrieve the account, used later to retrieve the CallFactory.
         Account account = client.getAccount();

         // Display the client endpoint. 
         out.println("<p>Using Twilio endpoint " + client.getEndpoint() + ".</p>");

         // Display the API version.
         String APIVERSION = TwilioRestClient.DEFAULT_VERSION;
         out.println("<p>Twilio client API version is " + APIVERSION + ".</p>");

         // Retrieve the values entered by the user.
         String callTo = request.getParameter("callTo");  
         // The Outgoing Caller ID, used for the From parameter,
         // must have previously been verified with Twilio.
         String callFrom = request.getParameter("callFrom");
         String userText = request.getParameter("callText");

         // Replace spaces in the user's text with '%20', 
         // to make the text suitable for a URL.
         userText = userText.replace(" ", "%20");

         // Create a URL using the Twilio message and the user-entered text.
         String Url="http://twimlets.com/message";
         Url = Url + "?Message%5B0%5D=" + userText;

         // Display the message URL.
         out.println("<p>");
         out.println("The URL is " + Url);
         out.println("</p>");

         // Place the call From, To and URL values into a hash map. 
         HashMap<String, String> params = new HashMap<String, String>();
         params.put("From", callFrom);
         params.put("To", callTo);
         params.put("Url", Url);

         CallFactory callFactory = account.getCallFactory();
         Call call = callFactory.create(params);
         out.println("<p>Call status: " + call.getStatus()  + "</p>"); 
    } 
    catch (TwilioRestException e) 
    {
        out.println("<p>TwilioRestException encountered: " + e.getMessage() + "</p>");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    catch (Exception e) 
    {
        out.println("<p>Exception encountered: " + e.getMessage() + "");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    %>
    </body>
    </html>

除了发起呼叫外，makecall.jsp 还可显示 Twilio 终结点、API 版本和呼叫状态。 如以下屏幕截图所示：

![使用 Twilio 和 Java 的 Azure 呼叫响应][twilio_java_response]

## <a name="run-the-application"></a>运行应用程序
以下是运行你的应用程序; 的高级步骤处找不到这些步骤的详细信息[使用创建 Hello World 应用程序使用 Azure Toolkit for Eclipse][azure_java_eclipse_hello_world]。

1. 将 TwilioCloud WAR 导出到 Azure **approot** 文件夹。 
2. 修改 **startup.cmd** 以解压缩 TwilioCloud WAR。
3. 针对计算模拟器编译应用程序。
4. 在计算模拟器中启动部署。
5. 打开浏览器，并运行 **http://localhost:8080/TwilioCloud/callform.jsp**。
6. 在窗体中输入值，单击“发起此呼叫”，并查看 makecall.jsp 中的结果。

准备好部署到 Azure 之后，请针对云部署重新进行编译，部署到 Azure，然后在浏览器中运行 http://*your_hosted_name*.cloudapp.net/TwilioCloud/callform.jsp（将 *your_hosted_name* 替换为自己的值）。

## <a name="next-steps"></a>后续步骤
提供此代码是为了向您演示在 Azure 上使用 java 的 Twilio 的基本功能。 在生产中部署到 Azure 之前，可能希望添加更多错误处理或其他功能。 例如：

* 可以使用 Azure 存储 Blob 或 SQL 数据库存储电话号码和呼叫文本，而不使用 Web 窗体。 有关使用 java 的 Azure 存储 blob 的信息，请参阅[如何使用 Blob 存储服务中通过 Java][howto_blob_storage_java]。 有关使用在 Java 中的 SQL 数据库的信息，请参阅[在 Java 中使用 SQL 数据库][howto_sql_azure_java]。
* 可以使用 **RoleEnvironment.getConfigurationSettings** 从部署的配置设置中检索 Twilio 帐户 ID 和身份验证令牌，而不是在 makecall.jsp 中对这些值进行硬编码。 璝惠**RoleEnvironment**类，请参阅[使用 Azure 服务运行时库在 JSP 中][ azure_runtime_jsp]和 Azure 服务运行时程序包文档[http://dl.windowsazure.com/javadoc][azure_javadoc]。
* Makecall.jsp 代码将分配 Twilio 提供的 URL， [http://twimlets.com/message][twimlet_message_url]到**Url**变量。 此 URL 提供了一个 Twilio 标记语言 (TwiML) 响应，指示 Twilio 如何继续进行呼叫。 例如，返回的 TwiML 可能包含 **&lt;Say&gt;** 谓词，该谓词生成了与呼叫接收人的谈话的文本。 而不是使用 Twilio 提供的 URL，你可以构建你自己服务来响应 Twilio 的请求;有关详细信息，请参阅[如何使用 Twilio 实现语音和 SMS 功能 Java][howto_twilio_voice_sms_java]。 有关 TwiML 的详细信息可以在找到[http://www.twilio.com/docs/api/twiml][twiml]，以及有关详细信息**&lt;说出&gt;**和处找不到其他 Twilio 谓词[http://www.twilio.com/docs/api/twiml/say][twilio_say]。
* 阅读 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全准则。

有关 Twilio 的更多信息，请参阅 [https://www.twilio.com/docs][twilio_docs]。

## <a name="see-also"></a>另请参阅
* [如何使用 Twilio 实现语音和 SMS 功能 Java][howto_twilio_voice_sms_java]
* [将证书添加到 Java CA 证书存储][add_ca_cert]

[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/api
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_java_github]: http://github.com/twilio/twilio-java
[twimlet_message_url]: http://twimlets.com/message
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api_service]: http://api.twilio.com
[add_ca_cert]: java-add-certificate-ca-store.md
[azure_java_eclipse_hello_world]: http://msdn.microsoft.com/library/windowsazure/hh690944.aspx
[howto_twilio_voice_sms_java]: partner-twilio-java-how-to-use-voice-sms.md
[howto_blob_storage_java]: http://www.windowsazure.com/develop/java/how-to-guides/blob-storage/
[howto_sql_azure_java]: http://msdn.microsoft.com/library/windowsazure/hh749029.aspx
[azure_runtime_jsp]: http://msdn.microsoft.com/library/windowsazure/hh690948.aspx
[azure_javadoc]: http://dl.windowsazure.com/javadoc
[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say
[twilio_java]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaCallForm.jpg
[twilio_java_response]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaMakeCall.jpg
