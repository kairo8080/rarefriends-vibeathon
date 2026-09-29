# DARKROOMS

Every room is pitch black. Your Rare Friend's light shows the holes for a moment, then goes out, and you walk it to the door from memory. A Key bought with RF opens the vault behind every door you reach.

**Builder:** [@kairo8080](https://github.com/kairo8080) · **Category:** Character Spotlight (also relevant to Economy Potential) · **SDK:** FriendSDK v0.1.4 (`spokesz/friendsdk@ca3bf18`)

**Play the preview: PREVIEW_URL** · [Source code](https://github.com/kairo8080/darkrooms/tree/SOURCE_SHA) · [Exact RF terms](https://github.com/kairo8080/darkrooms/blob/SOURCE_SHA/games/darkrooms/game.json) · [Game README](https://github.com/kairo8080/darkrooms/blob/SOURCE_SHA/games/darkrooms/README.md) · [Passing checks](CHECKS_URL)

**Wallet and network:** a browser wallet on **Robinhood mainnet (chain 4663)** holding a hardwired Rare Friends Generations NFT (**generation ≥ 1**). The SDK runtime connects the wallet and verifies ownership before play. No RF, ETH or signature is needed: the economy is simulated.

## Run it

Use Node.js 22+ on Linux or Ubuntu/WSL2:

```sh
git clone https://github.com/spokesz/friendsdk.git
cd friendsdk
git checkout ca3bf183b809ecf22d87c63d88ce03969a3f8da2   # FriendSDK v0.1.4
npm ci
git clone https://github.com/kairo8080/darkrooms.git ../darkrooms
git -C ../darkrooms checkout SOURCE_SHA
cp -R ../darkrooms/games/darkrooms games/darkrooms
npm run dev:game -- games/darkrooms
```

Open the printed URL (normally `http://localhost:4173`), connect your wallet, pick your Friend and choose **Enter the dark**. The hosted preview is the same game built with `friendsdk build` (see `vercel-build.sh` in the source).

## How it uses Rare Friends

The Friend is the only light in the game. The world is strict 1-bit black and white, and the Friend's canonical 16 × 16 Generations sprite (read through the SDK, black mask with a one-pixel white halo, whole-number scale) is the one thing drawn standing in its own pool of light. It walks and idles with all eight frames in four facings, turns to face every step, bonks against walls and sinks into the hole it steps on. The title card shows the Friend's family and token ID. Keys and relics belong to the Friend's canonical wallet, so they travel with the NFT.

## Play

1. **Memorize.** Each room starts lit for 2.6 s, 0.2 s less per room, never under 0.9 s. White tiles are holes, checkered tiles are walls, the door is on the far side.
2. **Walk in the dark.** Arrow keys or WASD move one tile per press. On touch screens, tap the side of your Friend you want to step toward. Walls and edges bonk harmlessly. A hole ends the run and the next one starts at Room 1.
3. **Open the vault.** At the door, spend one Key to open that room's vault and reveal a relic. Keep it or sell it back. Then take the next, harder room.

When a room ends, the whole map is revealed with the dotted path you walked, plus steps, near-misses and bonks: the moment to clip. Rooms get more holes (28% → 72%), more walls and a longer, twistier path. Landscape uses the 960 × 640 frame; phones in portrait get a 3 : 4 frame through `host.css`. Sound starts muted; mute and reduced motion are in Settings, and reduced motion removes the glide, shake and fall animations.

## How RF is spent, and the economy

**All balances, purchases and rewards are simulated.** You start with 20 RF and 100 RF of simulated prize backing.

**RF is spent in exactly one place:** a **Key costs 1 RF** and opens the vault behind one cleared door. Falling never costs a Key. One vault per cleared room, so every Key has to be earned by walking a dark room.

| Relic | Class | Chance | Redemption value |
|---|---|---:|---:|
| Burnt Match | Junk | 18% | 0 RF |
| Bent Nail | Common | 28% | 0.25 RF |
| Candle Stub | Common | 22% | 0.50 RF |
| Glass Eye | Uncommon | 16% | 1 RF |
| Silver Bell | Rare | 9% | 2 RF |
| Moth Lantern | Epic | 5% | 4 RF |
| The Last Light | Legendary | 2% | 10 RF |

Expected redemption value **0.92 RF per Key** (8% vault edge). Skill decides how often you reach a vault, never the odds inside it. Backing follows the SDK unchanged: every Key reserves the maximum 10 RF prize, kept relics stay backed with no redemption expiry, and new Key sales stop, taking no RF, when free stake cannot cover another maximum prize.

## What would be on-chain

Nothing is on-chain in this build. Going live needs no new contract: it is a deployment of the SDK's `ChanceGame` with this `game.json`, through the SDK's live runtime and the Friend's canonical wallet.

| In the game | SDK action | On-chain effect |
|---|---|---|
| Buy Keys | `buy` | Exact RF approval; RF moves from the Friend wallet into the game; 10 RF reserved per Key |
| Open the vault | `play` | Consumes the Key and commits a play; no outcome exists yet |
| *(same button)* | `settle` | Requests Dice randomness once, derives the relic on-chain and mints it to the Friend wallet |
| Sell a relic | `redeem` | Burns the relic and pays its fixed RF to the Friend wallet |

Rooms, steps and the run are off-chain presentation. A slow randomness delivery shows **Finish opening the vault**, resuming the same play without another Key.

## How randomness is used

Only a vault's contents are random, and only through the SDK's `play`/`settle`. Live, that is the SDK's Dice commit-then-reveal flow; in the preview, the SDK's simulated ledger draws the roll with `crypto.getRandomValues`. Room layouts come from a seeded generator (browser entropy for the seed) that always carves a walkable path to the door; they never touch RF outcomes.

## Needs future SDK support

- **Flares** (RF, burned on use): relight the current room once. Needs a second consumable.
- **Second Wind** (RF, burned): continue a run from the room you fell in. Needs a spend/burn action.
- **Brass Keys** for deeper vaults with a bigger table. Needs multiple consumable tiers.
- **Depth leaderboard** funded by a share of Key sales, and a burn or treasury split of the 8% edge. Needs persistence and a split action.

## Checks, credits and limitations

GitHub Actions on FriendSDK v0.1.4 ([latest run](CHECKS_URL)): `npm ci`, `npm run build`, `friendsdk check games/darkrooms`, `friendsdk build games/darkrooms`, `tsc` typecheck, and SDK mock-wallet browser checks at 960 px and 360 px (enter the dark, wait for lights out, step, settings, sound toggle, Key shop). **All pass.** A separate scripted playthrough at both sizes (memorize, walk the solved path, buy a Key, open the vault, reveal, next room, fall) reported no browser errors.

Browser checks use the SDK's mocked wallet and RPC. REAL_WALLET_STATUS Live mode has never run against a deployed contract.

Character sprites come from FriendSDK ([notice](https://github.com/spokesz/friendsdk/blob/main/NOTICE.md)). Rooms, door and relic pixel art are original and drawn in code; sounds are synthesized in code. No third-party assets. No trading, wearable NFTs, creator fees, persistence or live economy is included, and no Token Activity metrics are claimed. Production publication needs separate Rare Friends review.
