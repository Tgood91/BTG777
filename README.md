BTG DAO
This is the on-chain governance system for BTG. In plain terms: it's a set of rules, written into smart contracts, that let token holders vote on decisions — and it has one twist. Voting power isn't just about how many tokens you hold. It's also boosted by a wallet's Bushido Virtue Score, an on-chain reputation record built up from real trading activity and scored across eight traditional virtues.
What's in this repo
Four pieces work together: two foundation layers, a registry that builds reputation from trades, and the voting engine that uses it.
BTG1 — the foundation
The base rulebook every DAO here follows: how funds move in and out, how permissions are granted, how the DAO's basic settings get updated. Nothing about virtues yet — just the plumbing.
BTG2 — the virtue layer
Builds directly on top of BTG1 and adds one new idea: every wallet has a Bushido Virtue Score, tracked across eight virtues:
Virtue
Japanese
Meaning
Righteousness
Gi
Justice / doing what's right
Courage
Yu
Heroic courage
Benevolence
Jin
Compassion
Respect
Rei
Politeness
Honesty
Makoto
Sincerity
Honour
Meiyo
Reputation, honour
Duty
Chugi
Loyalty
Self-Control
Jisei
Discipline
Virtue Audit Registry — where scores actually get built
BTG2 defines what a virtue score looks like; this contract is what actually creates one. Every time someone swaps tokens through the Bushido Trading Protocol, the trade gets scored across five of the eight virtues and permanently logged here. Those scores then accumulate onto the wallet's running total:
Comes from trading
Comes from elsewhere
Righteousness (Gi)
Honour (Meiyo)
Courage (Yu)
Duty (Chugi)
Benevolence (Jin)
Self-Control (Jisei)
Respect (Rei)

Honesty (Makoto)

The other three virtues aren't trade-related — they're set separately, for example based on governance participation. Anyone can look up a wallet's full trade history and score breakdown; nothing here is hidden.
MyGovernance — the voting engine
Where actual votes happen. Built from a well-established, widely-used open-source voting toolkit (OpenZeppelin), configured like this:
A short delay after a proposal is submitted before voting opens
About a week-long voting window
No minimum token holding required to submit a proposal
A safety delay after a vote passes, before the change can actually go into effect — giving the community a window to notice and react
How a trade turns into voting power
This is the full loop, start to finish:
Someone swaps tokens through the trading dashboard
The swap is scored across five virtues (execution quality, conviction, liquidity depth, slippage adherence, token transparency)
Those scores are sent to the Virtue Audit Registry, logged permanently, and added onto the wallet's running virtue total
When it's time to vote, MyGovernance reads that wallet's total and adds a voting bonus on top of their normal token balance:
Every 100 combined virtue points adds 1% extra voting weight
That bonus is capped at +100% — virtue can at most double your voting power, never more
A wallet with zero tokens gets no bonus either way — virtue boosts a vote, it doesn't create one out of thin air
So two wallets holding the same number of tokens can end up with different amounts of voting power, depending on their track record.
How a proposal actually moves through the system
Someone submits a proposal
After the short delay, voting opens for about a week
Everyone's vote is weighted by tokens held, plus their virtue bonus
If it passes, it goes into a holding period (the safety delay)
Once that holding period ends, anyone can trigger the change to actually happen
Trust & security notes
Only one designated address can write virtue scores. The Virtue Audit Registry restricts trade-logging and score updates to a "recorder" role — meant to be the trusted service that actually calculates scores from real trade data, never the trader themselves. That's what stops a wallet from just handing itself a perfect score.
That recorder address needs real protection. Whoever controls it can shape everyone's voting power, so it should sit behind something more robust than a single private key long-term — a multisig, for instance — even though the contract itself doesn't enforce that.
What this is built with
Solidity 0.8.20, used consistently across all files
OpenZeppelin's governance toolkit (Governor + extensions) — a widely-used, security-audited set of building blocks for DAOs, rather than writing voting logic from scratch
OpenZeppelin's AccessControl — powers the recorder-role permission system in the Virtue Audit Registry
A governance token that tracks voting history (so past balances can be checked fairly)
A timelock — the mechanism behind the safety delay mentioned above
Deployment order
Deploy the Virtue Audit Registry first, with your admin wallet address
Grant the recorder role to whichever service will be submitting trade scores
Deploy MyGovernance, passing the registry's address in as its virtue source
License
BTG1 and BTG2 carry an AGPL-3.0-or-later license, inherited from the Aragon framework they're adapted from. MyGovernance and the Virtue Audit Registry are MIT licensed. Worth double-checking license compatibility before combining these with other code in the repo.