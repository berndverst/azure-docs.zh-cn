点到站点连接需要上载到 Azure 的证书公用密钥的.cer 文件 （不是私钥）。 以下步骤帮助你导出自签名的根证书的.cer 文件：

1. 若要从证书中获取.cer 文件，打开**管理用户证书**。 通常在证书-Current User\Personal\Certificates，找到自签名的根证书，并右键单击。 单击**所有任务**，然后单击**导出**。 这将打开**证书导出向导**。
2. 在向导中，单击**下一步**。 选择**否，不导出私钥**，然后单击**下一步**。
3. 上**导出文件格式**页上，选择**的 base-64 编码 X.509 (。CER)。**，然后单击**下一步**。 
4. 上**导出的文件**，**浏览**到你想要导出的证书的位置。 有关**文件名**，将证书文件。 然后单击“下一步” 。
5. 单击**完成**导出的证书。 你看到**已成功导出**。 单击**确定**关闭向导。