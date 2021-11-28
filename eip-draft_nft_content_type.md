---
eip: <to be assigned>
title: Identify NFT Content Type
description: NFT contracts to provide a contentType getter so clients can identify the type of NFT and how to fetch metadata
author: Louis Garoche <louis@garoche.me>
discussions-to: https://www.nftstandards.wtf
status: Draft
type: Standards Track
category: ERC
created: 2021-08-30
---

## Abstract
Define a standardized way to identify the underlying content of an NFT. The contract provides a getter field queried by clients (wallets, marketplaces...) to allow for underlying integrations into programs and tools along with rich integrations into applications and programs that can support propetary NFT content types as needed. This, for instance, would standardize on-chain MIDI or other protocols that are not supported by the metadata standard. The metadata standard can link to a website or tool that understands how to read this field and show it to the user on-chain. If platforms in the future support the midi standard through this content standard they can render it as needed. Autoglyphs used the metadata uri field to directly embed ascii art: this standard could have been used instead to prevent custom NFT rendering for sites that don't understand the content but give users a standard interface for verifying the content and downloading it from the NFT.

## Motivation
The ERC721/1155 standard defines a `tokenURI`/`uri` getter on the contract, pointing to a JSON document containing the NFT metadata according to a conventional schema. This works for many audio/image/video projects but this scheme falls short when a creator wants to combine on-chain and off-chain data. Even for fully on-chain data, building JSON documents from the smart contract code is quite costly and reading can be complicated. <br>
Every project has its own metadata requirements, and this EIP aims to unleash the creativity of NFT creators, by providing a common base to describe NFT content information.<br>

## Specification
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.<br>

Smart contracts implementing this ERC MUST implement the following interface.

```solidity
pragma solidity ^0.8.0;

struct ContentData {
    /// URI Referencing the content remotely
    string uri;
    /// SHA256 content hash
    bytes32 hash;
    /// Mime type of the content as a string "video/mp4"
    string mime;
    /// String of the content itself from an on-chain source
    string content;
}

/// Interface to show content getter for token
interface ITokenContent {
    /// @param tookenId token id to retrieve content for
    /// @return ContentData struct of content information
    function content(uint256 tokenId) external returns (ContentData memory);
}
```

It is RECOMMENDED that Smart contracts implementing this ERC also implement the ERC-165 interface, with the interface id ```???```, which is the result of ```bytes4(keccak256('content(string)')```.

The `mime` field is required if `content` is provided.<br>
If `uri` is provided, `hash` and `mime` fields are strongly recommended but not required. The `uri` field MUST conform to the RFC 3986 URI standard.<br>
At least one of `uri` or `content` are required to be provided.<br>

## Rationale
This design is inspired from the [IANA Media Types (formerly known as MIME types)](https://www.iana.org/assignments/media-types/media-types.xhtml), the same way an operating system recognizes supported formats with the available installed software, wallet or marketplace clients can check if they "support" a given NFT. The document does not intend to describe all possible NFT formats but provide an effective way to identify the NFT type. Creators are free to imagine new content types that provide additional on-chain or off-chain metadata content. The standard also provides a simple way of linking content for NFTs in a off-chain or on-chain method that is not reliant on metadata. This method can be used for linking large content types such as AI models, on-chain SVG that may break normal gas limits, or even code to the NFT. Most platforms should show these links as source full-resolution files or download links instead of attempting to render them.

### Type value
A struct being returned from the NFT will not much additional gas overhead and any field omitted can simply be left blank.<br>
    This proposal provides support for both on-chain and off-chain content to compliment the provided existing NFT metadata.<br>
    This also allows full provenance for on-chain HTML or text content that may need to be interperted by a metadata server or external file (such as metadata on IPFS)<br>
    By allowing arbitrary content-types, platforms can decide what to render from content as opposed to metadata. Content here can be `text/markdown`, for instance, and supporting platforms can render the markdown file alongside the metadata if recognized.

### Additional example
A NFT solely consisting of HTML content can be created with the following implementation:
```solidity
pragma solidity ^0.8.0;

import "./IERCContentType.sol";

abstract contract HtmlNFT is IERCContentType {
    
    /**
        @dev On-chain HTML rendering
    */
    function content(uint256 tokenId) public view override returns (ContentData) {
        return ContentData({
          mime: 'text/html',
          content: getHTMLContent(tokenId);
        });
    }
}
```
## Backwards Compatibility
This EIP does not introduce any backward compatibility with existing NFT standards. It extends these standards by adding the possibility of having additional metadata.<br>
NFT creators should keep the existing ERC721/1155 metadata URI schemes on top of new content types to compliment this standard and provide metadata associated with the content.
    
## Common questions
1. Why use a struct instead of multiple getter functions: it allows for easier fetching (one fetch call vs up to 4) from clients and more gas-efficent code for deployment
2. What is this inspired by: This is inspired by the larvalab's Autoglyphs project, returning the ascii art in the given metadata URI, this would be a perfect use case for that type of on-chain data. Zora's cryptomedia that seperates the content uri and hash allowing for updatable URIs and more rich support of different content-types.

## Reference Implementation
See the example in the Specification section.

## Security Considerations
Clients that decide to support and implement executable content (binary, javascript...) must ensure that proper security measures are in place before running any code on the client.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
