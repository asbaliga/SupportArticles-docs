---
title: Troubleshoot Web Application Proxy
description: Provides troubleshooting information for Web Application Proxy including event explanations and solutions.
ms.date: 10/16/2023
author: Deland-Han
ms.author: delhan
manager: dcscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika, kgremban, jgerend, v-lianna
ms.custom: sap:tcp/ip-communications, csstroubleshoot
ms.technology: networking
ms.assetid: a2fef55d-747b-4e20-8f21-5f8807e7ef87
---
# Troubleshoot Web Application Proxy

This article is relevant for the on-premises version of Web Application Proxy. To enable secure access to on-premises applications over the cloud, see the [Microsoft Entra Application Proxy content](/azure/active-directory/manage-apps/application-proxy).

_Applies to:_ &nbsp; Windows Server 2022, Windows Server 2019, Windows Server 2016

This section provides troubleshooting procedures for Web Application Proxy including event explanations and solutions. There are three places where errors are displayed:

- In the Web Application Proxy administrator console

    Each event ID listed in the administrator console can be viewed in the Windows Event Viewer and corresponding descriptions and solutions are found below.

    Open Event Viewer and look for events related to Web Application Proxy under **Applications and Services Logs** > **Microsoft** > **Windows** > **Web Application Proxy** > **Admin**.

    :::image type="content" source="./media/troubleshooting-web-application-proxy/web-application-proxy.png" alt-text="Screenshot of the Event Viewer shows events related to Web Application Proxy.":::

    If needed, detailed logs are available by turning on analytics and debugging logs and turning on the Web Application Proxy session log, found in the Windows Event Viewer under \\**Microsoft**\\**Windows**\\**Web Application Proxy**\\**Admin**.

- In PowerShell errors

    Events for issues encountered during configuration are displayed in PowerShell.

    All errors are presented to the PowerShell user using standard PowerShell error prompts. All PowerShell cmdlets are logged as events. All events that occur in PowerShell are listed in the Windows Event Viewer with the ID number 12016, and are defined below in the PowerShell section.

- In the Best Practices Analyzer

    These events are described in the [Best Practices Analyzer for Web Application Proxy](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn383651(v=ws.11)).

## PowerShell messages

|Event or symptom|Possible cause|Resolution|
|--------------------|------------------|--------------|
|The trust certificate ("ADFS ProxyTrust - \<WAP machine name>") isn't valid|This could be caused by any of the following:<p>- The Application Proxy machine was down for too long.<br />- Disconnections between the Web Application Proxy and AD FS<br />- Certificate infrastructure issues<br />- Changes on the AD FS machine, or the renew process between the Web Application Proxy and the AD FS didn't run as planned every 8 hours, then they need to renew trust<br />- The clock of the Web Application Proxy machine and the AD FS aren't synchronized.|Make sure the clocks are synchronized. Run the `Install-WebApplicationProxy` cmdlet.|
|Configuration data wasn't found in AD FS|This may be because Web Application Proxy wasn't fully installed yet or because of changes in the AD FS database or corruption of the database.|Run the `Install-WebApplicationProxy` cmdlet|
|An error occurred when Web Application Proxy tried to read configuration from AD FS.|This may indicate that AD FS isn't reachable, or that AD FS encountered an internal problem trying to read configuration from the AD FS database.|Verify that AD FS is reachable and working properly.|
|The configuration data stored in AD FS is corrupted or Web Application Proxy was unable to parse it.<p>OR<p>Web Application Proxy was unable to retrieve the list of Relying Parties from AD FS.|This may occur if the configuration data was modified in AD FS.|Restart the Web Application Proxy service. If the problem persists, run the `Install-WebApplicationProxy` cmdlet.|

## Administrator console events

The following administrator console events are indicative of authentication errors, invalid tokens or expired cookies.

|Event or symptom|Possible cause|Resolution|
|--------------------|------------------|--------------|
|11005<p>Web Application Proxy couldn't create the cookie encryption key using the secret from the configuration.|The global configuration `AccessCookiesEncryptionKey` parameter was changed by the PowerShell cmdlet: `Set-WebApplicationProxyConfiguration -RegenerateAccessCookiesEncryptionKey`|No action is required. The problematic cookie was removed and the user was redirected to STS for authentication.|
|12000<p>Web Application Proxy couldn't check for configuration changes for at least 60 minutes|Web Application Proxy can't access the Web Application Proxy configuration using the cmdlet `Get-WebApplicationProxyConfiguration/Application`. This is caused by lack of connectivity with AD FS or the need to renew trust with AD FS.|Check connectivity with AD FS. You can do this using the link `https://<FQDN_AD_FS_Proxy>/FederationMetadata/2007-06/FederationMetadata.xml`. Make sure there's trust established between the AD FS and the Web Application Proxy. If these solutions don't work, run the `Install-WebApplicationProxy` cmdlet.|
|12003<p>Web Application Proxy couldn't parse the access cookie.|This may indicate that the Web Application Proxy and the AD FS aren't connected or that they don't receive the same configuration.|Check connectivity with AD FS. You can do this using the link `https://<FQDN_AD_FS_Proxy>/FederationMetadata/2007-06/FederationMetadata.xml`. Make sure there's trust established between the AD FS and the Web Application Proxy. If these solutions don't work, run the `Install-WebApplicationProxy` cmdlet.|
|12004<p>Web Application Proxy received a request with a nonvalid access cookie.|This event may indicate that the Web Application Proxy and the AD FS aren't connected or that they don't receive the same configuration.<p>If you ran the `AccessCookiesEncryptionKey` parameter was changed by `Set-WebApplicationProxyConfiguration -RegenerateAccessCookiesEncryptionKey` PowerShell cmdlet, this event is normal and requires no resolution steps.|Check connectivity with AD FS. You can do this using the link `https://<FQDN_AD_FS_Proxy>/FederationMetadata/2007-06/FederationMetadata.xml`. Make sure there's trust established between the AD FS and the Web Application Proxy. If these solutions don't work, run the `Install-WebApplicationProxy` cmdlet.|
|12008<p>Web Application Proxy exceeded the maximum number of permitted Kerberos authentication attempts to the backend server.|This event may indicate incorrect configuration between Web Application Proxy and the backend application server, or a problem in time and date configuration on both machines.|The backend server declined the Kerberos ticket created by Web Application Proxy. Verify that the configuration of the Web Application Proxy and the backend application server are configured correctly.<p>Make sure that the time and date configuration on the Web Application Proxy and the backend application server are synchronized.|
|12011<p>Web Application Proxy received a request with a non-valid access cookie signature.|This event may indicate that the Web Application Proxy and the AD FS aren't connected or that they don't receive the same configuration. If you ran the `AccessCookiesEncryptionKey` parameter was changed by `Set-WebApplicationProxyConfiguration -RegenerateAccessCookiesEncryptionKey` PowerShell cmdlet, this event is normal and requires no resolution steps.|Check connectivity with AD FS. You can do this using the link `https://<FQDN_AD_FS_Proxy>/FederationMetadata/2007-06/FederationMetadata.xml`. Make sure there's trust established between the AD FS and the Web Application Proxy. If these solutions don't work, run the `Install-WebApplicationProxy` cmdlet.|
|12027<p>Proxy encountered an unexpected error while processing the request. The name provided isn't a properly formed account name.|This event may indicate incorrect configuration between Web Application Proxy and the domain controller server, or a problem in time and date configuration on both machines.|The domain controller declined the Kerberos ticket created by Web Application Proxy. Verify that the configuration of the Web Application Proxy and the backend application server are configured correctly, especially the SPN configuration. Make sure the Web Application Proxy is domain joined to the same domain as the domain controller to ensure that the domain controller establishes trust with Web Application Proxy. Make sure that the time and date configuration on the Web Application Proxy and the domain controller are synchronized.|
|13012<p>Web Application Proxy received a nonvalid edge token signature||Make sure you updated Web Application Proxy with [Web Application Proxy cannot detect the updated certificate after it automatically updates on Windows Server 2012 R2](https://go.microsoft.com/fwlink/?LinkId=400701).|
|13013<p>Web Application Proxy received a request that contained an expired edge token.|Web Application Proxy and AD FS don't have synchronized clocks.|Synchronize the clocks between Web Application Proxy and AD FS.|
|13014<p>Web Application Proxy received a request with a nonvalid edge token. The token isn't valid because it couldn't be parsed.|This may indicate an issue with the AD FS configuration.|Check your AD FS configuration and, if necessary, restore the default configuration.|
|13015<p>Web Application Proxy received a request with an expired access cookie.|This could indicate clocks that aren't synchronized.|If you're working with a cluster of Web Application Proxy machines, make sure that the time and date of the machines is synchronized.|
|13016<p>Web Application Proxy can't retrieve a Kerberos ticket on behalf of the user because there's no UPN in the edge token or in the access cookie.|There's a problem with the STS configuration.|Fix the UPN claim configuration in the STS.|
|13019<p>Web Application Proxy can't retrieve a Kerberos ticket on behalf of the user because of the following general API error|This event may indicate incorrect configuration between Web Application Proxy and the domain controller server, or a problem in time and date configuration on both machines.|The domain controller declined the Kerberos ticket created by Web Application Proxy. Verify that the configuration of the Web Application Proxy and the backend application server are configured correctly, especially the SPN configuration. Make sure the Web Application Proxy is domain joined to the same domain as the domain controller to ensure that the domain controller establishes trust with Web Application Proxy. Make sure that the time and date configuration on the Web Application Proxy and the domain controller are synchronized.|
|13020<p>Web Application Proxy can't retrieve a Kerberos ticket on behalf of the user because the backend server SPN isn't defined.|This event may indicate incorrect configuration between Web Application Proxy and the domain controller server, or a problem in time and date configuration on both machines.|The domain controller declined the Kerberos ticket created by Web Application Proxy. Verify that the configuration of the Web Application Proxy and the backend application server are configured correctly, especially the SPN configuration. Make sure the Web Application Proxy is domain joined to the same domain as the domain controller to ensure that the domain controller establishes trust with Web Application Proxy. Make sure that the time and date configuration on the Web Application Proxy and the domain controller are synchronized.|
|13022<p>Web Application Proxy can't authenticate the user because the backend server responds to Kerberos authentication attempts with an HTTP 401 error.|This event may indicate incorrect configuration between Web Application Proxy and the backend application server, or a problem in time and date configuration on both machines.|The backend server declined the Kerberos ticket created by Web Application Proxy. Verify that the configuration of the Web Application Proxy and the backend application server are configured correctly. Make sure that the time and date configuration on the Web Application Proxy and the backend application server are synchronized.|
|13025<p>The client didn't present an SSL certificate to Web Application Proxy.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13026<p>The client presented an SSL certificate to Web Application Proxy, but the certificate isn't valid: the certificate doesn't match the thumbprint.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13028<p>Web Application Proxy received a request that contained an edge token that isn't yet valid.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized.|
|13030<p>The client presented an SSL certificate to Web Application Proxy, but the trust provider doesn't trust the certificate authority that issued the client certificate.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13031<p>The client presented an SSL certificate to Web Application Proxy, but the certificate chain terminated in a root certificate that isn't trusted by the trust provider.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13032<p>The client presented an SSL certificate to Web Application Proxy, but the certificate wasn't valid for the requested usage.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13033<p>The client presented an SSL certificate to Web Application Proxy, but the certificate wasn't within its validity period when verifying against the current system clock or the timestamp in the signed file.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|
|13034<p>The client presented an SSL certificate to Web Application Proxy, but the certificate wasn't valid.|This event may indicate a problem in time and date configuration.|Make sure that the certificate infrastructure is valid and that the time and date of the Web Application Proxy and the AD FS are synchronized. Make sure that the thumbprint configured for the Web Application Proxy is the correct one.|

The following administrator console events are indicative of problems having to do with configuration such as provisioning, requests that aren't successful, backend servers that are unreachable and buffer overflows.

|Event or symptom|Possible cause|Resolution|
|--------------------|------------------|--------------|
|12019<p>Web Application Proxy couldn't create a listener for the following URL.|A possible cause for the event is that another service is listening to the same URL.|The admin must make sure that no one listens or binds to the same URLs. To check this, run the command: `netsh http show urlacl`. If this URL is used by another component running on the Web Application Proxy machine, either remove it, or use a different URL to publish the applications through Web Application Proxy.|
|12020<p>Web Application Proxy couldn't create a reservation for the following URL.|A possible cause for the event is that another service has a reservation on the same URL.|The admin must make sure that no one binds to the same URLs. To check this, run the command: `netsh http show urlacl`. If this URL is used by another component running on the Web Application Proxy machine, either remove it, or use a different URL to publish the applications through Web Application Proxy.|
|12021<p>Web Application Proxy couldn't bind the SSL server certificate. All other configuration settings were applied.|Unable to create and set a configuration record of SSL certificate data.|Make sure that the certificate thumbprints that are configured for Web Application Proxy applications are installed on all the Web Application Proxy machines with a private key in the local computer store.|
|13001<p>The SSL server certificate presented to Web Application Proxy by the backend server isn't valid; the certificate isn't trusted.|One or more errors were found in the Secure Sockets Layer (SSL) certificate sent by the server. This could indicate that the backend server provided an SSL that wasn't valid or that there's no trust between the Web Application Proxy and the backend server.|Validate a backend server SSL certificate. Make sure that the Web Application Proxy machine is configured with the right root CAs to trust the backend server certificate.|
|13006|When the error code is 0x80072ee7, the failure is caused by the inability to resolve the backend server URL. Other error codes are described in [WinHttpSendRequest function (winhttp.h)](https://msdn.microsoft.com/library/windows/desktop/aa384110(v=vs.85))|Check that the backend server URL is correct and that its name can be resolved correctly from the Web Application Proxy machine.|
|13007<p>The HTTP response from the backend server wasn't received within the expected interval.|The back-end server request timed out or is slow or unresponsive.|Check the backend server configuration. If it's slow, check the connectivity to the backend server, and also consider changing the Web Application Proxy global configuration parameter cmdlet for `InactiveTransactionsTimeoutSec`.|

## References

- [What's New in Web Application Proxy in Windows Server 2016](/windows-server/remote/remote-access/web-application-proxy/web-app-proxy-windows-server)
- [Working with Web Application Proxy](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn584113(v=ws.11))
- [Web Application Proxy Walkthrough Guide](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn280944(v=ws.11))