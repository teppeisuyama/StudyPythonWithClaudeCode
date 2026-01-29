# CLAUDE.md - ClaudeCode プロジェクト指示書

このファイルはClaudeCodeがプロジェクトを理解し、一貫した開発を行うための指示書です。

## プロジェクト概要

- **目的**: PythonとClaudeCodeを学習するためのリポジトリ
- **言語**: Python 3.12+
- **パッケージ管理**: uv（推奨）または pip

## ディレクトリ構造

```
.
├── CLAUDE.md           # このファイル（ClaudeCode用指示）
├── pyproject.toml      # プロジェクト設定・依存関係
├── src/                # ソースコード
│   └── study_python/   # メインパッケージ
├── tests/              # テストコード
├── docs/               # ドキュメント（必要に応じて）
└── scripts/            # ユーティリティスクリプト
```

## コーディング規約

### Python スタイルガイド

1. **PEP 8 準拠**: ruffによる自動フォーマット・リント
2. **型ヒント必須**: すべての関数に型アノテーションを付与
3. **docstring**: Google スタイルを使用

```python
def example_function(param1: str, param2: int) -> bool:
    """関数の説明を簡潔に記述する。

    Args:
        param1: 最初のパラメータの説明。
        param2: 2番目のパラメータの説明。

    Returns:
        戻り値の説明。

    Raises:
        ValueError: エラーが発生する条件の説明。
    """
    pass
```

### 命名規則

- **変数・関数**: snake_case（例: `user_name`, `calculate_total`）
- **クラス**: PascalCase（例: `UserAccount`, `DataProcessor`）
- **定数**: UPPER_SNAKE_CASE（例: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`）
- **プライベート**: 先頭にアンダースコア（例: `_internal_method`）

### インポート順序

1. 標準ライブラリ
2. サードパーティライブラリ
3. ローカルモジュール

```python
import os
import sys
from pathlib import Path

import requests
from pydantic import BaseModel

from study_python.core import utils
```

## 開発コマンド

```bash
# 依存関係のインストール
uv sync

# リンター実行
uv run ruff check .

# フォーマッター実行
uv run ruff format .

# 型チェック
uv run mypy src/

# テスト実行
uv run pytest

# テスト（カバレッジ付き）
uv run pytest --cov=src/study_python --cov-report=html
```

## コミットメッセージ規約

[Conventional Commits](https://www.conventionalcommits.org/) に従う：

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメントのみの変更
- `style`: コードの意味に影響しない変更（空白、フォーマット等）
- `refactor`: バグ修正でも機能追加でもないコード変更
- `test`: テストの追加・修正
- `chore`: ビルドプロセスやツールの変更

### 例

```
feat(auth): ユーザー認証機能を追加

- JWTトークンによる認証を実装
- ログイン/ログアウトAPIを追加

Closes #123
```

## テスト方針

1. **ユニットテスト**: 各関数・メソッドに対してテストを作成
2. **命名規則**: `test_<関数名>_<テスト条件>`
3. **配置**: `tests/` ディレクトリに `test_*.py` として配置
4. **Fixtures**: 共通のセットアップは `conftest.py` に定義

```python
# tests/test_calculator.py
import pytest
from study_python.calculator import add

def test_add_positive_numbers():
    assert add(1, 2) == 3

def test_add_negative_numbers():
    assert add(-1, -2) == -3

def test_add_with_zero():
    assert add(0, 5) == 5
```

## エラーハンドリング

1. **具体的な例外**: 汎用的な `Exception` ではなく具体的な例外を使用
2. **カスタム例外**: 必要に応じてドメイン固有の例外を定義
3. **ログ出力**: エラー時は適切なログレベルで記録

```python
import logging

logger = logging.getLogger(__name__)

class ValidationError(Exception):
    """入力検証エラー"""
    pass

def process_data(data: dict) -> None:
    try:
        validate(data)
    except ValidationError as e:
        logger.error(f"Validation failed: {e}")
        raise
```

## セキュリティ注意事項

- 機密情報（APIキー、パスワード等）はコードにハードコードしない
- 環境変数または `.env` ファイル（.gitignoreに含める）を使用
- ユーザー入力は必ずバリデーション・サニタイズを行う

## ClaudeCode 使用時の注意

1. **変更前に確認**: ファイルを編集する前に必ず内容を確認する
2. **小さな変更**: 大きな変更は小さなステップに分割する
3. **テスト実行**: コード変更後は必ずテストを実行する
4. **既存コードの尊重**: プロジェクトの既存パターンに従う
