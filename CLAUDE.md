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
├── scripts/            # ユーティリティスクリプト
└── logs/               # ログファイル（.gitignore対象）
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

## カバレッジ方針（重要）

コードの品質を保証するため、テストカバレッジ率100%を目指す。

### 目標

- **カバレッジ率**: 100%（除外対象を除く）
- **必須**: 新規コードには必ずテストを作成する
- **CI/CD**: カバレッジが基準を下回る場合はマージをブロック

### カバレッジ実行コマンド

```bash
# カバレッジ付きテスト実行
uv run pytest --cov=src/study_python --cov-report=html --cov-report=term-missing

# カバレッジレポートの確認
# htmlcov/index.html をブラウザで開く
```

### カバレッジ除外の方法

通常の処理では到達しないコードは `# pragma: no cover` コメントで除外する。

```python
def process_data(data: dict) -> str:
    if not data:
        raise ValueError("Data cannot be empty")  # テストで検証

    try:
        result = transform(data)
        return result
    except TransformError as e:
        logger.error(f"Transform failed: {e}")
        raise
    except Exception as e:  # pragma: no cover
        # 予期しないエラー（通常到達しない）
        logger.critical(f"Unexpected error: {e}")
        raise
```

### 除外してよいケース

以下のケースのみ `# pragma: no cover` の使用を許可する：

| ケース | 理由 | 例 |
|--------|------|-----|
| 予期しない例外ハンドラ | 防御的コードで通常到達しない | `except Exception:` |
| デバッグ専用コード | 本番では実行されない | `if DEBUG:` ブロック |
| 抽象メソッド | 実装クラスでテストする | `raise NotImplementedError` |
| 型チェック専用ブロック | 実行時には不要 | `if TYPE_CHECKING:` |
| プラットフォーム固有コード | 実行環境で到達不可 | `if sys.platform == 'win32':` |
| main実行ブロック | モジュールとして使用時は不要 | `if __name__ == '__main__':` |

### 除外してはいけないケース

以下は必ずテストでカバーすること：

- **バリデーションエラー**: 入力検証の例外は意図的に発生させてテスト
- **ビジネスロジックの分岐**: すべての条件分岐をテスト
- **エラーハンドリング**: 想定されるエラーケースはテスト
- **境界値**: 最小値、最大値、空のケースなど

### 実装例

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:  # pragma: no cover
    from typing import Optional

def divide(a: int, b: int) -> float:
    """2つの数値を除算する。

    Args:
        a: 被除数。
        b: 除数。

    Returns:
        除算結果。

    Raises:
        ValueError: 除数が0の場合。
    """
    if b == 0:
        raise ValueError("Cannot divide by zero")  # テストで検証する
    return a / b


def safe_divide(a: int, b: int) -> float | None:
    """安全に除算を行う。"""
    try:
        return divide(a, b)
    except ValueError:
        logger.warning(f"Division by zero attempted: {a}/{b}")
        return None
    except Exception as e:  # pragma: no cover
        # 予期しないエラー（通常到達しない防御的コード）
        logger.critical(f"Unexpected error in division: {e}")
        raise


if __name__ == "__main__":  # pragma: no cover
    # スクリプトとして直接実行時のみ
    result = divide(10, 2)
    print(f"Result: {result}")
```

### テストでのカバレッジ確保

```python
# tests/test_divide.py
import pytest
from study_python.math_utils import divide, safe_divide


def test_divide_normal():
    """正常な除算のテスト。"""
    assert divide(10, 2) == 5.0


def test_divide_by_zero():
    """ゼロ除算エラーのテスト。"""
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)


def test_safe_divide_normal():
    """safe_divideの正常系テスト。"""
    assert safe_divide(10, 2) == 5.0


def test_safe_divide_by_zero():
    """safe_divideのゼロ除算テスト。"""
    result = safe_divide(10, 0)
    assert result is None
```

### pyproject.toml でのカバレッジ設定

```toml
[tool.coverage.run]
source = ["src/study_python"]
branch = true
omit = [
    "*/__pycache__/*",
    "*/tests/*",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "if __name__ == .__main__.:",
    "raise NotImplementedError",
    "@abstractmethod",
]
fail_under = 90
show_missing = true
skip_covered = false

[tool.coverage.html]
directory = "htmlcov"
```

### 注意事項

- `# pragma: no cover` の乱用は禁止（レビューで確認）
- カバレッジ率が下がる変更はPRで理由を説明すること
- 新規ファイルは100%カバレッジを目指す
- レガシーコードは段階的にカバレッジを向上させる

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

## ログ出力ルール（重要）

すべてのツール・モジュールには、ユーザーからの問い合わせ対応（特に不具合調査）のためにログ出力機能を実装すること。

### 必須事項

1. **ログ出力の実装**: すべてのモジュールで `logging` を使用する
2. **ファイル出力**: ログは外部ファイル（`logs/` ディレクトリ）に保存する
3. **適切なログレベル**: 状況に応じたログレベルを使用する

### ログレベルの使い分け

| レベル | 用途 |
|--------|------|
| `DEBUG` | 開発時のデバッグ情報（変数値、処理フロー等） |
| `INFO` | 正常な処理の記録（開始、終了、重要なステップ） |
| `WARNING` | 注意が必要だが処理は継続可能な状況 |
| `ERROR` | エラー発生（処理失敗、例外キャッチ等） |
| `CRITICAL` | システム停止レベルの重大エラー |

### ログ設定の実装

プロジェクト共通のログ設定モジュールを使用する：

```python
# src/study_python/logging_config.py を使用
from study_python.logging_config import setup_logging

# モジュール初期化時にログ設定を適用
setup_logging()

# 各モジュールでロガーを取得
import logging
logger = logging.getLogger(__name__)
```

### ログフォーマット

デバッグしやすいフォーマットを使用：

```
2024-01-15 10:30:45.123 | INFO     | module_name:function_name:42 | メッセージ
```

フォーマット要素：
- **タイムスタンプ**: ミリ秒まで記録
- **ログレベル**: 見やすく固定幅
- **モジュール名**: 問題箇所の特定用
- **関数名・行番号**: デバッグ時の追跡用
- **メッセージ**: 具体的な内容

### 実装例

```python
import logging
from study_python.logging_config import setup_logging

# ログ設定を適用
setup_logging()
logger = logging.getLogger(__name__)

def process_user_data(user_id: int, data: dict) -> bool:
    """ユーザーデータを処理する。"""
    logger.info(f"Processing data for user_id={user_id}")
    logger.debug(f"Input data: {data}")

    try:
        # 処理実行
        result = validate_and_save(data)
        logger.info(f"Successfully processed user_id={user_id}")
        return result
    except ValidationError as e:
        logger.error(f"Validation failed for user_id={user_id}: {e}")
        raise
    except Exception as e:
        logger.critical(f"Unexpected error for user_id={user_id}: {e}", exc_info=True)
        raise
```

### ログファイル管理

- **保存先**: `logs/` ディレクトリ（.gitignoreに含める）
- **ファイル名**: `app_YYYY-MM-DD.log`（日付ローテーション）
- **保持期間**: 30日間（古いログは自動削除）
- **最大サイズ**: 10MB（サイズローテーション）

### 注意事項

- 機密情報（パスワード、トークン等）はログに出力しない
- 大量データはログに出力せず、件数やサマリーを記録する
- 本番環境では `INFO` 以上、開発環境では `DEBUG` 以上を出力する

## ClaudeCode 使用時の注意

1. **変更前に確認**: ファイルを編集する前に必ず内容を確認する
2. **小さな変更**: 大きな変更は小さなステップに分割する
3. **テスト実行**: コード変更後は必ずテストを実行する
4. **既存コードの尊重**: プロジェクトの既存パターンに従う
