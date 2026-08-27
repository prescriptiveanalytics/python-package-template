---
applyTo: '*.py'
---

## Docstrings
- *Do:* Use Google style with Args, Returns, Raises sections

## Formatting & Imports
- *Do:* Run `poe precommit` to format, sort imports, and lint
- *Do:* Run `poe check` to verify changes
- *Do:* Keep lines under 120 characters for readability

## Comments & Naming
- *Must:* Start comments with lowercase letter (except proper nouns/variables)
- *Do:* Include meaningful comments explaining "why" not "what"
- *Don't:* Use code section markers (`# ---`, `# ===`)
- *Must:* Use snake_case for functions and variables
- *Must:* Use PascalCase for classes

```python
# correct: lowercase comment start
def process_user_data(user_id: str) -> dict:  # snake_case function
    """Process user data from database."""
    current_user = get_user(user_id)  # snake_case variable
    return current_user

class DataProcessor:  # PascalCase class
    """Handles data processing operations."""
    pass
```

## Type Hints
- *Must:* Always annotate function parameters and returns
- *Do:* Use Python 3.12+ syntax: `| None`, `|` for unions, `list[int]`, `dict[str, int]`
- *Avoid:* Long inline type hints; create short aliases instead
- *Tolerate:* `Any` type only for untyped third-party libraries
- *Do:* Type class attributes and module-level variables

```python
from typing import Protocol

class Config:  # PascalCase
    timeout_ms: int = 5000  # snake_case attribute
    max_retries: int = 3

def process_items(items: list[str]) -> dict[str, int]:  # snake_case
    pass

class DataHandler(Protocol):  # PascalCase
    def load_data(self, file_path: str) -> dict: ...  # snake_case
```

## DataFrames & Schema Validation
- *Must:* Use Pandera DataFrameModel for all DataFrame schemas
- *Do:* Create type alias `TypeNameDF = pd.DataFrame` 
- *Must:* Access columns via schema attributes, never hardcoded strings
- *Do:* Use Field constraints for validation

```python
from pandera.pandas import DataFrameModel, Field
from pandera.typing import Series

class ItemSchema(DataFrameModel):
    """Schema with validation constraints."""
    item_id: Series[str] = Field(nullable=False)
    value: Series[float] = Field(ge=0, nullable=False)
    class Config:
        strict = True
        coerce = True

ItemDF = pd.DataFrame

def filter_items(data: ItemDF, min_val: float) -> ItemDF:
    return data[data[ItemSchema.value] >= min_val].copy()
```

## Method Chaining
- *Do:* Use parentheses for multi-line pandas/polars operations
- *Do:* Place each method on a new line

```python
result = (
    df.read_parquet('file.parquet')
    .map_partitions(transform)
    .to_parquet('out.parquet')
)
```

## Constants
- *Do:* Use UPPERCASE for constant names
- *Do:* Define constants in centralized `constants.py`

```python
MAX_RETRIES = 3
DEFAULT_TIMEOUT_MS = 5000
```

## Interfaces
- *Prefer:* `Protocol` (from `typing`) over abstract classes

## SOLID Principles
- **S - Single Responsibility**: Each class should have one reason to change
- **O - Open/Closed**: Classes open for extension, closed for modification
- **L - Liskov Substitution**: Derived classes must be substitutable for base classes
- **I - Interface Segregation**: Many specific interfaces better than one general interface
- **D - Dependency Inversion**: Depend on abstractions, not concretions

```python
# Single Responsibility - each class has one job
class UserValidator:
    def validate_email(self, email: str) -> bool:
        return "@" in email and "." in email

class UserRepository:
    def save_user(self, user: dict) -> None:
        # database logic only
        pass

# Interface Segregation - specific protocols
class Readable(Protocol):
    def read(self) -> str: ...

class Writable(Protocol):
    def write(self, data: str) -> None: ...

# Dependency Inversion - depend on protocols
class DataProcessor:
    def __init__(self, reader: Readable, writer: Writable) -> None:
        self.reader = reader
        self.writer = writer
    
    def process(self) -> None:
        data = self.reader.read()
        processed = data.upper()
        self.writer.write(processed)
```

## Error Handling
- *Do:* Raise meaningful exceptions with descriptive messages
- *Do:* Create custom exceptions for domain-specific errors
- *Avoid:* Catching and ignoring exceptions silently

```python
class ValidationError(Exception):
    """Raised when data validation fails."""
    pass

def validate_data(data: dict) -> None:
    if not data:
        raise ValidationError("data cannot be empty")
```

## Security
- *Must:* Never include credentials in code; use environment variables
- *Must:* Never log sensitive data (passwords, tokens, PII)
- *Must:* Never commit `.env` files; use `.env.example` for documentation
- *Must:* Validate all external inputs; use parameterized queries

```python
# Use environment variables
DATABASE_URL = os.getenv("DATABASE_URL")

# Validate inputs and use parameterized queries
def get_user(cursor, user_id: str) -> dict:
    if not user_id.isalnum():
        raise ValueError("Invalid user ID")
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    return cursor.fetchone()
```

## Logging
- *Do:* Create logger with `logger = logging.getLogger(__name__)`
- *Do:* Add at module level, use throughout the module
- *Avoid:* Printing to stdout (use logging instead)

```python
import logging

logger = logging.getLogger(__name__)

def process_item(item: dict) -> None:
    logger.info(f"processing item: {item['id']}")
    try:
        result = transform(item)
        logger.debug(f"transformation result: {result}")
    except Exception as e:
        logger.error(f"failed to process: {e}", exc_info=True)
```

## Paths
- *Do:* Use `Path` from `pathlib` for cross-platform compatibility
- *Do:* Use unix-style paths relative to project root
- *Do:* Centralize path definitions in a `Paths` class

```python
from pathlib import Path

class Paths:
    root = Path(__file__).parent.parent
    data = root / 'data'
```

## Project Structure
- *Do:* Organize code into: `typedef.py` (types), `constants.py` (magic strings), implementation
- *Do:* Use `notebooks/` directory for `.py` files with `# %%` magic strings
- *Avoid:* `.ipynb` format (use `.py` for version control friendliness)
- *Do:* Keep specific types/constants in innermost modules
- *Do:* Centralize shared types/constants above implementation
- *Do:* Separate concerns by module to avoid tight coupling
- *Avoid:* Creation of markdown files for documentation, summaries, analyses, explanations, or changelogs unless explicitly requested by the user
- *Don't:* Create .md files to document code changes, summarize work, or explain implementations

## Service Layer Architecture
- *Do:* Separate data access logic into dedicated service classes
- *Do:* Use dependency injection for service composition
- *Do:* Create clear boundaries between services, state management, and presentation layers
- *Prefer:* Composition over inheritance for service dependencies

```python
class EvaluationService:
    def __init__(self, data_path: Path) -> None:
        self.data_path = data_path
    
    def get_metrics(self, baseline: pd.DataFrame) -> pd.DataFrame:
        # data access logic here
        pass

class PageState:
    def __init__(self, evaluation_service: EvaluationService) -> None:
        self.evaluation_service = evaluation_service
```

## Testing
- *Do:* Run tests with `poe test` (uses `pytest`)
- *Do:* Keep tests short and focused
- *Do:* Use doctests for simple examples in docstrings
- *Do:* Move long test logic to `tests/` folder
- *Do:* Use fixtures for reusable test data
- *Do:* Name test files `test_*.py`, test functions `test_*`

```python
"""Module docstring with examples.

Example:
    >>> process([1, 2, 3])
    [2, 4, 6]
"""

import pytest

TEST_ID = "model-123"

@pytest.fixture
def sample_data():
    """Create sample test data."""
    return {"id": TEST_ID, "value": 42}

def test_process_basic(sample_data):
    """Test basic processing."""
    result = process(sample_data)
    assert result["id"] == TEST_ID
```

## Configuration Management
- *Do:* Centralize configuration in dedicated classes
- *Do:* Use environment variables for runtime configuration
- *Do:* Provide sensible defaults for development
- *Avoid:* Hardcoded paths and magic values scattered throughout code

```python
class Config:
    data_dir: Path = os.getenv("DATA_DIR", Path.cwd() / "data")
    timeout_seconds: int = int(os.getenv("TIMEOUT", "30"))
    
class Paths:
    root = Path(__file__).parent.parent
    data = root / "data"
    models = data / "models"
```

## Dependencies
- *Must:* Use `uv` (Astral) for dependency management
- *Do:* Define dependencies in `pyproject.toml`, install with `uv sync`

## Environment & Docker
- *Must:* Pin versions in production; document setup in README
- *Must:* Separate runtime/dev dependencies in multi-stage Dockerfile
- *Do:* Align Dockerfile with `.devcontainer.json`; use `.env.example`