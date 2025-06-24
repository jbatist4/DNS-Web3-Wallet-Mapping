# DNS to Web3 Wallet Mapping Proposal

This repository contains a detailed blueprint for standardizing the mapping of Web3 wallet addresses to domain names using the recently registered **WALLET RRType (IANA Type 262)**.

Our goal is to significantly improve Web3 usability and security by enabling human-readable wallet addressing directly through the DNS infrastructure.

---

## Key Highlights:

* **Leverages WALLET RRType (IANA 262):** Utilizes the newly official DNS record type.
* **Detailed Specification:** Provides a clear format (EBNF grammar) for `coin:address` pairs.
* **Robust Security:** Emphasizes reliance on DNSSEC for data integrity and authenticity.
* **Seamless Transition:** Includes a `_w3addr` TXT record fallback mechanism.
* **Multi-Coin Support:** Designed to handle multiple wallet addresses across different cryptocurrencies.

---

## Read the Full Proposal

For complete technical details, examples, and security considerations, please refer to the main proposal document:
[PROPOSAL.md](PROPOSAL.md) ---

## Support This Work

If this proposal aligns with your vision for a more user-friendly and secure Web3, and you find value in this foundational work, any support is greatly appreciated.

Your contributions help foster the development of open standards that benefit the entire decentralized ecosystem.

**Donation Addresses:**

* **Bitcoin (BTC):** `bc1qqh6hsh5q9lpxp2cs9ljlax5sxa456ctk2uj3qg`
* **Ethereum (ETH):** `0xf4B8FC6a857F864DD0944bf2333E986ea8224AC5`
* **Solana (SOL):** `8VDEqbz7yucGG6RWKEtEvRfxqYthn4cpkTJECQXY3EvX`

---

### License

This work is licensed under the [MIT License](LICENSE) (or your chosen license).
