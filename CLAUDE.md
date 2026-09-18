# lichess.org (lichess-org/lila)

## 프로젝트 개요
전 세계 수천만 명의 체스 팬들이 매일 광고 없이 100% 무료로 실시간 대국을 즐기는 "글로벌 1위 오픈소스 온라인 체스 경기장"
유료 결제 유도나 상업적 방해물 없이 가장 순수하고 공정한 환경에서 AI 분석과 전 세계 체스 기사들과의 대전을 지원
오픈소스 정신이 어떻게 세계 최고 수준의 대규모 글로벌 웹 서비스를 성공시킬 수 있는지 보여주는 살아있는 전설

## 핵심 특징 & 추천 분야
- 글로벌1위무료체스
- 광고없는실시간대국
- 오픈소스성공신화
- 전세계수천만명경기장
- 공정한체스생태계

---
*이 문서는 오픈소스 큐레이터(Curator-Agent)에 의해 자동 생성된 가이드 문서입니다.*


---
## 기존 CLAUDE.md 내용

# Lila Development Guide for Coding Agents

## Repository Overview

Lila (li[chess in sca]la) is the free, open-source chess server powering lichess.org - one of the world's largest chess platforms with 12+ billion games. This is a production-scale application serving millions of users with real-time gameplay, computer analysis, tournaments, and comprehensive chess features.

**Technology Stack:**

- **Backend**: Scala 3.8.2 with Liplay 3.2 (Lichess Play Framework fork), SBT build system
- **Frontend**: TypeScript with Snabbdom, PNPM workspace monorepo
- **Database**: MongoDB with Elasticsearch indexing
- **Real-time**: WebSocket connections via separate lila-ws server, Redis for communication
- **Chess Engine**: Stockfish via distributed fishnet cluster
- **Styling**: Sass with custom build pipeline

## Environment Requirements

**CRITICAL**: These exact versions are required - the build will fail without them:

- **Java 21** (JDK, not JRE - needs jdk.compiler module)
- **Node.js 24.15.0+** (specified in `.node-version`)
- **PNPM 11.22.0+** (specified in `package.json`)

**Installation:**

```bash
# Install PNPM globally
npm install -g pnpm@11.22.0

# Install dependencies (always run this first)
pnpm install
```

## Build & Development Commands

**Frontend (UI) Development:**

```bash
# Build all UI packages (requires Node 24+)
./ui/build

# Build specific packages
./ui/build site analyse lobby

# Watch mode for development
./ui/build -w

# Production build
./ui/build -p

# No dependency installation (faster rebuilds)
./ui/build --no-install
```

**Backend (Scala) Development:**

```bash
# Start development console (lila.sh is the SBT wrapper)
# Automatically copies .sbtopts.default and conf/application.conf.default if missing
./lila.sh

# In SBT console (via ./lila.sh), compile and run:
sbt> compile
sbt> run

# Run with specific JVM options
./lila.sh -Depoll=true

# Full build with tests and staging
./lila.sh "test;stage"
```

**Testing & Quality:**

```bash
# Run all frontend tests (Vitest)
pnpm test

# Watch mode for tests
pnpm test:watch

# Check code formatting (Oxfmt)
pnpm check-format

# Auto-format code
pnpm format

# Lint TypeScript code
pnpm lint

# Auto-fix lint issues
pnpm lint:fix

# Backend formatting check (Scalafmt) - via lila.sh wrapper
./lila.sh scalafmtCheckAll

# Auto-format Scala code
./lila.sh scalafmtAll
```

## Project Architecture

**Module Structure (80+ Scala modules):**

- Modules are in `/modules/[name]/src/main/` with strict dependency hierarchy
- Core modules: `core`, `common`, `db`, `memo`, `ui`
- Domain modules: `game`, `user`, `tournament`, `study`, `puzzle`, etc.
- Build order matters - see `build.sbt` for dependency graph

**Frontend Structure (36 UI packages):**

- `/ui/[package]/` - Individual TypeScript packages
- `/ui/lib/` - Shared utilities and types
- `/ui/@types/` - TypeScript definitions
- `/ui/.build/` - Custom build system with esbuild
- Each package has `package.json` with custom "build" property

**Key Directories:**

- `/app/` - Play Framework controllers, views, main application
- `/conf/` - Configuration files, routes, application settings
- `/public/` - Static assets, will contain compiled CSS/JS after build
- `/bin/` - Utility scripts (deploy, CLI tools, git hooks)
- `/project/` - SBT build configuration (BuildSettings.scala, Dependencies.scala)

## Common Issues & Solutions

**Node Version Error:** `Nodejs v24.15.0 or later is required`

- Install Node 24.15+ using nvm or package manager
- Check with: `node -v`

**Java Version Issues:**

- Ensure Java 21 JDK is installed (not JRE)
- Check: `java --list-modules | grep jdk.compiler`
- Set JAVA_HOME properly

**Build Failures:**

- Always run `pnpm install` after pulling changes
- For UI issues: `./ui/build --clean` then rebuild
- For Scala issues: `sbt clean compile`

**PNPM Workspace Issues:**

- Dependencies use `workspace:*` for internal packages
- Run `pnpm install` from repository root, not subdirectories

## Development Workflow

**Making Changes:**

1. **Frontend**: Edit files in `/ui/[package]/src/`, run `./ui/build -w` for live reload
2. **Backend**: Edit `/modules/[name]/src/main/`, use `sbt ~compile` for auto-recompile
3. **Styles**: Edit `.scss` files, included in UI build process
4. **Configuration**: Edit `/conf/` files, may require server restart

**Before Committing:**

1. Run `pnpm check-format` and `pnpm lint`
2. Run `./lila.sh scalafmtCheckAll`
3. Run `pnpm test` for frontend tests
4. Consider `./lila.sh test` for affected backend modules (can be slow)

**Utility Scripts:**

- `/bin/deploy` - Production deployment automation
- `/bin/trans-lint` - Translation validation
- `/bin/git-hooks/` - Git hooks for formatting/linting

## GitHub Actions & CI

**Workflows validate:**

- **server.yml**: Scala compilation, tests, formatting with Java 21
- **assets.yml**: UI build, tests, formatting with Node 24+
- **lint.yml**: Code quality checks

**Deployment:**

- Uses `/bin/deploy` script with workflow artifacts
- Server builds create `lila-3.0.tar.zst` artifact
- Asset builds create `assets.tar.zst` artifact

## Key Configuration Files

- **build.sbt**: Main SBT configuration, module definitions
- **package.json**: Root package with scripts and dependencies
- **pnpm-workspace.yaml**: Defines workspace packages
- **ui/.build/**: Custom frontend build system
- **.scalafmt.conf**: Scala formatting rules
- **ui/.oxlint.json**: TypeScript formatting rules (Oxlint)
- **ui/.oxfmt.json**: TypeScript/CSS/JSON/MD formatting rules (Oxfmt)
- **conf/routes**: HTTP route definitions
- **conf/application.conf.default**: Main application configuration template

## Development Tips

- **Large codebase**: Use specific module builds rather than full recompilation
- **Asset compilation**: The `./ui/build` system is sophisticated - study `/ui/README.md` for details
- **Hot reloading**: Backend changes require manual restart, frontend has watch mode
- **Module dependencies**: Check `build.sbt` before adding cross-module dependencies
- **Database**: Uses MongoDB - no migrations, but schema assumptions in code
- **Performance**: This is a high-traffic production system - consider performance impact

**Trust these instructions** - they are validated and comprehensive. Only search for additional information if these instructions are incomplete or incorrect for your specific task.
