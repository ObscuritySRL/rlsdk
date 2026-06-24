# rlsdk

TypeScript SDKs for reading Rocket League's Unreal Engine 3 objects out of live process memory. Each package is a generated map of the game's classes, structs, and enums — and, more to the point, the byte offset of every property — so you can walk real game objects with a memory reader like [`bun-memory`](https://www.npmjs.com/package/bun-memory).

There are two packages because there are two Rocket Leagues. The Epic Games and Steam builds are different binaries: the global object tables live at different addresses and the per-property offsets don't line up. The generated *shape* is identical, the *numbers* are not — so install the one that matches the client you're attaching to.

| Package | Targets |
| --- | --- |
| [`@rlsdk/epic-games`](packages/epic-games) | `RocketLeague.exe` — Epic Games Store |
| [`@rlsdk/steam`](packages/steam) | `RocketLeague.exe` — Steam |

## Install

```bash
bun add @rlsdk/epic-games bun-memory   # or @rlsdk/steam
```

These ship as raw `.ts` — no build step, no emitted `.d.ts`. They're meant for Bun (or any toolchain that reads TypeScript sources directly), which is also why `bun-memory` is a peer dependency instead of something they bundle. The SDK is nothing but types and offset constants; `bun-memory` does the actual reading and writing.

## Usage

```typescript
import Memory from 'bun-memory';
import { GNames, GObjects } from '@rlsdk/epic-games/offsets';
import { CarComponent_Boost_TA } from '@rlsdk/epic-games/offsets/TAGame';

const rl = new Memory('RocketLeague.exe');
const base = rl.modules['RocketLeague.exe'].base;

// GObjects/GNames get you from a class name to a live pointer; once you have
// a boost component, its fields are just base + offset:
const boost = rl.f32(boostComponentPtr + CarComponent_Boost_TA.CurrentBoostAmount);
```

Resolving that pointer in the first place — and the conventions worth understanding before you trust an offset — live in the package READMEs: how to read the `offset (size) [type]` comments, how UE3 packs several booleans into one bitfield, and how to walk GObjects/GNames. Start with [packages/epic-games](packages/epic-games/README.md) or [packages/steam](packages/steam/README.md). Steam also carries a full worked example at [`packages/steam/examples/basic.ts`](packages/steam/examples/basic.ts).

## Layout

```
packages/
  epic-games/   @rlsdk/epic-games
  steam/        @rlsdk/steam
```

The packages are independent — no shared code, no cross-import. The monorepo only exists to keep them, and their release process, in one place.

## Development

```bash
bun install        # once, at the root — sets up both packages
bun run typecheck  # tsc --noEmit across both packages
bun run format     # biome format --write .
bun run check      # biome check . — verify formatting
```

Formatting is [Biome](https://biomejs.dev) (see `biome.json`), pinned to 240-column LF.

## Publishing

Each package publishes on its own to npm under the `@rlsdk` scope:

```bash
bun run publish:epic-games
bun run publish:steam
```

## License

MIT © Stev Peifer
