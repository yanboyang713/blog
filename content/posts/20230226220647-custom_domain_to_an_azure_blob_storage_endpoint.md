---
title: "custom domain to an Azure Blob Storage endpoint"
draft: false
---

Creating [Azure Data Lake]({{< relref "20230104141434-azure_data_lake.md" >}}) first.


## Map a custom domain with only HTTP enabled {#map-a-custom-domain-with-only-http-enabled}


### Briefly Unavailable {#briefly-unavailable}

✔️ Step 1: Get the host name of your storage endpoint.

✔️ Step 2: Create a canonical name (CNAME) record with your domain provider.

✔️ Step 3: Register the custom domain with Azure.

✔️ Step 4: Test your custom domain.


#### Step 1: Get the host name of your storage endpoint {#step-1-get-the-host-name-of-your-storage-endpoint}

The host name is the storage endpoint URL without the protocol identifier and the trailing slash.

1.  In the Azure portal, go to your storage account.
2.  In the menu pane, under **Settings**, select **Endpoints**.
3.  Copy the value of the **Blob service** endpoint or the **Static website** endpoint to a text file.

**Note:**
The Data Lake storage endpoint is not supported (For example: <https://mystorageaccount.dfs.core.windows.net/>).

1.  Remove the protocol identifier (For example: HTTPS) and the trailing slash from that string. The following table contains examples.

    | Type of endpoint | endpoint                                         | host name                             |
    |------------------|--------------------------------------------------|---------------------------------------|
    | blob service     | <https://boyangpublic.blob.core.windows.net/>    | boyangpublic.blob.core.windows.net    |
    | static website   | <https://boyangpublic.z13.web.core.windows.net/> | boyangpublic.z13.web.core.windows.net |
    |                  |                                                  |                                       |

Set this value aside for later.


#### Step 2: Create a canonical name (CNAME) record with your domain provider {#step-2-create-a-canonical-name--cname--record-with-your-domain-provider}

Create a CNAME record to point to your host name. A CNAME record is a type of Domain Name System (DNS) record that maps a source domain name to a destination domain name.

For example, **cloudflare**: Domain -&gt; DNS -&gt; Records
![](http://res.cloudinary.com/dkvj6mo4c/image/upload/v1677468461/screenshot/izkfpldfdunfz7s2vanq.png)

Create a CNAME record. As part of that record, provide the following items:

The subdomain alias such as www or photos. The subdomain is required, root domains are not supported.

The host name that you obtained in the Get the host name of your storage endpoint section earlier in this article.


#### Step 3: Pre-register your custom domain with Azure {#step-3-pre-register-your-custom-domain-with-azure}

When you pre-register your custom domain with Azure, you permit Azure to recognize your custom domain without having to modify the DNS record for the domain. That way, when you do modify the DNS record for the domain, it will be mapped to the blob endpoint with no downtime.

Run the following PowerShell command.

```bash
az storage account update \
   --resource-group <resource-group-name> \
   --name <storage-account-name> \
   --custom-domain <custom-domain-name> \
   --use-subdomain false
```

For example:

```bash
az storage account update \
> --resource-group permanent \
> --name boyangpublic \
> --custom-domain blog.yanboyang.com \
> --use-subdomain false
```

-   Replace the &lt;resource-group-name&gt; placeholder with the name of the resource group.
-   Replace the &lt;storage-account-name&gt; placeholder with the name of the storage account.
-   Replace the &lt;custom-domain-name&gt; placeholder with the name of your custom domain, including the subdomain.

For example, if your domain is contoso.com and your subdomain alias is www, enter www.contoso.com. If your subdomain is photos, enter photos.contoso.com.

Traffic to your domain is not yet being routed to your storage account until you create a CNAME record with your domain provider. You'll do that in the next section.


#### Step 4: Test your custom domain {#step-4-test-your-custom-domain}

To confirm that your custom domain is mapped to your blob service endpoint, create a blob in a public container within your storage account. Then, in a web browser, access the blob by using a URI in the following format: <http://>&lt;subdomain.customdomain&gt;/&lt;mycontainer&gt;/&lt;myblob&gt;

For example, to access a web form in the myforms container in the photos.contoso.com custom subdomain, you might use the following URI: <http://photos.contoso.com/myforms/applicationform.htm>


## Map a custom domain with HTTPS enabled {#map-a-custom-domain-with-https-enabled}


## Reference List {#reference-list}

1.  <https://learn.microsoft.com/en-us/azure/storage/blobs/storage-custom-domain-name?tabs=azure-portal>
