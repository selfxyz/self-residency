# Openbands

**Status:** Building  
**What:** Anonymous proof-of-membership forum. Users prove attributes like nationality, age, company affiliation and more, to join gated communitites to post comment and engage in discussions.
**Repo:** https://github.com/openbandsxyz
**Demo:** https://openbandsv2.vercel.app/ 
**Team:** Stratos & Masanori  
**Contact:** X: @0xstratos @masaun2551


**How Self fits:** 
Users proves nationality and age with Self. Proofs are verified onchain and attributes are stored in the deployed contracts:

NATIONALITY_REGISTRY_CONTRACT_ADDRESS=0x5aCA8d5C9F44D69Fa48cCeCb6b566475c2A5961a
AGE_REGISTRY_CONTRACT_ADDRESS=0x72f1619824bcD499F4a27E28Bf9F1aa913c2EF2a

## Milestones

- [x] Updated frontend 
- [x] Self Protocol SDK and tested with mock passport data
- [x] Verify nationality via Self
- [x] Verify age via Self
- [x] Show users verified badges by querying from deployed contracts
- [x] Deployed on Vercel with oublic demo link
- [ ] Integrate verification button via Self for mobile users (e.g. Opera, Base, Farcaster app)
- [ ] Permissionless community creation based on verified atributes
- [ ] Deploy bridge contract from Celo to Base for proof verification via Hyperlane
- [ ] Store posts and comments on IPFS for censorship-resistance content

## Updates

Week 1: 
- Self Protocol integration on frontend
- Deployed proof of humanity contract to verify passport data
- Tested with mock passport data

Week 2: 
- Deployed the Nationality Registry contract on mainnet and tested with real passport and ID
- Updated frontend and UX
- Updated "My Badges" section to show users verified nationality badge
- Explored IPFS for data storage and planned the architecture

Week 3: 
- Deployed age registry contract on mainnet and tested with real passport and ID
- Updated "My Badges" section to show 18+ age badge in addition to nationality
- Integration and testing the contract that bridges the verification data from Celo to Base using Hyperlane (WIP)


## Contracts deployed

**Mainnet:**
Nationality Registriy Contract:
https://celoscan.io/address/0x5aca8d5c9f44d69fa48ccecb6b566475c2a5961a

Age Registriy Contract:
https://celoscan.io/address/0x72f1619824bcd499f4a27e28bf9f1aa913c2ef2a
