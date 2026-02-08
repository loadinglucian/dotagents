# Package Manager Detection

## Detection Order

1. `bun.lockb` or `bun.lock` -> `bun`
2. `pnpm-lock.yaml` -> `pnpm`
3. `yarn.lock` -> `yarn`
4. `package-lock.json` -> `npm`
5. no lock file -> `bun`

## Command Mapping

### bun

- Install: `bun install`
- Add dep: `bun add <pkg>`
- Add dev dep: `bun add -D <pkg>`
- Run script: `bun run <script>`
- One-off: `bunx <tool>`

### pnpm

- Install: `pnpm install`
- Add dep: `pnpm add <pkg>`
- Add dev dep: `pnpm add -D <pkg>`
- Run script: `pnpm run <script>`
- One-off: `pnpm dlx <tool>`

### yarn

- Install: `yarn install`
- Add dep: `yarn add <pkg>`
- Add dev dep: `yarn add -D <pkg>`
- Run script: `yarn <script>`
- One-off: `yarn dlx <tool>`

### npm

- Install: `npm install`
- Add dep: `npm install <pkg>`
- Add dev dep: `npm install -D <pkg>`
- Run script: `npm run <script>`
- One-off: `npx <tool>`
