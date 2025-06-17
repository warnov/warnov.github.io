---
title: How to Add an External SSL Certificate to an Endpoint in Azure CDN
date: 2024-07-11 02:13:00 -0500
categories: [Azure, Security and Configuration]
tags: [azure, CDN, SSL, azure key vault, certificates] 
image:
  path: /assets/img/posts/2024-07-11/2024-07-11-Azure-CDN-Addding-SSL-Header.jpg
---
Implementing an SSL certificate for your Azure CDN endpoint is crucial for securing communications and ensuring that content delivered through your CDN, especially static websites, adheres to modern browser security standards demanding HTTPS. This guide details the steps for adding an external SSL certificate to your Azure CDN endpoint, focusing particularly on using Azure Key Vault and an external Certificate Authority (CA) for apex domains.

## Understanding SSL Certificates for Azure CDN

**Apex Domains vs. Subdomains**:
- **Apex Domain**: This is your root domain without any prefix, such as `yourdomain.com`.
- **Subdomains**: These include any variations such as `www.yourdomain.com`, `support.yourdomain.com`, or `blog.yourdomain.com`.

Azure CDN can manage SSL certificates automatically for subdomains, but manual setup is required for apex domains.

## Create a Certificate in Azure Key Vault

Creating a certificate in Azure Key Vault involves selecting the correct type of certificate authority:

1. **Selecting the Right Certificate Type in Azure Key Vault**:
    - Navigate to Azure Key Vault and initiate the process to [create a new certificate](https://learn.microsoft.com/en-us/azure/key-vault/certificates/create-certificate-signing-request?tabs=azure-portal)
    - Choose a certificate issued by a **non-integrated CA**. This selection is crucial because it allows you to use external Certificate Authorities that provide broader acceptance and trust than self-signed and could be more economic than an integrated CA. Also leave the content as `PKCS` as it is the one valid in the CDN.

## Generate and Retrieve the CSR

Creating the CSR in Azure Key Vault and retrieving it for submission to a CA are key steps in obtaining your certificate:

1. **Generate and Download CSR**:
    - In Azure Key Vault, select the certificate you're creating.
    - Use the "Certificate Operations" to find and select "Download CSR". The CSR (Certificate Signing Request) is just a block of plaintext that includes encoded information about your company and the domain name for which the certificate is requested.


## Submit the CSR to an External CA

Once you have the CSR, the next step is to have it signed by your chosen external CA.

1. **Submit the CSR**:
    - Go to the website of your chosen CA (e.g., Namecheap) and initiate the process to obtain an SSL certificate.
    - You will need to paste the CSR into the appropriate form on the CA’s website.
    - Specify the exact domain that the certificate should cover (`yourdomain.com`). This specification is crucial because the SSL certificate will be explicitly linked to the domain names you specify here.

2. **Complete Domain Verification**:
    - The CA will typically require you to prove that you control the domains listed in your CSR. This can be done via DNS configuration, email verification, or file uploads to your domain.

## Import the Signed Certificate into Azure Key Vault

After your CA issues the signed certificate:

1. **Merge the Signed Certificate**:
    - Return to Azure Key Vault and go to the same certificate.
    - Use the "Merge Signed Request" option to upload and integrate the signed certificate file from the CA with the original CSR.

## Configure Azure CDN to Use the SSL Certificate

Setting up your Azure CDN to use the new SSL certificate involves a few more steps:

1. **Assign Managed Identity and Permissions**:
    - [Assign a system-managed identity to your Azure CDN](https://learn.microsoft.com/en-us/azure/cdn/cdn-custom-ssl?tabs=option-2-enable-https-with-your-own-certificate#select-the-certificate-for-azure-cdn-to-deploy) that allows it to securely access Azure Key Vault. Even when the documentation is referencing a procedure based on Key Vaults working with `Access Policies` you could give permissions for your CDN to the Key Vault using the modern and more recommended `Access Control IAM` approach. In this case, instead of choosing permissions, you have to choose the role `Certificate User`
    - Assign the Certificate permissions: Get and List (for Access Policies) **or**, "Certificate User" role (for IAM) to this identity within Key Vault to enable the CDN to read the certificate.

2. **Enable Custom Domain HTTPS**:
    - In the Azure CDN endpoint settings, select "Custom Domain HTTPS," choose your Key Vault, and select the certificate.

## Testing and Monitoring

- **Verify HTTPS Functionality**: Access your domain via HTTPS and check for any browser security warnings.
- **Monitor Certificate Expiry**: Keep track of when your certificate will expire and plan for its renewal.

## Conclusion

Adding an external SSL certificate to an Azure CDN endpoint ensures that your content is delivered securely, enhancing user trust and meeting security standards. By carefully selecting the certificate type in Azure Key Vault, generating and submitting a CSR, and configuring Azure CDN, you establish a secure foundation for your CDN content delivery.
