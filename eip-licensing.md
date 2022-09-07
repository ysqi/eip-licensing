---
eip: 52XX
title: NFT Licensing Standard
author:  Timi (@0xTimi), Robin, qige ...
discussions-to: https://github.com/ethereum/EIPs/issues/xxx
type: Standards Track
category: ERC
status: Draft
created: 2022-08-10
requires: 165
---


## Simple Summary

A standardized way to store and retrieve the granted licensing agreements for non-fungible token(NFT) derivative works, which are also NFTs but recreated based on some other underlying NFTs.  

In this standard, an NFT derivative work is hereinafter referred to as a "**dNFT**", while the original underlying NFT is hereinafter referred to as a "**oNFT**".

## Abstract

The NFT owner, knowns as "Licensor", may authorize another re-creator (person or entity), knowns as "Licensee", to use authorized rights to recreate derivative works (**dNFT**s),  in exchange for an agreed payment, known as a "Royalty". A licensing agreement is to outline the terms and conditions related to the deal between the licensor and licensee.

The following standard allows dNFTs' smart contracts ,  which support [ERC-721](./eip-721.md) and [ERC-1155](./eip-1155.md) interfaces,  to declare granted licensing agreements in a simple way.  This is intended for anyone, including but not limited to NFT marketplaces,  that wants to support traceable and verifiable licensing agreements. The standard also governs the terms and conditions related to royalty when **dNFT**s is sold or re-sold.

Any individual or marketplace is able to retrieve licensed credentials from a dNFT's contract with `authorizedBy()`, which specifies the detail of licensing agreement, include but not limited to  **oNFT**.  Those credentials can be verified in one **Registry** service, which implements this standard.

In general, there are three important roles in this standard,

- **oNFT**: an original underlying NFT. The holder of oNFT is Licensor.  An oNFT can be one arbitrary NFT which is not restricted by this standard.
- **dNFT**: an NFT derivative work recreated based on one or multiple oNFTs. the holder of dNFT is Licensee.
- **Registry**: a licensing agreement registry that is a smart contract as a service being able to verify whether a credential is signed or released by the holder of oNFT. A registry smart contract should have been audited by at least one creditable security audit platforms. 

Anyone can retrieve licensing royalty information with `licensingRoyalty` via the **Registry** *(? or from dNFT)*. It's worth noting that licensing royalty is different from royalty in [ERC-2981](./eip-2981.md) which declare royalty payment to creator while licensing royalty declares payment to licensors.  Despite It is hard to tell whether a transfer is a resale transaction or a gift or even a transfer to owner's another address such that the royalty payment may just be voluntary in practice, we encourage marketplaces to comply with licensing agreements to deliver royalties.

This ERC should be considered a minimal, gas-efficient building block for further innovation in NFT derivatives.

## Motivation

In general, some creators of NFTs will transfer intellectual property (IP), commercial, and exclusive licensing rights to the individual NFT holders. This enable NFT owners to create artworks and products by themselves. However, currently there isn't a standardized and clear way in which NFT owners are able to grant others rights of recreating derivative works based on their owned NFT. 

This standard allows the NFT owner, knowns as "Licensor", to have a standardized way to signal a licensing agreement to re-creators, knowns as "Licensee", in exchange for an agreed payment, known as a "Royalty".  With this licensing agreement,  the licensee has the right to recreate relevant dNFTs which clearly declare the authorized agreements.  Meanwhile, this declaration should be able to be verified by a creditable decentralized **Registry** service at any given time.  In such way,  the licensing royalty payment can be paid to licensors every time dNFT is sold or re-sold as regulated by the agreement. Taking the moment when the dNFT is minted as the cut-off point, the phase before is called the **Authorization** phase, and the subsequent phase is called the **Discovery** phase.

This standard defines the **Authorization** as a loose and optional interface, but the **Discovery** as a strict and compulsory interface. The reason here is that, no matter how the licensing agreement is licensed via a **Registry**, the most important thing is that marketplaces and application can adopt a  unified interface to discover and verify these agreements.

To be specific, to support standard **Discovery**, 

1. **dNFT** must implement `authorizedBy()` interface to return a list of 
- the credential which represent the authorized licensing agreements . 
  
- the address of  **Registry** contract which issued the credential. 
  
2. The credentials can be verified by **Registry** at any anytime where verification includes: 

   - the **oNFT** contract and ID
   - Licensor
   - validity of license / expiry date
   - date of signature
   - some identifier similar to `SPDX-License-Identifier: MIT `
   - type of licensing agreement
     - Non-Exclusive   *// default*
     - Exclusive
     - Sole
   - scopes, e.g.  artworks

*To implement an practicable **Authorization**, two recommend options are proposed in the **Specification** section. The first is that both the licensor and the licensee sign the license agreement at the same time. The second is that the licensor release the licensing agreement first, and then sells the licensing agreement to the licensee through auction or resale. A **Registry** can also use other mechanism to implement **Authorization** as long as it can fairly support the validation of licensing agreements following the standard **Discovery** as mentioned before.*

It is worth noting that an infringing **dNFT** may forge a credential with a faked **Registry**, where the faked **Registry** offers a forged verification for the infringing **dNFT**. In order to avoid this problem, marketplaces/applications must be sure that is a reliable **Registry** , e.g.  which is in a creditable list  or double checked by a reliable oracle service.

Without an licensing agreement standard, the NFT ecosystem will lack an effective means to verify whether an NFT derivative work is licensed or not, original NFTs' owners will not receive ongoing licensed fee they deserve,  recreators cannot clearly know ehether they are entitled to perform derivative creations, and NFT marketplaces aren't able to distinguish licensed derivative works from unauthorised ones so that it is hard to ward off infringements. This will hamper the growth of NFTs, damage the interests of the NTF owners, and demotivate recreators from minting new and innovative derivatives.

Enabling all NFT ecosystem to unify on a single licensing agreement standard will benefit the entire NFT ecosystem.

While this standard focuses on NFTs and compatibility with  [ERC-721](./eip-721.md), [ERC-1155](./eip-1155.md) and [EIP-2981](./eip-2981.md) standards, EIP-52xx does not require compatibility with [ERC-721](./eip-721.md), [ERC-1155](./eip-1155.md) and [EIP-2981](./eip-2981.md) standards. Any other contract could integrate with EIP-52xx to return licensing agreement information. ERC-52xx is, therefore, a universal licensing agreement standard for many asset types.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

Every **dNFT** contract must implement the **IERC52XXNFT** and **IERC165** inferfaces.

```solidity
pragma solidity ^0.6.0;
import "./IERC165.sol";

///
/// @dev Interface of NFT derivatives (dNFT) for the NFT Licensing Standard
///  Note: the ERC-165 identifier for this interface is 0xd584841c.
interface IERC55XXDNFT is IERC165 {

    /// ERC165 bytes to add to interface array - set in parent contract
    /// implementing this standard
    ///
    /// bytes4(keccak256("IERC52XXDNFT{}")) == 0xd584841c
    /// bytes4 private constant _INTERFACE_ID_IERC55XXDNFT = 0xd584841c;
    /// _registerInterface(_INTERFACE_ID_IERC55XXDNFT);
    
    /// @notice Get the number of credentials.
    /// @param _tokenId - ID of the dNFT asset queried
    /// @return _number - the number of credentials 
    function numberOfCredentials(
		uint256 _tokenId
    ) external view returns (
        uint256 _number
    );

    /// @notice Called with the sale price to determine how much royalty
    //          is owed and to whom.
    /// @param _tokenId - ID of the dNFT asset queried
    /// @param _credentialId - ID of the licensing agreement credential 
    /// @return _oNFT - the oNFT address where the licensing from
    /// @return _tokenID - the oNFT ID where the licensing from
    /// @return _registry - the address of registry which can varify this credential
    function authorizedBy(
        uint256 _tokenId,
        uint256 _credentialId
    ) external view returns (
        address _oNFT,
        uint256 _tokenId,
        address _registry
    );
    
}

interface IERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. This function
    ///  uses less than 30,000 gas.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```

Every **Registry** contract must implement the **IERC52XXRegistry** and **IERC165** inferfaces.

```solidity
pragma solidity ^0.6.0;
import "./IERC165.sol";

///
/// @dev Interface of NFT derivatives (dNFT) for the NFT Licensing Standard
///  Note: the ERC-165 identifier for this interface is 0xb5065e9f
interface IERC55XXRegistry is IERC165 {

    /// ERC165 bytes to add to interface array - set in parent contract
    /// implementing this standard
    ///
    /// bytes4(keccak256("IERC55XXRegistry{}")) == 0xb5065e9f
    /// bytes4 private constant _INTERFACE_ID_IERC55XXRegistry = 0xb5065e9f;
    /// _registerInterface(_INTERFACE_ID_IERC55XXRegistry);

    // TODO: Is the syntax correct?
    enum licensing_agreement_type {
      NonExclusive,
      Exclusive,
      Sole
    } 


    /// @notice 
    /// @param _dNFT - 
    /// @param _dNFT_Id - 
    /// @param _oNFT - 
    /// @param _oNFT_Id - 
    /// @return _licensed - 
    /// @return _tokenID - the oNFT ID where the licensing from
    /// @return _registry - the address of registry which can varify this credential
    function isLicensed(
        address _dNFT,
        uint256 _dNFT_Id,
        address _oNFT,
        uint256 _oNFT_Id
    ) external view returns (
        bool _licensed
    );
    
    /// @return _licenseIdentifier - the indentifier, e.g. `MIT` or `Apache`, 
    //                                similar to `SPDX-License-Identifier: MIT` in SPDX.
    function licensingInfo(
        address _dNFT,
        uint256 _dNFT_Id,
        address _oNFT,
        uint256 _oNFT_Id
    ) external view returns (
        bool _licensed,
        address _licensor,
        uint64 _timeOfSignature,
        uint64 _expiryTime,
        string _licenseIdentifier,
        licensing_agreement_type _type,
        string _scope  //TODO: or JSON data?
    );
    
    function royaltyRate(
        address _dNFT,
        uint256 _dNFT_Id,
        address _oNFT,
        uint256 _oNFT_Id
    ) external view returns (
        address beneficiary, 
        uint256 rate,   // TODO:怎么表示百分比？
        uint8 decimal
    );
}
```

This is the "ERC52XX licensing agreement Metadata JSON Schema" referenced above.

```json
{
    "title": "Licensing Agreement Metadata",
    "type": "object",
    "properties": {
        "licensor": {
            "type": "string",
            "description": "0x***"
        },
        "timeOfSignature": {
            "type": "string",
            "description": "the format conforms to ISO 8601"
        },
        "expiryTime": {
            "type": "string",
            "description": "the format conforms to ISO 8601"
        },
        "licenseIdentifier": {
            "type": "string",
            "description": "MIT,Apache..."
        },
        "type": {
            "type": "string",
            "description": "NonExclusive,Exclusive,Sole"
        },
        "scope": {
            "type": "string",
            "description": ""
        },
        "memo": {
            "type": "string",
            "description": ""
        },
    }
} // the strings of timeOfSignature
```

### Examples

#### Deploying an ERC-721 and signaling support for ERC-52XX

```solidity
constructor (string memory name, string memory symbol, string memory baseURI) {
        ....
    }
```

#### Checking if the NFT being sold on your marketplace is a DNFT implemented ERC52XXDNFT

```solidity
bytes4 private constant _INTERFACE_ID_IERC55XXDNFT = 0xd584841c;

function checkDNFT(address _contract) internal returns (bool) {
    (bool success) = IERC165(_contract).supportsInterface(_INTERFACE_ID_IERC55XXDNFT);
    return success;
 }
```

#### Checking if an address is a Registry implemented ERC52XXRegistry

```solidity
bytes4 private constant _INTERFACE_ID_IERC55XXRegistry = 0xb5065e9f;

function checkLARegistry(address _contract) internal returns (bool) {
    (bool success) = IERC165(_contract).supportsInterface(_INTERFACE_ID_IERC55XXRegistry);
    return success;
 }
```

## Rationale

### Optional licensing royalty payments

## Security Considerations

*in discussion.*

## Backwards Compatibility

This standard is compatible with current [ERC-721](./eip-721.md) , [ERC-1155](./eip-1155.md) and [ERC-2981](./eip-2981.md) standards.

## References

**Standards**

1. [ERC-165](./eip-165.md) Standard Interface Detection.
1. [ERC-721](./eip-721.md),
1. [ERC-1155](./eip-1155.md) 
1. [EIP-2981](./eip-2981.md)  NFT Royalty Standard

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
