# DNS to Web3 Wallet Mapping: A Standard and Implementation Proposal

**Author:** Jorge Batista  
**Contact:** jbatist4321@gmail.com

---

## Executive Summary

This document presents a detailed proposal for an implementation standard that enables secure, scalable, and unbiased mapping of Web3 wallet addresses to domain names using the DNS system. The core solution revolves around the new **`WALLET RRType`**, which was officially registered by IANA (Internet Assigned Numbers Authority) on June 21, 2024, with type number 262. Additionally, a fallback mechanism using TXT records is suggested to ensure compatibility during the propagation and adoption phases.

The primary innovation of this proposal lies in the **technical detail and standardization of *how the WALLET RRType will be used***. It defines a clear format for associating wallet addresses with digital currencies and outlines how to manage multiple records for different assets under a single domain. The goal is to eliminate the current fragmentation and complexity in Web3 wallet addressing, drastically improve usability for millions of users, reduce critical errors, and strengthen security through reliance on DNSSEC. This specification offers a **clear and immediate path** for native integration of Web3 wallets into the existing DNS infrastructure, catalyzing mass adoption and trust within the decentralized ecosystem, especially relevant for the Ethereum ecosystem.

---

## 1. Introduction: The Usability Challenge in Mass Web3 Adoption

The increasing use of digital wallets and Web3 services exposes a critical gap in usability and security: the absence of a standardized, human-readable method for associating wallet addresses with domain names. The current reliance on long, alphanumeric character strings is a significant barrier to mass adoption, increasing the risk of typing errors and hindering the user experience. While solutions like the Ethereum Name Service (ENS) exist, they operate on a different layer and, while valuable, do not replace the need for a universal standard intrinsic to the global Domain Name System (DNS) infrastructure.

My proposal addresses this fragmentation by detailing the use of the newly registered **`WALLET RRType`** [IANA-WALLET-RRTYPE] to map Web3 wallet addresses to domain names. The foundation of this specification is the robust security provided by **DNSSEC** [RFC4033] [RFC9364], ensuring the integrity and authenticity of the mappings. The inclusion of a fallback mechanism via TXT records ensures a smooth transition and broad compatibility as the `WALLET RRType` propagates across DNS providers.

The objective is to provide an **easy-to-implement, scalable, unbiased, standardized, and inherently secure** solution for associating wallets with domain names. This paves the way for a more accessible and trustworthy internet, which is fundamental for the future of Ethereum and the entire Web3 space.

### 1.1. Requirement Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals.

---

## 2. Essential Terminology

This document will refer to domain name terminology [RFC9499]. Additionally, the following terms are defined for this context:

- **Web3 Wallet:** Software or hardware that stores public and private cryptographic keys, allowing users to interact with blockchains and manage digital assets.
- **Wallet Address:** The public identifier of a wallet on a specific blockchain, to which funds can be sent.
- **Coin Name:** Symbol or identifier for a cryptocurrency or token, as listed in standardized registries (e.g., [SLIP-0044]). For the Ethereum ecosystem, this includes "ETH" and ERC-20 token symbols.
- **`WALLET RRType`:** The new DNS resource record type (number 262) designated by IANA for wallet mapping.
- **DNSSEC:** DNS Security Extensions, a suite of specifications that adds layers of security to DNS, ensuring the authenticity and integrity of data.

---

## 3. Domain to Wallet Mapping: The Technical Solution

My proposal defines a standardized format for associating a domain name with Web3 wallet addresses, utilizing both the new `WALLET RRType` and a TXT record fallback, guaranteeing maximum compatibility and resilience.

### 3.1. Record Format

The `WALLET` or `TXT` record **MUST** follow this format:

@ IN WALLET "coin1:address1,coin2:address2"
@ IN TXT "_w3addr=coin1:address1,coin2:address2"


Where:

- **`@`**: Indicates the domain name to which the record applies (representing the root domain). Subdomains also can be used (e.g., `donate.example.com`).
- **`IN`**: Represents the Internet record class.
- **`WALLET / TXT`**: Is the resource record type.
- **`"coin:address"`**: Is the record value, containing one or more `coin:address` entries separated by commas. This format allows for a clear and flexible association between currencies and their respective addresses.

### 3.2. Grammar for the Record in EBNF Format

To ensure consistency, interoperability, and facilitate parsing by automated systems (like wallets and resolvers), the syntax of the record value adheres to the following grammar (Extended Backus-Naur Form):

```ebnf
list_of_items = item { "," item }
item          = (coin_name) ":" address
coin_name     = (letter | digit | "_")+
address       = (letter | digit | special_char)+
letter        = "A" | ... | "Z" | "a" | ... | "z"
digit         = "0" | ... | "9"
special_char  = "-" | "_" | "." | "/" | "+" | "=" | "$" | "#" | "@" | "!" | "&" | "*" | "(" | ")" | "[" | "]" | "{" | "}" | "<" | ">" | "?" | "~" | "`" | "|" | ";" | ":" | "," | "'" | "\"" | " "
```

- item: Represents a coin_name-address pair.

- list_of_items: Allows multiple coin:address pairs in the same record, optimizing the number of DNS queries.

- coin_name: Represents the symbol of a cryptocurrency or token. RECOMMENDED to use symbols registered in [SLIP-0044] (e.g., BTC, ETH, SOL) or widely recognized token identifiers (e.g., DAI, USDC for ERC-20 tokens). This MUST NOT be case-sensitive for matching purposes.

- address: Represents the public wallet address associated with a currency (e.g., "0xabcdefg12345", "bc1q..."). The address MUST be a valid string for the corresponding coin_name.

### 3.3. Example WALLET Record

Suppose an individual or organization wishes to map their BTC and ETH wallets (and ERC-20 tokens, using the same ETH address) to the domain example.com:
```
example.com. IN WALLET "BTC:bc1q1234567890abcdef,ETH:0xAbcDeF1234567890abcdef"
```

Alternatively, for finer granularity or administrator preference, individual WALLET records for each currency can be used:
```
example.com. IN WALLET "BTC:bc1q1234567890abcdef"
example.com. IN WALLET "ETH:0xAbcDeF1234567890abcdef"
```

### 3.4. Example TXT Record (Fallback for Compatibility)

To ensure maximum compatibility during the initial phase of propagation and adoption of the WALLET RRType by DNS resolvers and providers, a TXT fallback record SHOULD be used. The subdomain _w3addr MUST be used for this purpose:
```
_w3addr.example.com. IN TXT "BTC:bc1q1234567890abcdef,ETH:0xAbcDeF1234567890abcdef"
```
Or, with individual TXT records:
```
_w3addr.example.com. IN TXT "BTC:bc1q1234567890abcdef"
_w3addr.example.com. IN TXT "ETH:0xAbcDeF1234567890abcdef"
```
The TXT field MUST contain the string _w3addr= followed by the coin:address pairs, as per the defined grammar.

### 3.5. Handling Multiple and Duplicate Records

To support multiple assets or different addresses for the same currency (in cases like multisig, addresses on different networks/layers, or smart contracts acting as wallets), multiple coin:address pairs MAY be represented.
When a resolver finds multiple WALLET or TXT records for the same domain/subdomain, all valid coin:address pairs SHOULD be collected.

In the case of identical coin:address pairs (i.e., BTC:address1 appears twice), the resolver is RECOMMENDED to return only one instance of the pair to avoid redundancy.

The client application (wallet, dApp) MUST have logic to handle multiple addresses for the same currency, possibly prompting the user to choose or prioritizing according to predefined rules (e.g., preference for L2 addresses over L1, or specific contract addresses).

### 3.6. Implementation Guidelines for Resolvers and Applications

Implementations of DNS resolvers and client applications (such as Web3 wallets) utilizing this specification MUST:
Support the creation and retrieval of WALLET and TXT (_w3addr) records for any level of the DNS system.

Prioritize WALLET RRType queries over TXT (_w3addr) when both are available and valid, given that WALLET is the dedicated record type.

Validate records as properly signed by DNSSEC [RFC4033] [RFC9364], ensuring data authenticity and protecting against Man-in-the-Middle attacks.

Provide the wallet address for a human-readable domain name transparently.

Provide an authoritative NXADDR (or similar "not found" response) if no wallet address is found for the specified domain name and currency.

## 4. Security Considerations: Trust in DNS Infrastructure

Security is absolutely paramount for wallet address mapping, given the irreversible nature of blockchain transactions. The following measures are REQUIRED and emphasize the importance of DNSSEC:
The WALLET RRType MUST be stored in a secure DNS zone, mandatorily signed by DNSSEC [RFC4033] [RFC9364]. This ensures the authenticity of data origin and prevents tampering in transit. For the Ethereum Foundation, this aligns perfectly with the pursuit of robust, decentralized solutions.

Resolver implementations SHALL validate DNSSEC signatures to ensure the integrity and authenticity of the returned record. Failure to validate MUST result in rejection of the record.

While the WALLET RRType is official, its full propagation across the global DNS infrastructure will take time. The TXT (_w3addr) fallback mechanism offers a secure alternative, but security verification (via DNSSEC) still applies rigorously to the TXT record.

The wallet record MUST be protected against replay attacks through DNSSEC time invalidation (or approved successors).

Critical Attention: If the DNS zone origin (the registrar or the name server) is compromised, the wallet address mapping will also be compromised. It is imperative that domain administrators implement best security practices for their DNS zones, including strong authentication, two-factor authentication, and protection against unauthorized access.

## 5. IANA Considerations: Validation and Alignment

The WALLET RRType was officially registered by IANA (Internet Assigned Numbers Authority) on June 21, 2024, with type number 262. This fact not only validates the urgency and necessity of such a standard, but also positions this proposal as an immediate blueprint for the practical utilization of this newly designated record type. My specification provides detailed guidelines for the usage and formatting of data within this record type, requiring no new IANA changes beyond what has been established.

## 6. References
### 6.1. Normative References

[RFC1034] Mockapetris, P., "Domain Names - Concepts and Facilities", STD 13, RFC 1034, DOI 10.17487/RFC1034, November 1987.

[RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997.

[RFC3597] Gustafsson, A., "Handling of Unknown DNS Resource Record (RR) Types", RFC 3597, DOI 10.17487/RFC3597, September 2003.

[RFC4033] Arends, R., Austein, R., Larson, M., Massey, D., and S. Rose, "DNS Security Introduction and Requirements", RFC 4033, DOI 10.17487/RFC4033, March 2005.

[RFC8174] Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017.

[RFC9364] Hoffman, P., "DNS Security Extensions (DNSSEC)", BCP 237, RFC 9364, DOI 10.17487/RFC9364, February 2023.

[RFC9499] Hoffman, P. and K. Fujiwara, "DNS Terminology", BCP 219, RFC 9499, DOI 10.17487/RFC9499, March 2024.

### 6.2. Informative References

[SLIP-0044] "Registered coin types for BIP-0044", https://raw.githubusercontent.com/satoshilabs/slips/master/slip-0044.md.

[IANA-WALLET-RRTYPE] "DNS Parameter Registry - Resource Record (RR) Types", WALLET (Type 262), Assigned on June 21, 2024. (Official IANA link for WALLET RRType 262 will be inserted here when the corresponding IETF document is published and referenced by IANA).


### Appendix A. Conceptual Interaction and Implementation Example

This appendix illustrates a simplified example of how a client application (like a Web3 wallet or a dApp) could interact with the DNS system to resolve a wallet address using this specification. This demonstrates the viability and technical depth of the proposal. Note that this is a conceptual example to illustrate the logic, not production-ready code.
```
# Conceptual Python example for resolving WALLET/TXT records
import dns.resolver  # Would use a robust DNS library like dnspython
from dns.resolver import NXDOMAIN, NoAnswer

def resolve_web3_wallet_address(domain_name: str, coin_symbol: str) -> str | None:
    """
    Resolves a Web3 wallet address for a domain and coin symbol.
    Prioritizes WALLET RRType and uses TXT as fallback, with basic validation.
    """
    normalized_coin_symbol = coin_symbol.upper()  # Normalizes to uppercase

    # 1. Attempt to resolve WALLET RRType (new standard)
    try:
        # The dns.resolver library does not natively support WALLET yet,
        # so this is a conceptual simulation of how it would work
        # with a resolver that supported it.
        # In a real implementation, it would be something like:
        # answers = dns.resolver.resolve(domain_name, 'WALLET')
        
        # --- WALLET response simulation for example purposes ---
        mock_wallet_response = {
            'example.com': ["BTC:bc1qexamplebtc,ETH:0xAbcDeF1234567890abcdef"],
            'dapp.io': ["ETH:0x1234567890AbcDeF1234567890,USDC:0xFeDCBa0987654321"],
            'donations.org': ["ETH:0xf4B8FC6a857F864DD0944bf2333E986ea8224AC5", "SOL:8VDEqbz7yucGG6RWKEtEvRfxqYthn4cpkTJECQXY3EvX"]
        }
        
        if domain_name in mock_wallet_response:
            answers = mock_wallet_response[domain_name]
        else:
            raise NoAnswer  # Simulates that no WALLET record was found
        # --- End of simulation ---

        for rdata_value in answers:  # rdata_value would be the string from the record (e.g., "BTC:addr,ETH:addr")
            # Assume rdata_value is a string that might contain multiple pairs
            parts = rdata_value.split(',')
            for part in parts:
                if ':' in part:
                    coin, address = part.split(':', 1)
                    if coin.upper() == normalized_coin_symbol:
                        print(f"  -> WALLET RRType found: {coin}:{address}")
                        # Basic address validation (conceptual)
                        if len(address) > 10:  # Just a very basic check
                            return address
                        else:
                            print(f"    [WARNING] Invalid/short address for {coin}: {address}")
                            continue  # Try the next pair

    except (NXDOMAIN, NoAnswer):
        print(f"  WALLET RRType not found or empty for '{domain_name}'.")
    except Exception as e:
        print(f"  Error querying WALLET RRType for '{domain_name}': {e}")

    # 2. If WALLET is not found or valid, try the TXT fallback (_w3addr)
    print(f"Attempting TXT fallback for '_w3addr.{domain_name}'...")
    try:
        txt_qname = f'_w3addr.{domain_name}'
        txt_answers = dns.resolver.resolve(txt_qname, 'TXT')

        for rdata in txt_answers:
            txt_string = rdata.strings[0].decode('utf-8')  # TXT records return bytes
            if txt_string.startswith('_w3addr='):
                data = txt_string[len('_w3addr='):]
                parts = data.split(',')
                for part in parts:
                    if ':' in part:
                        coin, address = part.split(':', 1)
                        if coin.upper() == normalized_coin_symbol:
                            print(f"  -> TXT fallback found: {coin}:{address}")
                            # Basic address validation
                            if len(address) > 10:
                                return address
                            else:
                                print(f"    [WARNING] Invalid/short address for {coin}: {address}")
                                continue

    except (NXDOMAIN, NoAnswer):
        print(f"  TXT fallback '_w3addr.{domain_name}' not found or empty.")
    except Exception as e:
        print(f"  Error querying TXT fallback for '{txt_qname}': {e}")
            
    print(f"No address found for '{domain_name}' with coin '{coin_symbol}'.")
    return None

# --- Example Usage ---
if __name__ == '__main__':
    print("--- Test 1: Resolving ETH for dapp.io (via WALLET) ---")
    eth_addr_dapp = resolve_web3_wallet_address("dapp.io", "ETH")
    print(f"ETH address for dapp.io: {eth_addr_dapp}\n")  # Expected: 0x1234...

    print("--- Test 2: Resolving BTC for example.com (via WALLET) ---")
    btc_addr_example = resolve_web3_wallet_address("example.com", "BTC")
    print(f"BTC address for example.com: {btc_addr_example}\n")  # Expected: bc1q...

    print("--- Test 3: Resolving SOL for donations.org (via WALLET) ---")
    sol_addr_donations = resolve_web3_wallet_address("donations.org", "SOL")
    print(f"SOL address for donations.org: {sol_addr_donations}\n")  # Expected: 8VDE...

    print("--- Test 4: Resolving ADA for example.com (NOT FOUND) ---")
    ada_addr_example = resolve_web3_wallet_address("example.com", "ADA")
    print(f"ADA address for example.com: {ada_addr_example}\n")  # Expected: None

    print("--- Test 5: Resolving ETH for non-existent-domain.com (NOT FOUND) ---")
    non_existent_addr = resolve_web3_wallet_address("non-existent-domain.com", "ETH")
    print(f"ETH address for non-existent-domain.com: {non_existent_addr}\n")  # Expected: None
```

### Donations and Support

My passion is to see Web3 flourish through robust, user-friendly solutions. If this proposal resonates with the Ethereum Foundation's vision and you consider that this work contributes to the advancement of the ecosystem, any form of support is immensely appreciated.
Your contribution will help foster research and the development of standards that make Web3 more accessible and secure for everyone.

**Bitcoin (BTC):** bc1qqh6hsh5q9lpxp2cs9ljlax5sxa456ctk2uj3qg

**Ethereum (ETH):** 0xf4B8FC6a857F864DD0944bf2333E986ea8224AC5

**Solana (SOL):** 8VDEqbz7yucGG6RWKEtEvRfxqYthn4cpkTJECQXY3EvX


















