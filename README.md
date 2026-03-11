TEC-CHAIN CORE ENGINE
1. Project Overview
TEC-CHAIN is a modular Layer-1 blockchain infrastructure designed for high-performance decentralized finance and enterprise-level data integrity. The network employs a specialized Hybrid Consensus mechanism combining Nominated Proof of Stake (NPoS) for governance and Proof of Work (PoW) for block security.

2. Technical Specifications
Chain Name: TEC-CHAIN

Native Token: TEC (18 Decimals)

Consensus: Hybrid PoW + NPoS

Block Time: 10 Seconds

Accounting Cycle: 45 Minutes (270 Blocks per Cycle)

Validator Group: 27 Nodes (Rotation based)

Total Supply: 200,000,000 TEC

Pre-Distribution: 10,000,000 TEC

Mining Allocation: 190,000,000 TEC

Validator Threshold: 2000 TEC Stake

3. Core Protocol Implementation
   package tec_kernel

import (
    "crypto/sha256"
    "math"
    "math/big"
    "sync"
    "time"
)

// Constants defining the TEC-CHAIN economy
const (
    ChainID            = "TEC-CHAIN-MAINNET"
    Currency           = "TEC"
    MinStakeRequired   = 2000.0
    ConsensusGroupSize = 27
    BlocksPerTurn      = 10
)

// TECAccount represents the state of an address in the ledger
type TECAccount struct {
    Address     [20]byte
    Balance     *big.Int
    Staked      *big.Int
    VoteWeight  *big.Int
}

// RewardManager calculates the dynamic emission of TEC
type RewardManager struct {
    DecayRate float64
}

// GetSubsidy returns the block reward based on height (15% annual decay)
func (rm *RewardManager) GetSubsidy(height int64) *big.Int {
    const BlocksPerYear = 315360
    year := height / BlocksPerYear

    var reward float64
    if year < 10 {
        // Initial reward 10 TEC/block, 15% decay annually
        reward = 10.0 * math.Pow(0.85, float64(year))
    } else {
        // Post Year 10: Fixed 3M TEC per year
        reward = 3000000.0 / float64(BlocksPerYear)
    }

    // Convert to 10^18 precision (Wei equivalent)
    f := new(big.Float).Mul(big.NewFloat(reward), big.NewFloat(1e18))
    result, _ := f.Int(nil)
    return result
}

// NPoSEngine handles validator rotation every 45 minutes
type NPoSEngine struct {
    mu          sync.Mutex
    ActiveNodes []string
    Pool        map[string]*TECAccount
}

// RotateValidators performs NPoS election based on stake and votes
func (n *NPoSEngine) RotateValidators() {
    n.mu.Lock()
    defer n.mu.Unlock()

    // Logic: Rank candidates by (Staked + Votes)
    // Select Top 27 nodes as the next accounting group
    // Each node produces 10 blocks sequentially
}
4. Economic Policy
The TEC-CHAIN economic model is designed for long-term scarcity and security.

Distribution: 5% Genesis Pre-mining, 95% Mined via Block Rewards.

Halving Logic: Unlike the sharp halving of Bitcoin, TEC-CHAIN uses a 15% smoothing decay mechanism annually for the first decade to ensure miner revenue stability.

Finality: After 10 years, the network enters a "Tail Emission" phase with a fixed 3,000,000 TEC annual issuance to cover operational costs.

5. Network API Interface
Blockchain Status
GET /v1/chain/info
Returns current block height, cycle index, and circulating supply.

Account Ledger
GET /v1/ledger/balance/{address}
Returns TEC balance, staking status, and nonce.

Consensus Operations
POST /v1/consensus/stake
Requires 2000 TEC. Registers the sender as a validator candidate in the NPoS pool.

POST /v1/consensus/vote
Nominate a validator candidate to increase their weight in the next cycle.

6. Project Directory Structure
/cmd: Entry point for the TEC node software.

/core: Blockchain ledger and transaction processing.

/consensus: Hybrid PoW + NPoS protocol implementation.

/p2p: Peer-to-peer discovery and gossip protocols.

/tokenomics: Reward calculation and emission schedules.

/api: RESTful gateway for dApp integration.
