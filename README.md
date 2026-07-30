# Bonsai Demo

<p align="center">
  <img src="./assets/bonsai-logo.svg" width="280" alt="Bonsai">
</p>

<p align="center">
  <a href="https://prismml.com"><b>ウェブサイト</b></a> &nbsp;|&nbsp;
  <a href="https://github.com/PrismML-Eng/Bonsai-demo"><b>GitHub</b></a> &nbsp;|&nbsp;
  <a href="https://discord.gg/prismml"><b>Discord</b></a>
</p>

<p align="center">
  <b>HF コレクションズ:</b>
  <a href="https://huggingface.co/collections/prism-ml/bonsai-27b">Bonsai 27B</a> ·
  <a href="https://huggingface.co/collections/prism-ml/bonsai">Bonsai (1-ビット版)</a> ·
  <a href="https://huggingface.co/collections/prism-ml/ternary-bonsai">Ternary-Bonsai</a>
</p>

<p align="center">
  <b>白書類:</b>
  <a href="bonsai-27b-whitepaper.pdf">Bonsai 27B</a> ·
  <a href="1-bit-bonsai-8b-whitepaper.pdf">1-ビット版 Bonsai 8B</a> ·
  <a href="ternary-bonsai-8b-whitepaper.pdf">Ternary-Bonsai 8B</a>
</p>

---


本デモ・リポジトリを用いれば、あなたは **Bonsai** (1-ビット版) および **Ternary-Bonsai** 言語モデル達をMac (Metal), Linux/Windows (CUDA, Vulkan, ROCm), もしくはCPU上でローカルに実行できます。

## 🌱 New: Bonsai 27B

The family's newest and largest generation, and its first **vision-language** models ([Bonsai 27B collection](https://huggingface.co/collections/prism-ml/bonsai-27b)):

- **Vision:** send photos, screenshots, and PDFs; ask about them (see [VISION.md](VISION.md)).
- **Agentic tool calling:** native OpenAI-style `tool_calls` with full round-trips, plus MCP servers in both demo UIs (see [TOOLS.md](TOOLS.md)).
- **Thinking:** a reasoning model; pick the reasoning effort per chat in the UI or budget it per request.
- **Long context:** 256k+ token conversations.
- **Tiny footprint:** the 1-bit Bonsai-27B packs to ~1.125 bits per weight: it fits on a modern iPhone without memory offloading. Ternary-Bonsai-27B (~1.7 bits per weight, packed into 2-bit for fast accelerated kernels) is the higher-quality option and this demo's default.

Quick Start below gets you there in two commands: `./setup.sh` downloads Ternary-Bonsai-27B by default, then `./scripts/start_llama_server.sh` gives you chat, vision, and tools at http://localhost:8080.

## クイック・スタート

AIコーディング・エージェントとしてのセットアップをご所望ですか？ (それなら)それに(エージェント向けに〈ハードウェアに特有のツマミ、既定値、そしてユーザに尋ねるべき内容が〉書かれたガイド)、[AGENTS.md](AGENTS.md)を指示・参照させてください。

### macOS / Linux

```bash
git clone https://github.com/PrismML-Eng/Bonsai-demo.git
cd Bonsai-demo

# (オプション値) モデルサイズを選択: 27B (既定値), 8B, 4B, あるいは 1.7B
export BONSAI_MODEL=27B

# あなたの HuggingFace トークンをセット (コレが必要なのはリポがプライベート化されてる27Bモデルのみ)
export BONSAI_TOKEN="hf_your_token_here"

# １コマンドが全てやってのける: 依存関係インストール, モデル + バイナリのダウンロード
./setup.sh
```

### Windows (PowerShell)

```powershell
git clone https://github.com/PrismML-Eng/Bonsai-demo.git
cd Bonsai-demo

# (オプション値) モデルサイズを選択: 27B (既定値), 8B, 4B, あるいは 1.7B
$env:BONSAI_MODEL = "27B"

# あなたの HuggingFace トークンをセット (コレが必要なのはリポがプライベート化されてる27Bモデルのみ)
$env:BONSAI_TOKEN = "hf_your_token_here"

# セットアップ実行
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\setup.ps1
```

### ファミリーとサイズの切り替え

あなたはTernary版 (既定値) と1-ビット版ファミリー間および、異なるモデル・サイズ間でも瞬時に切り替えられる:

```bash
# Ternary-Bonsai 4Bの実行
BONSAI_FAMILY=ternary BONSAI_MODEL=4B ./scripts/download_models.sh
BONSAI_FAMILY=ternary BONSAI_MODEL=4B ./scripts/run_llama.sh -p "Hello!"
```

Windows用:
```powershell
$env:BONSAI_FAMILY="ternary"; $env:BONSAI_MODEL="4B"
.\setup.ps1
.\scripts\run_llama.ps1 -p "Hello!"
```

---

## 速度ベンチマーク

種々のハードウェアにおける結果とあなたの測定結果の提出用テンプレートについては [community-benchmarks/](community-benchmarks/) を参照してください。

## モデル群

２つのモデル・ファミリー群が利用可能であり, それぞれのサイズは**27B**, **8B**, **4B**, そして **1.7B**です。このうち27Bモデルは視覚言語モデルであり: それらは文書同様、画像を受け入れます; 全27Bリポは[Bonsai 27B HF コレクション](https://huggingface.co/collections/prism-ml/bonsai-27b)に集められてます。

両形式ともllama.cppのメインラインに実装されつつあります: **Q1_0 (1-ビット版) は完全にアップストリームにマージされ**, そして **Q2_0 (ternary版) は現在はメインラインCPU、Metal、Vulkanで動作し**, CUDAについては現在レビュー中です。詳細およびメインライン互換ファイル: [binary版の現況](#アップストリームの現況binary版向け) と [ternary版の現況](#アップストリームの現況ternary版向け) は後述にて。

### Bonsai (1-ビット版)

GGUF (llama.cpp) および MLX 1-ビット形式にて利用可能。

| モデル               | 形式   | HuggingFace リポ                                                                          |
|---------------------|----------|-------------------------------------------------------------------------------------------|
| Bonsai-27B          | GGUF     | [prism-ml/Bonsai-27B-gguf](https://huggingface.co/prism-ml/Bonsai-27B-gguf)             |
| Bonsai-27B          | MLX      | [prism-ml/Bonsai-27B-mlx-1bit](https://huggingface.co/prism-ml/Bonsai-27B-mlx-1bit)     |
| Bonsai-8B           | GGUF     | [prism-ml/Bonsai-8B-gguf](https://huggingface.co/prism-ml/Bonsai-8B-gguf)               |
| Bonsai-8B           | MLX      | [prism-ml/Bonsai-8B-mlx-1bit](https://huggingface.co/prism-ml/Bonsai-8B-mlx-1bit)       |
| Bonsai-4B           | GGUF     | [prism-ml/Bonsai-4B-gguf](https://huggingface.co/prism-ml/Bonsai-4B-gguf)               |
| Bonsai-4B           | MLX      | [prism-ml/Bonsai-4B-mlx-1bit](https://huggingface.co/prism-ml/Bonsai-4B-mlx-1bit)       |
| Bonsai-1.7B         | GGUF     | [prism-ml/Bonsai-1.7B-gguf](https://huggingface.co/prism-ml/Bonsai-1.7B-gguf)           |
| Bonsai-1.7B         | MLX      | [prism-ml/Bonsai-1.7B-mlx-1bit](https://huggingface.co/prism-ml/Bonsai-1.7B-mlx-1bit)   |

ダウンロードおよび実行時にサイズ選択をするには `BONSAI_MODEL` 
(既定値: `27B`)をセットする。

### Ternary-Bonsai

GGUF (llama.cpp) および MLX 2-ビット形式にて利用可能。


| モデル                  | 形式        | HuggingFace リポ                                                                                        |
|------------------------|---------------|---------------------------------------------------------------------------------------------------------|
| Ternary-Bonsai-27B     | GGUF          | [prism-ml/Ternary-Bonsai-27B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-27B-gguf)             |
| Ternary-Bonsai-27B     | MLX (2-ビット版)   | [prism-ml/Ternary-Bonsai-27B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-27B-mlx-2bit)     |
| Ternary-Bonsai-8B      | GGUF          | [prism-ml/Ternary-Bonsai-8B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-8B-gguf)               |
| Ternary-Bonsai-8B      | MLX (2-ビット版)   | [prism-ml/Ternary-Bonsai-8B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-8B-mlx-2bit)       |
| Ternary-Bonsai-4B      | GGUF          | [prism-ml/Ternary-Bonsai-4B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-4B-gguf)               |
| Ternary-Bonsai-4B      | MLX (2-ビット版)   | [prism-ml/Ternary-Bonsai-4B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-4B-mlx-2bit)       |
| Ternary-Bonsai-1.7B    | GGUF          | [prism-ml/Ternary-Bonsai-1.7B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-1.7B-gguf)           |
| Ternary-Bonsai-1.7B    | MLX (2-ビット版)   | [prism-ml/Ternary-Bonsai-1.7B-mlx-2bit](https://huggingface.co/prism-ml/Ternary-Bonsai-1.7B-mlx-2bit)   |

コレが既定のファミリーです。1-ビット版 Bonsai ファミリーを代わりに使うには `BONSAI_FAMILY=bonsai` をセットする。

### 環境変数

両変数ともオプション値。**どちらでも無いものをセットした時, 既定値は `Ternary-Bonsai-27B` となる:** それこそが素の `./setup.sh` がダウンロードならびに実行する中身である. それらは `setup.sh`, `setup.ps1`, `download_models.sh`,
および毎度 `run_*` / `start_*` スクリプト (Linux, macOS, と Windows)に読取られる.

| 変数        | 既定値   | 有効値                       | 目的 |
|-----------------|-----------|------------------------------------|---------|
| `BONSAI_FAMILY` | `ternary` | `ternary`, `bonsai`, `all`         | モデル・ファミリ. `ternary` = Ternary-Bonsai; `bonsai` = 1-ビット版 Bonsai. `all` は両ファミリに拡大する (セットアップ/ダウンロード限定). |
| `BONSAI_MODEL`  | `27B`    | `27B`, `8B`, `4B`, `1.7B`, `all`   | モデル・サイズ. `all` は全4サイズに拡大する (セットアップ/ダウンロード限定). |
| `BONSAI_TOKEN`  | —        | HF 読取り専用トークン                 | リポがプライベート化されてる期間中の27Bモデルだけ必要 (ローンチ時には除去される). |
| `BONSAI_SKIP_GGUF` | 未セット  | `1`                                 | GGUF ダウンロードを全体的にスキップする (macOS MLX-限定セットアップ, ディスク容量節約). llama.cppスクリプトは、その後代わりのMLX用スクリプトを指示・案内する（後述の「モデルの実行」を参照). |
| `BONSAI_SKIP_MLX`  | 未セット  | `1`                                 | MLX ダウンロードをスキップする (macOS 限定; Intel Mac類 および 非-macOS上ではMLXは自動的にスキップされる). |

`all` は `setup.sh` / `setup.ps1` / `download_models.sh` についてのみ有効 — run/server スクリプトは確固たるファミリー/サイズを必要とする。

コレらを自由に組み合わせる:

```bash
./setup.sh                                                  # Ternary-Bonsai-27B (既定値)
BONSAI_MODEL=1.7B ./setup.sh                                # Ternary-Bonsai-1.7B
BONSAI_FAMILY=bonsai ./setup.sh                             # Bonsai-27B (1-ビット版)
BONSAI_FAMILY=bonsai BONSAI_MODEL=4B ./setup.sh             # Bonsai-4B
BONSAI_MODEL=all ./setup.sh                                 # Ternary-Bonsaiファミリ 全４サイズ
BONSAI_FAMILY=all BONSAI_MODEL=all ./setup.sh               # マトリックス図全部 (計8 ダウンロードを行う)
BONSAI_FAMILY=bonsai BONSAI_SKIP_GGUF=1 ./setup.sh          # Bonsai-27B, MLX 限定化 (macOS向け, ディスク容量節約)
```

## アップストリームの現況、Binary版向け

Q1_0 は多岐に渡るバックエンド: CPU (ジェネリック, NEON, および最適化済x86), Metal, CUDA, そして Vulkanを [llama.cpp](https://github.com/ggml-org/llama.cpp) アップストリームにてOOTBがサポートされる.

| ランタイム | 現況 |
|---------|--------|
| llama.cpp (CPU, Metal, CUDA, Vulkan) | ✅ アップストリームにマージ済み, 導入後即座に稼働可能 |
| MLX (1-ビット版) | ⏳ アップストリームで保留中: [mlx#3161](https://github.com/ml-explore/mlx/pull/3161); それがマージされるまでは, [PrismML-Eng/mlx](https://github.com/PrismML-Eng/mlx) (branch `prism`, `setup.sh`によって自動的にビルドされる)を使え |

## アップストリームの現況、Ternary版向け

Ternary版サポートは[llama.cpp](https://github.com/ggml-org/llama.cpp)メインラインへの移行段階中期にあり: バックエンドを1つずつ陸揚げしている最中であり、したがって今日のそれはメインラインと我々のフォークの混合物です。実際の影響をまずは: **我々は現在3つのternary版 GGUF 派生種を出荷し, そして各々を適宜適切な場所で稼働させる必要性があります。**

| ファイル | 形式 | 稼働環境 |
|------|--------|---------|
| `*-Q2_0.gguf` | グループ・サイズは128. **この形式は本デモが使用し**, 我々のフォークと互換性がある. ひとたびllama.cppの移行が完了した暁には, これらのファイルは廃止予定となり、そして `PQ2_0` ggufs によってリプレイスされる | このデモ / およびフォーク・バイナリ. メインラインでは読み込まれない (同一種別IDだが, ブロック・サイズが異なる) |
| `*-Q2_0_g64.gguf` | グループ・サイズは64 (2.25 ビット/加重). コレは公式 llama.cpp 形式であり; これらは現在のものに代わり、単に「`Q2_0`」へと改名される。 | llama.cppメインライン (これまでのCPUとMetal) |
| `*-PQ2_0.gguf` | まだサポートされてない. 今後のフォーク形式として計画された: アップストリームの `Q2_0` と共存できるよう、単にそれ自身独自の ggml 種別IDのもと現在のグループ-128 `Q2_0`と(中身が)同じ形式である。 | まだ何も無し (フォークのサポートが計画済) |

バックエンド別-バックエンド移行状況:

| バックエンド | 現況 | 引用箇所 |
|---------|--------|-------|
| CPU (ARM NEON + 汎用スカラー) | ✅ llama.cppメインラインにマージ済 | [ggml-org/llama.cpp#24448](https://github.com/ggml-org/llama.cpp/pull/24448) |
| Metal | ✅ llama.cppメインラインにマージ済 | [ggml-org/llama.cpp#25419](https://github.com/ggml-org/llama.cpp/pull/25419) |
| Vulkan | ✅ llama.cppメインラインにマージ済 | [ggml-org/llama.cpp#25430](https://github.com/ggml-org/llama.cpp/pull/25430) |
| CUDA | 🔄 アップストリームにてレビュー中 | [ggml-org/llama.cpp#25707](https://github.com/ggml-org/llama.cpp/pull/25707) |
| x86 (AVX-512-VNNI) | ⏳ 保留中 | TBD(詳細未定・確認中・後日発表予定) |

**CPU, Metal, そして Vulkanについては今llama.cppメインライン上で `Q2_0` を走らせられ、フォーク版を必要としない** (`*-Q2_0_g64.gguf` ファイルには `ggml-org/llama.cpp` の最新版ビルドを使え). CUDAはアップストリーム・レビューでラスイチ ([#25707](https://github.com/ggml-org/llama.cpp/pull/25707)) であり; マージされるまで, このデモ版を使え: コレにはフォーク版の[ビルド済みバイナリ](https://github.com/PrismML-Eng/llama.cpp/releases/tag/prism-b9596-9fcaed7)が同梱され, それ故ダウンロードされたグループ-128な `*-Q2_0.gguf` ファイルを使えば、全てが何の設定もなしにそのまま稼働する. MLX 2-ビット版は吊るしの[MLX](https://github.com/ml-explore/mlx)でサポートされており、フォーク版は不要。

より小さい ternary モデルを直に吊るしの `ggml-org/llama.cpp` (CPU もしくは Metal)で走らすには, グループ-64なファイルを使え:

| モデル | リポ | ファイル (メインライン-互換) |
|-------|------|----------------------------|
| 1.7B | [prism-ml/Ternary-Bonsai-1.7B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-1.7B-gguf) | `Ternary-Bonsai-1.7B-Q2_0_g64.gguf` |
| 4B | [prism-ml/Ternary-Bonsai-4B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-4B-gguf) | `Ternary-Bonsai-4B-Q2_0_g64.gguf` |
| 8B | [prism-ml/Ternary-Bonsai-8B-gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-8B-gguf) | `Ternary-Bonsai-8B-Q2_0_g64.gguf` |

```bash
hf download prism-ml/Ternary-Bonsai-1.7B-gguf Ternary-Bonsai-1.7B-Q2_0_g64.gguf --local-dir models
hf download prism-ml/Ternary-Bonsai-4B-gguf  Ternary-Bonsai-4B-Q2_0_g64.gguf  --local-dir models
hf download prism-ml/Ternary-Bonsai-8B-gguf  Ternary-Bonsai-8B-Q2_0_g64.gguf  --local-dir models
```

## `setup.sh` は何をしてるのか

このセットアップ・スクリプトは貴方のために全てを捌く、例え新品のマシンであったとしても:

1. **システム依存関係をチェック/インストール:** macOSではXcode CLT, Linuxではbuild-essential
2. **[uv](https://docs.astral.sh/uv/)をインストール:** 高速Pythonパッケージ・マネージャ (グローバルじゃなく、ローカル・ユーザに限局化してインストする)
3. **Python venvの作成** 及び `uv sync` の実行 —  `pyproject.toml`からcmake, ninja, huggingface-cliのインストール
4. HuggingFace (リポがプライベート期間中の27Bに至っては `BONSAI_TOKEN` が必要)から**モデルのダウンロード**
5. [GitHub Release](https://github.com/PrismML-Eng/llama.cpp/releases/tag/prism-b9596-9fcaed7)から**ビルド済バイナリのダウンロード** (かもしくは、そっちが優先したけりゃソースからビルドする)
6. **ソースからMLXをビルド** (macOS 限定): 我々のフォークをクローンし, ビルドしたそれをvenvへ放り込んで, MLスタック(mlx-lm, torch, transformers)をインストール 
7. エージェント型デモ用に venv へ**Open WebUIをインストール**  (`BONSAI_OPENWEBUI=0`でスキップ)
8. **コード-インタープリタを venv にビルド** (`.venv-jupyter`): Open WebUI コード インタープリタ用にJupyter + matplotlib / pandas / numpy / scipy / sympy / yfinance (`BONSAI_CODE_INTERPRETER=0`でスキップ)

 `setup.sh` の再実行は安全 — すでに完了した段階については飛ばす.

---

## モデルの実行

全ての実行スクリプトはBONSAI_MODEL (既定値 `27B`)を尊重する. 異なるサイズを所望ならコレをセットせよ:

### llama.cpp (Mac / Linux — 自動-検出 プラットフォーム)

```bash
./scripts/run_llama.sh -p "フランスの首都は何？"

# 異なるモデルサイズの実行
BONSAI_MODEL=4B ./scripts/run_llama.sh -p "盆栽樹についての俳句を一句ちょうだい"
```

これらのスクリプトは llama.cpp パックエンドの実行と GGUF 加重を必要とする. MLX-限定セットアップでは (e.g. あなたが `BONSAI_SKIP_GGUF=1`を使ったとして), それらは両オプションを指示するエラーで停止する — すなわち MLX スクリプトの直接実行 (`run_mlx.sh` / `start_mlx_server.sh`) ならびに GGUF 加重のダウンロードである.

### llama.cpp (Windows PowerShell)

```powershell
.\scripts\run_llama.ps1 -p "フランスの首都は何？"

# 異なるモデルサイズの実行
$env:BONSAI_MODEL = "4B"
.\scripts\run_llama.ps1 -p "盆栽樹についての俳句を一句ちょうだい"
```

### MLX — Mac (Apple Silicon)

```bash
source .venv/bin/activate
./scripts/run_mlx.sh -p "フランスの首都は何？"
```

**テスト中バージョン (再現性).** リリースされた MLX 加重は素のセーフテンソルであり、かつランタイム・パッチの必要はない. この1-ビット版パックは1-ビット量子化対応を果たした MLX ビルドを要し: コレは [PrismML-Eng/mlx](https://github.com/PrismML-Eng/mlx) フォーク(`prism`ブランチ)の [mlx#3161](https://github.com/ml-explore/mlx/pull/3161) がアップストリームにマージするまでである. 2-ビット ternary版パックは吊るしのMLXで走る. リリースされた27Bパックは以下の条件で検証された:

- Python 3.11
- mlxからフォークした`prism`ブランチのコミット時点[`88c9c20`](https://github.com/PrismML-Eng/mlx/commit/88c9c205a50f)
- `mlx-lm==0.31.2` (このバージョンは `setup.sh` がピン留めする)

`setup.sh` はブランチのヒントから(最新版)フォークをビルドする. 特定の検証済みランタイムを代わりにピン留め(固定化)するには, セットアップ実行前に当該コミットのクローンとチェックアウトをせよ; なおセットアップは既存の `./mlx` チェックアウトを再利用する:

```bash
git clone -b prism https://github.com/PrismML-Eng/mlx.git mlx
git -C mlx checkout 88c9c20
./setup.sh
```

### チャット・サーバ

その内蔵チャットUIで llama-server を起動するには:

```bash
./scripts/start_llama_server.sh    # http://localhost:8080

# 別のモデルサイズでサービスさせるには
BONSAI_MODEL=4B ./scripts/start_llama_server.sh
```

Windows PowerShell 向け:

```powershell
.\scripts\start_llama_server.ps1
```

このスクリプトは、あなたのGPU (Metal, CUDA, ROCm, Vulkan) を自動検出し、かつ全レイヤーをオフロードする。もし検出器が選び抜いたGPUが気に食わないのなら (例えば弱っちい統合型GPUしか持ち得ないマシン上のVulkan等) CPU限定推論, あるいは任意レイヤー数の部分的オフロードのために `BONSAI_NGL=0` をセットせよ (PowerShellなら: `$env:BONSAI_NGL = "0"`).

#### Thinking

27Bは thinking モデルであり、初期から thinking を**有効化**した状態でサービスする. チャットUI内の会話毎にそれを調整するには (再起動する事なしに): メッセージ・ボックス内の電球アイコンをクリックし、**Reasoningの努力レベル**を以下から選び出す: Off, Low (512 トークン), Medium (2,048〃), High (8,192〃), もしくは Max (無制限). その選択はブラウザ毎に保持され、以降全リクエストにつき送信される.

より低速なハードウェアでは, thinking は通常、待機が大半であり; この場合はUIでより低努力なレベルを選び出す. reasoning 努力レベルを指定しない API クライアントについては, 起動スクリプトを通じてllama-serverフラグを直にバイパス＆パススルーする事でサーバ-ワイドな既定値にキャップをかけられる:

```bash
./scripts/start_llama_server.sh --reasoning-budget 2048
```

#### Tool calling & MCP

The 27B does native OpenAI-style tool calling over the API, and the chat UI has an MCP client with Hugging Face + DeepWiki preconfigured (per-chat opt-in from the MCP selector in the message box, no prompt cost until you turn one on). Details, costs, and how to add your own servers: [TOOLS.md](TOOLS.md).

#### Vision

Upload images in the chat UI (`+` in the message box) or send `image_url` parts over the API; the scripts load the vision projector automatically and downscale very large images on slower backends. Costs, the image-token cap, and OCR tips: [VISION.md](VISION.md).

#### Optional extras

Two experimental, off-by-default features for the llama.cpp chat server:

- **Speculative decoding**: `BONSAI_SPECULATIVE=1` pairs the 27B with its dspark drafter for roughly 1.8-2x faster decode on code and reasoning (CUDA; Apple Silicon support will be improved later). Trade-offs and verification: [SPECULATIVE.md](SPECULATIVE.md).
- **4-bit KV cache**: `BONSAI_KV4=1` cuts KV-cache memory roughly 3.5x for very long contexts, with an optional calibration bias for better quality (`./scripts/make_kv_bias.sh`). Details: [KV-CACHE.md](KV-CACHE.md).
- **Vision projector in RAM**: `BONSAI_MMPROJ_CPU=1` keeps the 27B's vision projector in system RAM instead of VRAM (`--no-mmproj-offload`), freeing ~0.9 GiB of VRAM for KV/context on tight cards. The cost is a slower image prompt (the projector runs on CPU); text-only chat is unaffected.

### Context Size

The 27B models support up to **262,144 tokens** of context. The FP16 KV cache costs 64 KiB per token (~6.3 GiB at 100K), so **100K context fits on many consumer devices even without KV-cache quantization**. The model's hybrid attention keeps the cache small for its size.

The launch scripts pick a **default context sized to your machine's RAM**, from 8K on small machines up to 131K for the 27B on machines with more than 71 GB (roughly 0.5 to 8 GiB of KV cache), so memory use stays predictable. Override with the `BONSAI_CTX` environment variable: pass any number up to 262144, or `0` (the same as leaving it unset) for the automatic RAM-tiered size. To force the model's full training context, pass the explicit number (e.g. `BONSAI_CTX=262144`) — only recommended on machines with plenty of headroom, since the scripts will not silently do this for you.

With the optional [4-bit KV cache](KV-CACHE.md) (`BONSAI_KV4=1`) the cache drops to roughly 18 KiB per token, about **1.8 GiB at 100K**, shaving ~4.5 GiB off the 100K figures below (for example, Ternary-Bonsai-27B on llama.cpp goes from ~13.7 to ~9.2 GiB).

*Peak memory for the 27B (weights + activations + FP16 KV cache + ~1.2 GiB overhead; text-only, add ~0.9 GiB for the vision projector):*

| Model | Format | Weights | 4K context | 10K context | 100K context |
|---|---|---|---|---|---|
| Bonsai-27B (1-bit) | llama.cpp `Q1_0` | 3.53 GiB | 4.8 GiB | 5.2 GiB | 10.8 GiB |
| Bonsai-27B (1-bit) | MLX 1-bit | 3.92 GiB | 5.5 GiB | 5.9 GiB | 11.4 GiB |
| Ternary-Bonsai-27B | llama.cpp `Q2_0` | 6.66 GiB | 7.8 GiB | 8.1 GiB | 13.7 GiB |
| Ternary-Bonsai-27B | MLX 2-bit | 7.05 GiB | 8.6 GiB | 8.9 GiB | 14.4 GiB |
| *reference: 27B 16-bit* | GGUF BF16 | 47.73 GiB | 49 GiB | 49.6 GiB | 55.2 GiB |
| *reference: 27B "4-bit"* | llama.cpp `UD Q4_K_M` | 15.73 GiB | 17.2 GiB | 17.6 GiB | 23.2 GiB |
| *reference: 27B "4-bit"* | MLX 4-bit | 13.3 GiB | 17.0 GiB | 17.3 GiB | 22 GiB |

(The MLX packs are ~400 MiB larger than GGUF because MLX stores both scales and biases, GGUF only scales.)

Extra arguments pass straight through to llama.cpp, so `./scripts/run_llama.sh -c 8192 -p "Your prompt"` also works for a one-off context override.

The older text-only sizes are smaller across the board; the 8B supports up to 65,536 tokens of context:

*Estimates for Bonsai-8B (weights + KV cache + activations):*

| Context Size        | Est. Memory Usage |
|---------------------|-------------------|
| 8,192 tokens        | ~2.5 GB           |
| 32,768 tokens       | ~5.9 GB           |
| 65,536 tokens       | ~10.5 GB          |

---

## Open WebUI (Optional): the full agentic demo

[Open WebUI](https://github.com/open-webui/open-webui) gives you a ChatGPT-like interface on top of the local 27B: chat with images, tool calling against live tools, a server-side code interpreter (plots + market data), and a hidden-story sales database to investigate. Everything is configured automatically, no clicking through settings:

```bash
./scripts/start_openwebui.sh
```

`setup.sh` installs it for you; the script starts the backend, seeds the demo (tools, model settings, demo database), and opens **http://localhost:9090**. Backends, what to try, and customizing: [OPENWEBUI.md](OPENWEBUI.md).

---

## Building from Source

If you prefer to build llama.cpp from source instead of using pre-built binaries:

### Mac (Apple Silicon — Metal)

```bash
./scripts/build_mac.sh
```

Clones [PrismML-Eng/llama.cpp](https://github.com/PrismML-Eng/llama.cpp), builds with Metal, outputs to `bin/mac/`.

### Mac (Intel — CPU only)

```bash
./scripts/build_mac.sh
```

The script auto-detects Intel vs Apple Silicon. On Intel Macs, it builds with `-DGGML_METAL=OFF` (CPU only). MLX is also skipped automatically since it requires Apple Silicon.

### Linux (CPU only)

```bash
./scripts/build_cpu_linux.sh
```

Builds a CPU-only binary with no GPU dependencies. Works on both x64 and arm64. Outputs to `bin/cpu/`.

### Linux (CUDA)

```bash
./scripts/build_cuda_linux.sh
```

Auto-detects CUDA version. Pass `--cuda-path /usr/local/cuda-12.8` to use a specific toolkit.

### Linux (Vulkan)

```bash
# Install Vulkan SDK first (e.g. sudo apt install libvulkan-dev glslc)
git clone -b prism https://github.com/PrismML-Eng/llama.cpp.git
cd llama.cpp
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_VULKAN=ON
cmake --build build -j$(nproc)
# Binaries in build/bin/
```

### Linux (ROCm / AMD GPU)

```bash
# Requires ROCm toolkit (hipcc)
git clone -b prism https://github.com/PrismML-Eng/llama.cpp.git
cd llama.cpp
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_HIP=ON
cmake --build build -j$(nproc)
# Binaries in build/bin/
```

### Windows (CUDA)

```powershell
.\scripts\build_cuda_windows.ps1
```

Auto-detects CUDA toolkit. Pass `-CudaPath "C:\path\to\cuda"` to use a specific version.
Requires Visual Studio Build Tools (or full Visual Studio) and CUDA toolkit.

### Windows (CPU only)

```powershell
git clone -b prism https://github.com/PrismML-Eng/llama.cpp.git
cd llama.cpp
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
# Binaries in build\bin\Release\
```

Requires Visual Studio Build Tools or full Visual Studio with C++ workload.

---

## llama.cpp Pre-built Binary Downloads

All binaries are available from the [GitHub Release](https://github.com/PrismML-Eng/llama.cpp/releases/tag/prism-b9596-9fcaed7):

| Platform                          |
|-----------------------------------|
| macOS Apple Silicon (arm64)       |
| macOS Apple Silicon (KleidiAI)    |
| macOS Intel (x64)                 |
| Linux x64 (CPU)                   |
| Linux arm64 (CPU)                 |
| Linux x64 (CUDA 12.4)            |
| Linux x64 (CUDA 12.8)            |
| Linux x64 (Vulkan)               |
| Linux arm64 (Vulkan)             |
| Linux x64 (ROCm 7.2)             |
| Windows x64 (CPU)                |
| Windows arm64 (CPU)              |
| Windows x64 (CUDA 12.4)          |
| Windows x64 (Vulkan)             |
| Windows x64 (HIP/ROCm)           |
| iOS (XCFramework)                 |

---

## Folder Structure

After setup, the directory looks like this:

```
Bonsai-demo/
├── README.md
├── TOOLS.md                        # Tool calling & MCP guide
├── OPENWEBUI.md                    # Open WebUI agentic demo guide
├── VISION.md                       # Image input: costs, caps, OCR tips
├── SPECULATIVE.md                  # Speculative decoding (experimental)
├── KV-CACHE.md                     # 4-bit KV cache (experimental)
├── AGENTS.md                       # Agent guide (hardware tuning knobs)
├── setup.sh                        # macOS/Linux setup
├── setup.ps1                       # Windows setup
├── pyproject.toml                  # Python dependencies
├── scripts/
│   ├── common.sh                   # Shared helpers + BONSAI_MODEL
│   ├── download_models.sh          # HuggingFace download
│   ├── download_binaries.sh        # GitHub release download
│   ├── run_llama.sh                # llama.cpp (auto-detects Mac/Linux)
│   ├── run_llama.ps1               # llama.cpp (Windows PowerShell)
│   ├── run_mlx.sh                  # MLX inference
│   ├── mlx_generate.py             # MLX Python script
│   ├── start_llama_server.sh       # llama.cpp server (port 8080)
│   ├── start_llama_server.ps1      # llama.cpp server (Windows PowerShell)
│   ├── start_mlx_server.sh         # MLX server (port 8081)
│   ├── start_openwebui.sh          # Open WebUI + auto-starts backends
│   ├── openwebui/                  # Open WebUI demo tools + seeding
│   ├── build_mac.sh                # Build llama.cpp for Mac
│   ├── build_cpu_linux.sh          # Build llama.cpp for Linux (CPU only)
│   ├── build_cuda_linux.sh         # Build llama.cpp for Linux CUDA
│   └── build_cuda_windows.ps1      # Build llama.cpp for Windows CUDA
├── models/                         # ← downloaded by setup
│   ├── gguf/
│   │   ├── 27B/                    # GGUF 27B model (+ mmproj for vision)
│   │   ├── 8B/                     # GGUF 8B model
│   │   ├── 4B/                     # GGUF 4B model
│   │   └── 1.7B/                   # GGUF 1.7B model
│   ├── Bonsai-27B-mlx/            # MLX 27B model (macOS)
│   ├── Bonsai-8B-mlx/             # MLX 8B model (macOS)
│   ├── Bonsai-4B-mlx/             # MLX 4B model (macOS)
│   └── Bonsai-1.7B-mlx/           # MLX 1.7B model (macOS)
├── bin/                            # ← downloaded or built by setup
│   ├── mac/                        # macOS binaries (Metal or CPU)
│   ├── cuda/                       # CUDA binaries (Linux/Windows)
│   ├── cpu/                        # CPU-only binaries (Linux/Windows)
│   ├── vulkan/                     # Vulkan binaries
│   ├── rocm/                       # ROCm binaries (AMD Linux)
│   └── hip/                        # HIP binaries (AMD Windows)
├── mlx/                            # ← cloned by setup (macOS)
└── .venv/                          # ← created by setup
```

Items marked with ← are created at setup time and excluded from git.

---

## Appendix — FAQ

### The model allocates huge memory or the machine freezes at startup

Older revisions defaulted to llama.cpp's `-c 0`, which uses the model's full training context (262K on the 27B) regardless of available memory and could exhaust it on constrained machines. The scripts now always use a RAM-tiered context instead; `BONSAI_CTX=0` maps to that same safe default rather than `-c 0`. If you still hit memory pressure, pin a smaller context:

```bash
BONSAI_CTX=8192 ./scripts/start_llama_server.sh
```

### M5 Mac on macOS 26.2/26.3: Metal compile errors, then out-of-memory

On M5 devices with certain macOS 26 point releases, the Metal tensor-API probe fails to compile at runtime (`ggml_metal_library_init_from_source: error compiling source`) and can leave the GPU in a bad state. This is an ecosystem-wide issue in the OS Metal headers, hitting every ggml-based project. Workaround, keeps full Metal speed and just skips the tensor-API path:

```bash
GGML_METAL_TENSOR_DISABLE=1 ./scripts/run_llama.sh -p "Hello"
```


### CUDA source build runs out of memory or freezes

**Symptom:** `cmake --build` hangs, the system becomes unresponsive, or the build process is killed with an OOM error when building llama.cpp from source with CUDA enabled.

**Cause:** Compiling CUDA kernels is memory-intensive — each parallel compile job can consume several GB of GPU VRAM and/or system RAM. Running `make -j$(nproc)` on a machine with a low-VRAM GPU (< 16 GB) or limited system RAM can exhaust available memory.

**How the build scripts handle this:** `build_cuda_linux.sh` and `build_cuda_windows.ps1` automatically detect the GPU's VRAM before building. If the maximum detected VRAM is less than 16 GB, the scripts cap parallelism at `-j 2` instead of using all logical CPU cores. You will see a message like:

```
Detected GPU VRAM: 8.0 GB (< 16 GB) -- limiting CUDA build to -j 2
```

**Manual override:** If you still encounter OOM errors, reduce parallelism further by editing the build invocation in the relevant script, or close other GPU-heavy applications before building.

### Metal fails to initialize on Apple M5 (macOS 26.2–26.4)

**Symptom:** On M5 Macs, `run_llama.sh` / `start_llama_server.sh` fail with Metal errors and produce no output, e.g.:

```
ggml_metal_library_init_from_source: error compiling source
ggml_metal_device_init: - the tensor API is not supported in this environment - disabling
ggml_metal_synchronize: error: command buffer 0 failed with status 5
```

Pre-M5 Apple Silicon (M1–M4) is not affected — those devices load the embedded, precompiled Metal library and never compile shaders at runtime.

**Cause:** On M5 (and A19) devices, ggml compiles its Metal library from source at runtime to enable the tensor API (Neural Accelerators). Some macOS 26 point releases ship stricter MetalPerformancePrimitives headers whose `static_assert` (bfloat/half type mismatch) breaks that runtime compile. This is an ecosystem-wide issue also seen in [ollama](https://github.com/ollama/ollama/issues/15594) and [whisper.cpp](https://github.com/ggml-org/whisper.cpp/issues/3722); see [#93](https://github.com/PrismML-Eng/Bonsai-demo/issues/93).

**Workaround:** Disable the tensor API so the M5 uses the embedded library like pre-M5 devices — full Metal speed is kept, only the Neural Accelerator prefill boost is lost:

```bash
GGML_METAL_TENSOR_DISABLE=1 ./scripts/run_llama.sh -p "Hello"
GGML_METAL_TENSOR_DISABLE=1 ./scripts/start_llama_server.sh
```

This is much faster than falling back to CPU (`BONSAI_NGL=0`). If out-of-memory errors persist afterwards on lower-memory machines, additionally pin a smaller context, e.g. `-c 16384` (extra args pass through to llama.cpp and override the default).
