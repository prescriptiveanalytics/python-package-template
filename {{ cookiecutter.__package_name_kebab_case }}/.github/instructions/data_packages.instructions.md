---
applyTo: '**/*'
---

## Data Package Development Guidelines

## Package Structure
- *Must:* Create dedicated data package classes for each data source/operator
- *Do:* Organize packages by domain (operator, area, data type)
- *Do:* Use consistent naming: `{Source}DataPackage` (e.g., `OperatorDataPackage`)
- *Do:* Provide path management and validation in package classes
- *Do:* Include metadata files (`version.json`, `meta.json`) for reproducibility

```python
class OperatorDataPackage:
    """Data package for operator-specific datasets."""
    
    def __init__(
        self, 
        base_path: SupportedPath | str = LocalPaths.data_dir,
        operator: str = "default"
    ) -> None:
        self.base_path = PathHandler.process_base_path(base_path)
        self.package_dir = self.base_path / operator
        
        # define all expected files
        self.preprocessed = self.package_dir / "preprocessed.parquet"
        self.meta = self.package_dir / "meta.parquet"
        self.version = self.package_dir / "version.json"
```

## Reproducibility & Generation
- *Must:* Make data packages deterministically regeneratable via `poe` command
- *Do:* Version all generated data packages with timestamp and hash
- *Do:* Include generation metadata (parameters, versions, seeds)
- *Do:* Provide clear regeneration commands in package documentation
- *Must:* Store generation parameters in `meta_info.json`

```python
# in pyproject.toml
[tool.poe.tasks]
regenerate_data_package = "python -m data_package.generate --operator=linznetz --area=friensdorf"
regenerate_all = ["clean_data", "regenerate_data_package", "validate_data"]

# generation metadata
{
    "generated_at": "2024-10-28T10:30:00Z",
    "generator_version": "1.2.3",
    "parameters": {
        "operator": "linznetz",
        "area": "friensdorf",
        "seed": 42
    },
    "data_hash": "sha256:abc123..."
}
```

## Path Management
- *Do:* Use `PathHandler` for cross-platform path processing
- *Do:* Support both local and cloud storage paths via `SupportedPath`
- *Do:* Centralize path definitions in data package classes
- *Must:* Validate file existence with meaningful error messages
- *Do:* Provide `ensure_dirs()` method for directory creation

```python
class PathHandler:
    @staticmethod
    def process_base_path(path: SupportedPath | str) -> Path:
        """Process and validate base path for data packages."""
        if isinstance(path, str):
            return Path(path)
        return path
    
    @staticmethod
    def ensure_paths_exist(package: object) -> None:
        """Ensure all package directories exist."""
        for attr_name in dir(package):
            attr = getattr(package, attr_name)
            if isinstance(attr, Path) and not attr.name.endswith(('.parquet', '.csv', '.json')):
                attr.mkdir(parents=True, exist_ok=True)
```

## Type Definitions
- *Must:* Define type aliases for path handling and data structures
- *Do:* Use union types for flexible path support

```python
from pathlib import Path
from cloudpathlib import CloudPath

# type alias for supported path types
SupportedPath = Path | CloudPath | str
PackageDataDF = pd.DataFrame
```

## Data Validation & Schema
- *Must:* Define schemas for all data package formats
- *Must:* Access DataFrame columns via schema attributes, never hardcoded strings
- *Do:* Use Pandera DataFrameModel for tabular data validation
- *Do:* Include schema validation in package loading methods
- *Do:* Provide clear error messages for schema violations
- *Must:* Document expected data structure and validation rules
- *Avoid:* Hardcoded column names like `df['column_name']` or `df.column_name`

```python
class PackageDataSchema(pa.DataFrameModel):
    """Schema for package time series data.
    
    Validates structure and ensures data quality constraints.
    """
    timestamp: Series[pd.Timestamp] = pa.Field(nullable=False)
    target_value: Series[float] = pa.Field(ge=0, nullable=False) 
    metadata_id: Series[str] = pa.Field(nullable=False)
    
    class Config:
        strict = True
        coerce = True

def load_validated_data(self, file_path: Path) -> PackageDataDF:
    """Load and validate data against schema."""
    try:
        data = pd.read_parquet(file_path)
        return PackageDataSchema.validate(data)
    except pa.errors.SchemaError as e:
        raise DataValidationError(f"Schema validation failed for {file_path}: {e}")
```

## Preprocessing & Imputation
- *Do:* Centralize preprocessing logic in dedicated modules
- *Do:* Make imputation strategies configurable via constants
- *Do:* Track imputation metadata separately from original data
- *Must:* Generate before/after availability reports
- *Do:* Provide visualization of preprocessing effects

```python
# preprocessing constants
IMPUTATION_STRATEGIES = {
    'temperature': 'linear_interpolation',
    'load_values': 'forward_fill',
    'categorical': 'mode_fill'
}

def apply_imputation(
    data: PackageDataDF, 
    strategies: dict[str, str] = IMPUTATION_STRATEGIES
) -> tuple[PackageDataDF, pd.DataFrame]:
    """Apply imputation using schema-based column access."""
    validated_data = PackageDataSchema.validate(data)
    imputed_data = validated_data.copy()
    imputation_mask = pd.DataFrame(False, index=data.index, columns=data.columns)
    
    # use schema attributes instead of hardcoded column names
    temp_col = PackageDataSchema.temperature
    if hasattr(PackageDataSchema, 'temperature'):
        mask = imputed_data[temp_col].isna()
        imputed_data[temp_col] = apply_strategy(imputed_data[temp_col], 'linear_interpolation')
        imputation_mask[temp_col] = mask
    
    return imputed_data, imputation_mask
```

## Visualization & Documentation  
- *Must:* Generate automated documentation for each data package
- *Do:* Create availability charts (before/after preprocessing)
- *Do:* Generate data visualizations in PDF format for archival
- *Do:* Include README files with package usage examples
- *Do:* Provide data profiling and quality metrics

```python
def generate_package_documentation(self) -> None:
    """Generate comprehensive package documentation."""
    # availability analysis
    self._generate_availability_charts()
    
    # data visualization
    with PdfPages(self.data_visualization) as pdf:
        for column in self.get_numeric_columns():
            fig = self._plot_time_series(column)
            pdf.savefig(fig, bbox_inches="tight")
    
    # readme generation
    self._generate_readme_html()
    
    # version tracking
    self._update_version_file()
```

## Environment & Configuration
- *Do:* Support configurable data directories via environment variables
- *Do:* Provide sensible defaults for development environments
- *Must:* Handle both development and production path configurations
- *Do:* Use resource files for package-relative paths

```python
class LocalPaths:
    """Centralized path configuration for data packages."""
    _package_root = resources.files("data_package")
    _development_root = _package_root.parent.parent
    data_dir = os.getenv("DATA_DIR", _development_root.parent / "data")
    
    @classmethod
    def for_environment(cls, env: str = "development") -> Path:
        """Get data directory for specific environment."""
        if env == "production":
            return Path(os.getenv("PROD_DATA_DIR", "/data"))
        return cls.data_dir
```

## Testing & Quality Assurance
- *Must:* Test data package generation and validation
- *Do:* Create fixtures for test data packages
- *Do:* Test path resolution across platforms
- *Must:* Validate schema compliance in tests
- *Do:* Test regeneration determinism

```python
@pytest.fixture
def test_data_package(tmp_path):
    """Create test data package structure."""
    package = OperatorDataPackage(tmp_path, operator="test")
    package.ensure_dirs()
    
    # create minimal test data
    test_data = pd.DataFrame({
        'timestamp': pd.date_range('2024-01-01', periods=100, freq='15min'),
        'value': range(100)
    })
    test_data.to_parquet(package.preprocessed)
    return package

def test_package_regeneration_determinism(test_data_package):
    """Test that package regeneration produces identical results."""
    # generate twice with same parameters
    result1 = generate_package(seed=42, **params)
    result2 = generate_package(seed=42, **params) 
    
    assert result1.data_hash == result2.data_hash
```

## Logging & Monitoring
- *Must:* Log all data package operations to CSV file
- *Do:* Include timestamps, operation type, file paths, and outcomes
- *Do:* Use structured logging format for analysis
- *Must:* Create log.csv in package directory for tracking operations

```python
import logging
import csv
from datetime import datetime

class DataPackageLogger:
    def __init__(self, package_dir: Path):
        self.log_file = package_dir / "log.csv"
        self.ensure_log_headers()
    
    def ensure_log_headers(self) -> None:
        """Ensure CSV log file has proper headers."""
        if not self.log_file.exists():
            with open(self.log_file, 'w', newline='') as f:
                writer = csv.writer(f)
                writer.writerow(['timestamp', 'operation', 'file_path', 'status', 'message'])
    
    def log_operation(self, operation: str, file_path: Path, status: str, message: str = "") -> None:
        """Log data package operation to CSV file."""
        with open(self.log_file, 'a', newline='') as f:
            writer = csv.writer(f)
            writer.writerow([datetime.now().isoformat(), operation, str(file_path), status, message])
```

## Error Handling
- *Must:* Provide clear error messages with resolution guidance
- *Do:* Handle missing files gracefully with informative errors
- *Avoid:* Silent failures in data validation
- *Do:* Fail fast and early
- *Must:* Log all errors to CSV file with context

```python
class DataPackageError(Exception):
    """Base exception for data package operations."""
    pass

class DataValidationError(DataPackageError):
    """Raised when data fails schema validation."""
    pass

class PackageNotFoundError(DataPackageError):
    """Raised when expected data package files are missing."""
    
    def __init__(self, missing_files: list[Path]):
        files_str = ", ".join(str(f) for f in missing_files)
        super().__init__(
            f"Missing required data package files: {files_str}. "
        )
```