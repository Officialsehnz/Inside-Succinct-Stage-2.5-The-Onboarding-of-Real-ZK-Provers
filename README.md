# Inside Succinct Stage 2.5: The Onboarding of Real ZK Provers
A technical report on Succinct’s Stage 2.5 testnet, where real GPU provers stake $PROVE tokens to generate SP1 proofs in a live marketplace. Includes setup guide, early feedback, and performance insights.

**Author:** \[Sehnz]
**Date:** June 2025

* [Twitter:] (https://x.com/tradewithsehnz)
* [Discord:] @officialsehnz

Let's hit it

## 1. Architecture & Objectives

- **Testnet evolution:** Stage 2 (“Prove with Us”) was a gamified testnet where users ran *virtual rigs* (simulated GPU/CPU/RAM) to earn points and stars by fulfilling “proof contests”. Stage 2.5 is the first *fully real* test of the Succinct Prover Network: an end-to-end marketplace where provers run actual proof-generating nodes. According to Succinct, “Stage 2.5 … will feature an end-to-end deployment of this system”.
- **New features:** In Stage 2.5, developers can directly *request zk-proofs*, and provers must *stake $PROVE* tokens, bid in real auctions, and fulfill proofs on demand. This completes the two-sided network (off-chain auctioneer + on-chain settlement) described in their architecture. In short, Stage 2.5 shifts from a simplified “proof contest” game to the actual proof-generation network with real cryptography and token economics.

## 2. Participation Requirements

- **Hardware:** A GPU-equipped machine is required. Competitive provers use top-tier GPUs (NVIDIA RTX 4090 or data-center cards like A10/L4) with powerful multi-core CPUs and ≥16 GB RAM per GPU. In principle even a mid-range GPU (e.g. RTX 3090) can run the reference node, but throughput (and thus rewards) will be lower.
- **PROVE tokens (testnet):** Each prover must **stake $PROVE**. On Sepolia, you need **≈1000 PROVE** to bid on proofs. A faucet is available for testnet PROVE (as noted in the Quickstart docs). Without enough PROVE staked, your node can’t participate.
- **Wallet:** A fresh Ethereum account with Sepolia ETH is required. (The private key is loaded into the prover CLI, so **don’t use a main wallet**.) You’ll use this wallet to create the prover, stake tokens, and pay transaction gas.
- **Whitelisting/access:** Early Stage 2.5 was a limited rollout. Active Stage 1 provers and community members were auto-whitelisted for Stage 2, and Succinct indicated they’d “onboard provers in Stage 2.5” in the weeks following the explorer release. In practice Stage 2.5 was initially available only to invited/private testers before a public release.

## 3. Node Setup & Tools

- **Software:** Succinct provides a reference “spn-node” implementation. It runs on Linux/macOS via Docker or building from source. The recommended Docker image is `public.ecr.aws/succinct-labs/spn-node:latest-gpu` (GPU-enabled).
- **Prover creation:** You create a new prover on-chain (the owner of the node) by calling `createProver`. Succinct recommends using their web frontend at [staking.sepolia.succinct.xyz/prover](https://staking.sepolia.succinct.xyz/prover) for this. (Alternatively, you can call the contract via a CLI or Etherscan.) You’ll then stake your PROVE tokens to that prover (also via the web UI or contract call).
- **Calibration:** Before running proofs, you *calibrate* your prover to set bidding parameters. The CLI has a `calibrate` command that benchmarks your GPU and suggests a bid price. This uses sample proof workloads.
- **Running the node:** Finally, you start the proving loop (`spn-node prove`) supplying your RPC URL, bidding price, throughput (from calibration), prover address, and private key (from your fresh wallet). This can be done via Docker or by running the compiled binary. The node will continuously bid on new proofs it can handle.
- **Interfaces:** Aside from the staking dashboard, there’s no full GUI. Monitoring is done via the CLI logs and the **Succinct Explorer**. The Explorer (live at [explorer.succinct.xyz](https://explorer.succinct.xyz)) shows proofs in flight, completed proofs, and trends (proof throughput, prover success rates, etc.) in real time.
- **Documentation:** Succinct’s official docs cover every step (see the Running-a-Prover Quickstart). These detail prerequisites (wallet, tokens), installation, and CLI commands.

## 4. Early Participant Feedback

- **Performance:** Early testers report that proofs are being fulfilled and tracked. The network Explorer shows a growing number of completed proofs and overall prover success rates. Some community posts mention very high proof-fill rates (e.g. “100% win rate” on a 3090 GPU), though official stats are not yet published.
- **Stability:** A few users noted needing to occasionally restart the prover process (likely to avoid issues with long-running GPU tasks). Otherwise, no widespread outages have been reported. One caution: because the CLI uses your private key, users emphasize using a throwaway wallet.
- **Times & Throughput:** Real ZK proofs (SP1) can take minutes on even a fast GPU. Participants note that Stage 2.5 proofs are far heavier than Stage 2’s instant contests. Throughput scales with hardware – e.g. a 4090 will generate proofs significantly faster than a 3090.
- **Success rates:** The Explorer’s “Trends” page explicitly tracks the fraction of proofs successfully delivered by provers. At launch, with few provers, almost all proofs are filled quickly. As load increases, this metric will show if demand outstrips prover supply.

## 5. Limitations & Risks

- **Testnet conditions:** This is still a testnet. All tokens are testnet-only, and many protections (like stake-slashing on missed proofs) are disabled on Sepolia. Thus the incentives are not “real” yet.
- **Hardware demands:** The docs note that *truly* competitive proving setups may require dozens of GPUs or even specialized hardware. A single consumer GPU can participate, but may earn little once more provers join.
- **Complexity:** Setting up a node involves many manual steps and careful configuration (wallets, CLI flags, calibration). Mistakes (like mis-setting throughput or bid) can lead to poor performance. Documentation is thorough, but less-technical users will find the learning curve steep.
- **Security caution:** Since the CLI needs your private key, any key compromise risks your staked funds. Hence Succinct’s advice to use a *fresh* wallet.
- **Edge-cases:** Because this is an early release, some behaviors may not be fully documented. For example, current auctions are reverse-auctions (lowest bid wins) by design, but future phases might change this. Testers should monitor announcements for any rule changes.

## 6. Scalability Considerations

- **Architecture:** Succinct’s vApp design (off-chain auctioneer + periodic on-chain proofs) is meant to scale high throughputs. In theory, the auctioneer can match many proofs in real time without on-chain congestion, and only root proofs are posted on Ethereum.
- **More users:** As more provers and requesters enter, the network should see higher overall proof volume. Individual provers will face stiffer competition (bids may fall to very low PROVE amounts). The Explorer’s trend charts will reflect this – e.g. total proofs/time rising, success rate possibly dipping if demand spikes.
- **Performance monitoring:** The Explorer explicitly shows metrics (like daily proofs and success rates) to help gauge scaling effects. This transparency will help both developers and provers tune behavior as the network grows.
- **Network load:** While Succinct’s team is testing performance limits, actual stress will come with the mainnet. Potential bottlenecks include the off-chain auctioneer service and Ethereum settlement (many proofs posted). But Succinct plans suggest their off-chain layer can handle web-scale volumes, and only succinct SNARKs hit the blockchain.
- **Future adjustments:** As Stage 2.5 feedback comes in, Succinct may adjust parameters (e.g. staking requirements or auction format) before mainnet. Provers should be prepared for evolving rules, and keep an eye on community channels for updates.

---

**Sources:** Official Succinct Labs blog posts and documentation, along with early community discussions. These details summarize the technical setup and early experience of the Stage 2.5 testnet.
