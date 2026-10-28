# Goose 言語の VS Code 拡張

このディレクトリには公式 Goose コンパイラの言語サーバーを使う既存拡張が含まれます。`.goose` の色分け、補完、定義への移動、診断、JIT 実行などが利用できます。詳しい機能と制約は [英語版 README](README.md) を参照してください。

## インストール

VS Code 1.85 以降で、コマンドパレットから **Extensions: Install from VSIX...** を選び、このリポジトリの `vscode/goose-language.vsix` を指定します。コマンドラインに `code` がある場合は、`goose/` ディレクトリから次でもインストールできます。

```sh
code --install-extension vscode/goose-language.vsix
```

拡張のインストールには Node.js や Python は不要ですが、診断や実行には Goose コンパイラが別途必要です。VSIX はコンパイラを含みません。

## コンパイラと設定

[公式リリース](https://github.com/aardappel/goose/releases)のコンパイラを使うか、`goose/` で CMake 3.27 以降、C++20 コンパイラ、Python 3 を用いてビルドします。

```sh
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

コンパイラは通常 `build/goose`（Visual Studio ジェネレーターでは `build/Release/goose.exe`）です。JIT には TinyCC 対応ビルドが必要です。標準ライブラリとビルド条件の詳細は[本体 README](../README.md#build-and-run)を参照してください。

VS Code で `goose/` をフォルダーとして開けば、拡張は `build/goose` などを自動検出し、見つからなければ PATH の `goose` を使います。別の場所のコンパイラを使う場合は、ワークスペース設定に実行ファイルのパスを指定します。例えば今回のローカルビルドが `build/llvm/goose` の場合:

```json
{
  "goose.compilerPath": "${workspaceFolder}/build/llvm/goose",
  "goose.entryFile": "samples/01_tour.goose"
}
```

`goose.entryFile` は複数ファイルのプログラムの入口です。単一ファイルを編集中は省略でき、その場合は開いているファイルを使います。他のフォルダーを開いた場合、`${workspaceFolder}` はそのフォルダーを指すため、コンパイラの絶対パスを指定するか PATH に追加してください。必要なら `goose.stdlibPath` で標準ライブラリを指定できます。

`.goose` ファイルを開き、コマンドパレットの **Goose: Check Program** で確認、**Goose: Run with JIT** で実行できます。対応コンパイラなら編集中の診断や補完に `goose lsp` を使用し、非対応なら保存時の `goose check` にフォールバックします。実行やコンパイラ診断にはワークスペースの信頼が必要です。デバッグのブレークポイントには対応していません。

## 開発と配布

`goose/` から以下を実行すると、ロック済み npm 依存関係をインストールし、テスト・バンドル後に `vscode/goose-language.vsix` を再生成します。このファイルは Git 管理されているため、再生成すると既存ファイルを上書きします。

```sh
python3 vscode/build_vsix.py
```

Python 3 と Node.js/npm（Node.js 22 以降推奨）が必要です。拡張の開発時は `vscode/` を VS Code で開き F5 を押すと Extension Development Host を起動できます。テストだけなら `cd vscode && npm ci && npm test`。自動検出されないコンパイラを使う場合は `GOOSE_TEST_COMPILER` にその実行ファイルの絶対パスを設定してください。コンパイラがないと実コンパイラ連携テストはスキップされます。エディター統合テストの手順は[英語版 README](README.md#verification)にあります。
