# Mainnet Deploy 手順書

## 前提条件
以下がインストールされていること：
- Solana CLI (`solana --version`)
- Anchor CLI (`anchor --version`)
- Rust (`rustc --version`)

インストールされていない場合：
```bash
# Solana CLI
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"

# Anchor CLI
cargo install --git https://github.com/coral-xyz/anchor avm --locked
avm install latest && avm use latest
```

---

## Step 1: mainnet ウォレット確認

```bash
solana config set --url mainnet-beta
solana address          # 現在のウォレットアドレス確認
solana balance          # 残高確認（最低3 SOL必要）
```

残高が足りない場合、そのアドレスにSOLを送金してください。

---

## Step 2: プログラムのキーペアを生成

```bash
cd ~/Desktop/TimeLockSol/timelock-sol
solana-keygen new --outfile target/deploy/timelock_sol-keypair.json --no-bip39-passphrase
solana address -k target/deploy/timelock_sol-keypair.json
```

表示されたアドレス（例: `ABC123...`）をコピーしておく。

---

## Step 3: declare_id! を更新

`programs/timelock_sol/src/lib.rs` の4行目を更新：

```rust
declare_id!("ここにStep2で取得したアドレスを貼る");
```

`Anchor.toml` の `[programs.mainnet]` も更新：
```toml
[programs.mainnet]
timelock_sol = "ここにStep2で取得したアドレスを貼る"
```

---

## Step 4: ビルド

```bash
anchor build
```

完了まで数分かかります。

---

## Step 5: mainnet にデプロイ

```bash
anchor deploy --provider.cluster mainnet
```

デプロイコスト: プログラムサイズに応じて **1〜3 SOL** 程度。

---

## Step 6: index.html の PROGRAM_ID を更新

`~/Desktop/TimeLockSol/index.html` の CONFIG を更新：

```javascript
PROGRAM_ID: 'Step2で取得したアドレス',
```

---

## Step 7: initialize を実行（config を mainnet に作成）

ブラウザでサイトを開き、ウォレット接続後に初めてロックしようとすると
自動的に initialize と init_user_state が実行されます。

---

## Step 8: GitHub に push → Netlify 自動デプロイ

```bash
cd ~/Desktop/TimeLockSol
git add index.html
git commit -m "Deploy to mainnet: update PROGRAM_ID"
git push
```

Netlify が自動でデプロイして完了です。
