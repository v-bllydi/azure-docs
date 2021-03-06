---
title: Protect your content with Azure Media Services | Microsoft Docs
description: This articles give an overview of content protection with Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''

ms.assetid: 81bc00e1-dcda-4d69-b9ab-8768b793422b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/29/2017
ms.author: juliako

---
# Protecting content overview
Microsoft Azure Media Services enables you to secure your media from the time it leaves your computer through storage, processing, and delivery. Media Services allows you to deliver your live and on-demand content encrypted dynamically with Advanced Encryption Standard (AES-128) or any of the three major DRM systems: Microsoft PlayReady, Google Widevine, and Apple FairPlay. Media Services also provides a service for delivering AES keys and DRM (PlayReady, Widevine, and FairPlay) licenses to authorized clients. 

The following image illustrates the Azure Media Services content protection workflow. 

![Protect with PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

This article explains concepts and terminology relevant to understanding content protection with AMS. The article also provides links to articles that discuss how to protect content. 

## Dynamic encryption
Azure Media Services enables you to deliver your content encrypted dynamically with AES clear key or DRM encryption: Microsoft PlayReady, Google Widevine, and Apple FairPlay. Currently, you can encrypt the following streaming formats: HLS, MPEG DASH, and Smooth Streaming. Encryption on progressive downloads is not supported. Each encryption method supports the following streaming protocols:
- AES: MPEG-DASH, Smooth Streaming, and HLS
- PlayReady: MPEG-DASH, Smooth Streaming, and HLS
- Widevine: MPEG-DASH
- FairPlay: HLS

To encrypt an asset, you need to associate an encryption content key with your asset and also configure an authorization policy for the key. Content keys can be specified or automatically generated by Media Services.

You also need to configure the asset's delivery policy. If you want to stream a storage encrypted asset, make sure to specify how you want to deliver it by configuring the asset delivery policy.

When a stream is requested by a player, Media Services uses the specified key to dynamically encrypt your content using AES clear key or DRM encryption. To decrypt the stream, the player requests the key from AMS key delivery service. To decide whether or not the user is authorized to get the key, the service evaluates the authorization policies that you specified for the key.

## AES-128 Clear Key vs DRM
Customers often wonder whether they should use AES encryption or a DRM system. The primary difference between using AES encryption and the DRM systems is that with AES encryption the content key is transmitted to the client in an unencrypted format ("in the clear"). As a result, the key used to encrypt the content can be viewed in a network trace on the client in plaintext. AES-128 clear key is suitable for use cases where the viewer is a trusted party (eg: encrypting corporate videos distributed within a company to be viewed by employees).

PlayReady, Widevine, and FairPlay all provide a higher level of encryption compared to AES-128 clear key encryption. The content key is transmitted in an encrypted format. Additionally, decryption is handle in a secure environment at the operating system level where is significantly more difficult for a malicious user to attack. DRM is recommended for use cases where the viewer may not be a trusted party and you require the highest level of security.

## Storage encryption
You can use storage encryption to encrypt your clear content locally using AES-256 bit encryption and then upload it to Azure Storage where it is stored encrypted at rest. Assets protected with storage encryption are automatically unencrypted and placed in an encrypted file system prior to encoding, and optionally re-encrypted prior to uploading back as a new output asset. The primary use case for storage encryption is when you want to secure your high-quality input media files with strong encryption at rest on disk.

In order to deliver a storage encrypted asset, you must configure the asset’s delivery policy so Media Services knows how you want to deliver your content. Before your asset can be streamed, the streaming server decrypts and streams your content using the specified delivery policy (for example, AES, common encryption, or no encryption).

## Types of encryption
Playready and Widevine utilize Common Encryption (AES CTR mode). FairPlay utilizes AES CBC mode encryption. AES-128 clear key encryption utilizes Envelope Encryption.

## Licenses and keys delivery service
Media Services provides a key delivery service for delivering DRM (PlayReady, Widevine, FairPlay) licenses and AES keys to authorized clients. You can use [the Azure portal](media-services-portal-protect-content.md), REST API, or Media Services SDK for .NET to configure authorization and authentication policies for your licenses and keys.

## Control content access
You can control who has access to your content by configuring the content key authorization policy. The content key authorization policy supports either open or token restriction.

### Open authorization
With an open authorization policy, the content key is sent to any client (no restriction).

### Token authorization
With a token-restricted authorization policy, the content key will only be send to a client that presents a valid JSON Web Token (JWT) or Simple Web Token (SWT) in the key/license request. This token must be issued by a Secure Token Service (STS). You can use Microsoft Active Directory as an STS or deploy a custom STS. The STS must be configured to create a token signed with the specified key and issue claims that you specified in the token restriction configuration. The Media Services key delivery service will return the requested key/license to the client if the token is valid and the claims in the token match those configured for the key/license.

When configuring the token restricted policy, you must specify the primary verification key, issuer and audience parameters. The primary verification key contains the key that the token was signed with, issuer is the secure token service that issues the token. The audience, sometimes called scope, describes the intent of the token or the resource the token authorizes access to. The Media Services key delivery service validates that these values in the token match the values in the template.

## Streaming URLs
If your asset was encrypted with more than one DRM, you should use an encryption tag in the streaming URL: (format='m3u8-aapl', encryption='xxx').

The following considerations apply:
* No more than one encryption type can be specified.
* Encryption type does not have to be specified in the URL if only one encryption was applied to the asset.
* Encryption type is case insensitive.
* The following encryption types can be specified:  
  * **cenc**:  for PlayReady or Widevine (Common Encryption)
  * **cbcs-aapl**: for FairPlay (AES CBC encryption)
  * **cbc**: for AES envelope encryption (Envelope Encryption)

## Next steps
The following articles describe next steps to get started with content protection:
* [Protect with storage encryption](media-services-rest-storage-encryption.md)
* [Protect with AES encryption](media-services-protect-with-aes128.md)
* [Protect with PlayReady and/or Widevine](media-services-protect-with-playready-widevine.md)
* [Protect with FairPlay](media-services-protect-hls-with-FairPlay.md)

## Related Links
[Azure Media Services PlayReady license delivery pricing explained](http://mingfeiy.com/playready-pricing-explained-in-azure-media-services)

[How to debug for AES encrypted stream in Azure Media Services](http://mingfeiy.com/debug-aes-encrypted-stream-azure-media-services)

[JWT token authentication](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[Integrate Azure Media Services OWIN MVC-based app with Azure Active Directory and restrict content key delivery based on JWT claims](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/).

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png
