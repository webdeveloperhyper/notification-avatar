# Notification Avatar

([日本語の説明は下部をご覧ください / Japanese version below](#notification-avatar日本語))

Notification Avatar is a free desktop companion app — your VRoid (VRM) avatar pops up in the corner of your screen and reacts whenever **Claude Code or Codex** needs your attention or finishes a task, with a speech bubble and a cute animation. Works even when VS Code isn't open, since it reacts to Claude Code/Codex CLI hooks directly. 🥰

> Notification Avatar is a spin-off of **[AI Avatar](https://github.com/webdeveloperhyper/ai-avatar)**. AI Avatar is a free app where your VRoid (VRM) avatar cheers you with all its might. Lives in your VS Code sidebar or browser side panel.

**Jump to:** [✨ Features](#features) · [📖 Details](#getting-started) · [🗺️ Roadmap](#roadmap)

🛒 **[Download from GitHub Releases](https://github.com/webdeveloperhyper/notification-avatar/releases)**

---

## Features

- 🔔 **Reacts to Claude Code or Codex hooks** — pops up on approval-needed and task-finished events from either tool, with a different speech-bubble message for each
- 💃 **Cute animation** — nod, smile, and a random cute pose on every appearance
- 🚦 **Color-coded speech bubble** — red for `Stop` (task finished), yellow for `Notification` (needs your approval), so you can tell which one fired at a glance
- ⏱️ **Auto fade** — fades out a few seconds after appearing
- 🔁 **Change Avatar** — swap the VRM character anytime from the system tray
- 🌐 **Language toggle** — switch speech-bubble messages between EN and JP from the tray menu
- 🚀 **Launch at startup** — toggle auto-start at login from the tray menu

---

## Getting Started

### 1. Install the app
1. Download **`Notification-Avatar-win.zip`** from [GitHub Releases](https://github.com/webdeveloperhyper/notification-avatar/releases) *(not "Source code")*
2. Extract all and double-click `Notification Avatar.exe` — no installation needed
3. If Windows SmartScreen warns you, click **More info** → **Run anyway** (the app is safe but not yet code-signed)
4. The app starts in the system tray — you won't see a window until an avatar reaction pops up

### 2. Set up the hooks
Both Claude Code and Codex work — pick the section for your tool and setup below. Each just writes a small trigger file the app watches; requires Node.js to be installed. If Claude Code or Codex itself runs inside WSL2, use the WSL2 variant (`node.exe` instead of `node`) — otherwise the hook would write to WSL2's own home directory instead of the Windows one this app watches. The WSL2 variant needs **Node.js for Windows** specifically (not just WSL2's own Node) installed and on the Windows `PATH`, so `node.exe` resolves via WSL2's interop.

#### Claude Code (native Windows)
Add this to your `~/.claude/settings.json` (or a project's `.claude/settings.json`):

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))\""
          }
        ]
      }
    ]
  }
}
```

#### Claude Code (WSL2)
Same file, same JSON — just call `node.exe` instead of `node` so it's the Windows-side Node.js writing to the Windows-side trigger file (not WSL2's own `~/.claude`):

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))\""
          }
        ]
      }
    ]
  }
}
```

#### Codex (native Windows)
Codex needs two separate mechanisms for the same coverage — `notify` fires when a response finishes (maps to `Stop`), `hooks.PermissionRequest` fires on an approval prompt (maps to `Notification`). Add this to your **user-level** `~/.codex/config.toml`:

```toml
notify = ["node", "-e", "require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))"]

[[hooks.PermissionRequest]]
matcher = ""

[[hooks.PermissionRequest.hooks]]
type = "command"
command = "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
```

Restart Codex for the config change to take effect — `config.toml` is only read at startup. If you already have a `config.toml`, put the `notify = [...]` line at the very top of the file — putting it below other settings can make it silently stop working.

#### Codex (WSL2)
Same as above, with `node.exe` instead of `node`:

```toml
notify = ["node.exe", "-e", "require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))"]

[[hooks.PermissionRequest]]
matcher = ""

[[hooks.PermissionRequest.hooks]]
type = "command"
command = "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
```

Restart Codex for the config change to take effect, and keep `notify = [...]` at the very top of the file, same as above.

---

That's it — the app watches `~/.claude/notification-avatar-trigger.json` and pops up whenever any of the hooks above writes to it.

> This is the same Claude Code/Codex hook mechanism used by our fully open-source **[claude-code-codex-notification](https://github.com/webdeveloperhyper/claude-code-codex-notification)** — a tiny, dependency-free script version with no avatar. Check that repo if you want to see exactly how hook-triggered notifications work, with nothing hidden.

---

## Change Avatar

Right-click the tray icon → **Change Avatar** → select any `.vrm` file. Your choice is saved automatically and restored on next launch. If you move the `.vrm` file, select it again from the tray.

You can create your own 3D avatar for free using **[VRoid](https://vroid.com/en)** — no 3D modeling skills needed. Design your character and export it as a `.vrm` file.

Or use the **Candy Pink** sample avatar — a kind and cheerful girl. Good at making fake smiles.

| v2 | v3 | v4 | v5 | v6 | v7 | v8 cat |
|---|---|---|---|---|---|---|
| <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v2.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v3.png" width="45px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v4.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v5.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v6.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v7.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v8-cat.png" width="40px"> |

👉 **[Download Candy Pink from GitHub](https://github.com/webdeveloperhyper/ai-avatar/tree/main/avatars)**

---

## Language Toggle

Right-click the tray icon → **Language: EN** to switch to JP, or **Language: JP** to switch back to EN.

Speech bubble messages will show in the selected language. The setting is saved automatically and restored on next launch.

---

## Launch at Startup

Right-click the tray icon → **Launch at Startup: ON/OFF** to toggle. When ON, Notification Avatar starts automatically when you log in to Windows.

> If you move the `.exe`, toggle **Launch at Startup** OFF then ON again to re-register the new location.

---

## Quit

Right-click the tray icon → **Quit**.

---

## Roadmap

**v1** ✅
- 🎉 Notification Avatar initial release!

**v2** — Now creating!
- 🍎 Mac support
- Other fun updates!

[↑ Back to top](#notification-avatar)

---

# Notification Avatar（日本語）

Notification Avatar は無料のデスクトップ companion アプリです。**Claude CodeまたはCodex**があなたの入力を必要としたとき、またはタスクを完了したときに、画面の隅に VRoid（VRM）アバターが現れ、吹き出しとかわいいアニメーションでお知らせしてくれます。VS Code を開いていなくても、Claude Code/Codex CLI のフックに直接反応するので動作します。🥰

> Notification Avatar は **[AI Avatar](https://github.com/webdeveloperhyper/ai-avatar)** のスピンオフ。AI AvatarはVRoid（VRM）アバターが全力で応援してくれる無料アプリ。VS Code サイドバーやブラウザのサイドパネルで動作します。

**ジャンプ:** [✨ 機能](#機能) · [📖 詳細](#はじめかた) · [🗺️ ロードマップ](#ロードマップ)

🛒 **[GitHub Releases からダウンロード](https://github.com/webdeveloperhyper/notification-avatar/releases)**

---

## 機能

- 🔔 **Claude CodeまたはCodexのフックに反応** — どちらのツールでも、承認待ち・タスク完了のイベントで出現し、それぞれ異なる吹き出しメッセージを表示
- 💃 **かわいいアニメーション** — 出現のたびにうなずき・笑顔・ランダムポーズを再生
- 🚦 **吹き出しを色分け表示** — `Stop`（タスク完了）は赤、`Notification`（承認待ち）は黄色で、ひと目でどちらか分かる
- ⏱️ **自動フェード** — 数秒後に自動でフェードアウト
- 🔁 **アバター変更** — システムトレイからいつでも VRM キャラを変更可能
- 🌐 **言語切替** — トレイメニューから吹き出しメッセージを EN/JP で切り替え
- 🚀 **自動起動** — トレイメニューからログイン時の自動起動をトグル

---

## はじめかた

### 1. アプリをインストール
1. [GitHub Releases](https://github.com/webdeveloperhyper/notification-avatar/releases) から **`Notification-Avatar-win.zip`** をダウンロード *（"Source code" ではなく）*
2. すべて展開して `Notification Avatar.exe` をダブルクリック — インストール不要
3. Windows SmartScreen の警告が出た場合は **詳細情報** → **実行** をクリック（コード署名未対応ですが安全です）
4. アプリがシステムトレイに常駐します — アバターのリアクションが出るまでウィンドウは表示されません

### 2. フックを設定
Claude CodeとCodexのどちらでも動作します — お使いのツールと環境のセクションを選んでください。どちらも、アプリが監視する小さなトリガーファイルを書き出すだけです。Node.jsがインストールされている必要があります。Claude CodeやCodex自体がWSL2内で動いている場合はWSL2版（`node` の代わりに `node.exe`）を使ってください — そうしないとフックがWSL2自身のホームディレクトリに書き込んでしまい、このアプリが監視しているWindows側のファイルに届きません。WSL2版では、`node.exe` がWSL2のinterop経由で見つかるように、**Windows版のNode.js**（WSL2自身のNodeだけでは不可）がインストールされ、WindowsのPATHに通っている必要があります。

#### Claude Code（ネイティブWindows）
`~/.claude/settings.json`（またはプロジェクトの `.claude/settings.json`）に以下を追加してください：

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))\""
          }
        ]
      }
    ]
  }
}
```

#### Claude Code（WSL2）
同じファイル、同じJSONです — `node` の代わりに `node.exe` を呼ぶだけで、Windows側のNode.jsがWindows側のトリガーファイルに書き込むようになります（WSL2自身の `~/.claude` ではなく）：

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))\""
          }
        ]
      }
    ]
  }
}
```

#### Codex（ネイティブWindows）
Codexでは同じカバレッジを得るために2つの仕組みが必要です — `notify` は応答完了時（`Stop` に相当）、`hooks.PermissionRequest` は承認プロンプト表示時（`Notification` に相当）に発火します。**ユーザーレベル**の `~/.codex/config.toml` に以下を追加してください：

```toml
notify = ["node", "-e", "require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))"]

[[hooks.PermissionRequest]]
matcher = ""

[[hooks.PermissionRequest.hooks]]
type = "command"
command = "node -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
```

設定を反映させるにはCodexを再起動してください — `config.toml` は起動時にのみ読み込まれます。すでに `config.toml` がある場合、`notify = [...]` の行はファイルの一番上に置いてください — 他の設定より下に置くと、サイレントに動かなくなることがあります。

#### Codex（WSL2）
上と同じ内容で、`node` の代わりに `node.exe` を使います：

```toml
notify = ["node.exe", "-e", "require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'stop',timestamp:Date.now()}))"]

[[hooks.PermissionRequest]]
matcher = ""

[[hooks.PermissionRequest.hooks]]
type = "command"
command = "node.exe -e \"require('fs').writeFileSync(require('path').join(require('os').homedir(),'.claude','notification-avatar-trigger.json'), JSON.stringify({event:'notification',timestamp:Date.now()}))\""
```

設定を反映させるにはCodexを再起動してください。`notify = [...]` をファイルの一番上に置くのも上記と同様です。

---

これだけです — アプリは `~/.claude/notification-avatar-trigger.json` を監視し、上記いずれかのフックが書き込むたびに出現します。

> これはClaude Code/Codexのフックの仕組みとして、完全オープンソースの **[claude-code-codex-notification](https://github.com/webdeveloperhyper/claude-code-codex-notification)** と同じものです — アバターなしの、依存関係ゼロの小さなスクリプト版です。フックによる通知がどう動いているか、隠すことなくそのまま確認したい方はそちらのリポジトリをご覧ください。

---

## アバターの変更

トレイアイコンを右クリック → **Change Avatar** → `.vrm` ファイルを選択。選択したアバターは自動保存され、次回起動時に復元されます。`.vrm` ファイルを移動した場合は、トレイから再度選択してください。

**[VRoid](https://vroid.com/ja)** を使えば3Dモデリングの知識なしに無料でオリジナルアバターを作れます。キャラクターをデザインして `.vrm` ファイルとしてエクスポートしてください。

サンプルアバター **Candy Pink** もご利用いただけます — 優しく明るい女の子。作り笑いが得意。

| v2 | v3 | v4 | v5 | v6 | v7 | v8 cat |
|---|---|---|---|---|---|---|
| <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v2.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v3.png" width="45px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v4.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v5.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v6.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v7.png" width="40px"> | <img src="https://raw.githubusercontent.com/webdeveloperhyper/ai-avatar/main/docs/images/candy-pink-v8-cat.png" width="40px"> |

👉 **[Candy Pink を GitHub からダウンロード](https://github.com/webdeveloperhyper/ai-avatar/tree/main/avatars)**

---

## 言語切替

トレイアイコンを右クリック → **Language: EN** で JP に切り替え、**Language: JP** で EN に戻します。

選択した言語で吹き出しメッセージが表示されます。設定は自動保存され、次回起動時に復元されます。

---

## 自動起動

トレイアイコンを右クリック → **Launch at Startup: ON/OFF** でトグル。ON のとき、ログイン時に Notification Avatar が自動で起動します。

> `.exe` を移動した場合は、**Launch at Startup** を一度 OFF にしてから ON に戻すと再登録されます。

---

## 終了

トレイアイコンを右クリック → **Quit**

---

## ロードマップ

**v1** ✅
- 🎉 Notification Avatar 初回リリース！

**v2** — 作成中！
- 🍎 Mac 対応
- 他の楽しいアップデート！

[↑ ページトップへ戻る](#notification-avatar)

