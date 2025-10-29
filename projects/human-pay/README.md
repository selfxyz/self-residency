# HumanPay (Verifiable Multisend)

**Status:** Production-ready testnet
**What:** Onchain compliance infrastructure for token operations with zero-knowledge identity verification
**Repo:** https://github.com/0xleal/verifiable-multisend
**Demo:** https://humanpay.vercel.app
**Team:** @francisco
**Contact:** @francisco

**How Self fits:** Combine Self.xyz personhood proofs (passport/ID NFC verification) with ZK claims to enable compliant token distributions, DAO operations, and sybil-resistant airdrops. Users prove OFAC compliance, jurisdiction eligibility, and age requirements without exposing personal data on-chain. Enables DeFi protocols to enforce compliance rules directly in smart contracts.

## Quickstart

- Clone: `git clone https://github.com/0xleal/verifiable-multisend.git && cd verifiable-multisend`
- Install frontend: `cd web && npm install`
- Install contracts: `cd contracts && npm install`
- Run frontend: `cd web && npm run dev`
- Deploy contracts: `cd contracts && npx hardhat run scripts/deploy-self-verified-drop-celo-sepolia.ts --network celo-sepolia`

## Env

Create `web/.env` from example:

```bash
# Celo Network
CELO_SEPOLIA_RPC_URL=https://alfajores-forno.celo-testnet.org
NEXT_PUBLIC_CHAIN_ID=44787

# Supabase (Optional)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Self.xyz Configuration
NEXT_PUBLIC_SELF_HUB_ADDRESS=0x16ECBA51e18a4a7e61fdC417f0d47AFEeDfbed74
NEXT_PUBLIC_SELF_SCOPE_SEED=your-seed
NEXT_PUBLIC_SELF_CONFIG_ID=0x32332b93...
```

Create `contracts/.env`:

```bash
PRIVATE_KEY=your-private-key-here
CELO_SEPOLIA_RPC_URL=https://alfajores-forno.celo-testnet.org
CELOSCAN_API_KEY=your-celoscan-api-key (optional for verification)
```

## Milestones

- [x] v0 demo scope defined
- [x] Self.xyz proof verified end-to-end (30-day verification expiry)
- [x] Smart contracts deployed (Distribute + Airdrop modes)
- [x] Frontend for batch ETH distribution with CSV upload
- [x] Gas-optimized assembly batch operations
- [x] Public demo link (Vercel deployment)
- [x] Airdrop mode frontend (creator + claimer UI)
- [ ] ERC20 token support in frontend (USDC, cUSD, USDT)
- [ ] Mainnet deployment (Celo, Ethereum, Base, Optimism)

## Updates

- **Week 1 (Oct 20-27):** Onchain integration with Self.xyz + gas-optimized distribution method
- **Week 2 (Oct 28-Nov 3):** Frontend claim flow finalized, UI polished + multi-chain support (Celo + Base)
- **Week 3:** ERC20 token selector + mainnet deployment preparation

## Deployed Contracts

**Celo Sepolia (Testnet):**

- Registry: `0x9f0eA3fc541415BaacED50dacb06FFdc7ADced72`
- MultiSend: `0x5787B4FDcb1437Cc6fcEFAb845924E1697ed5fE2`
- Airdrop: `0x120e3798B2284f875659813101BB984D56c36022`
- Self.xyz Identity Hub V2: `0x16ECBA51e18a4a7e61fdC417f0d47AFEeDfbed74`
- Block Explorer: https://sepolia.celoscan.io

**Base Sepolia (Testnet):**

- Registry: `0x1b0e6584Caf81f1cBca7Fb390192aAa192074C4d`
- MultiSend: `0xF6368F4464f6D571aa1C061D5279b52F5FC79BE0`
- Airdrop: `0x0F6086351cbC9caF0b74d985a2876A6F14b08A70`
- Block Explorer: https://sepolia.basescan.org

## Resources

- **Self.xyz Docs:** https://docs.self.xyz
- **Celo Docs:** https://docs.celo.org
