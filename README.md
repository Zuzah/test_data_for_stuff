# test_data_for_stuff
tbd

```bash
[project]
name = "gco-fenx-reporting-platform"
version = "0.1.0"
description = "Shared execution platform integrating FastAPI and Airflow with Fen-X Advanced Reporting APIs"
dependencies = [
    "fastapi>=0.110.0",
    "uvicorn>=0.28.0",
    "pydantic-settings>=2.2.1",
    "loguru>=0.7.2",
    "httpx>=0.27.0",
    "pandas>=2.2.1",
    "sqlalchemy>=2.0.28"
]
```

```bash
FENERGO_BASE_URL=https://api.can1.fenergox.com
DATABASE_URL=sqlite:///./reporting_platform.db
DEFAULT_DELIMITER=,
DEFAULT_DATE_FORMAT=M/d/yyyy h:mm:ss tt

# Bank Authorization Layer Assignments (Safe here, local only)
FENERGO_CLIENT_ID=tbd
FENERGO_CLIENT_SECRET=tbd
FENERGO_TOKEN_URL=tbd
FENERGO_SCOPE=12569.fenergo.api.read
```

```python
import os
from pathlib import Path
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # App Environment Routing Mappings
    FENERGO_BASE_URL: str = "https://api.can1.fenergox.com"
    DATABASE_URL: str = "sqlite:///./reporting_platform.db"
    
    # --- Bank Enterprise Authentication Mapping (No Defaults - Safely Injected) ---
    FENERGO_CLIENT_ID: str
    FENERGO_CLIENT_SECRET: str
    FENERGO_TOKEN_URL: str
    FENERGO_SCOPE: str
    
    # Legacy Fallback Configuration Parameters
    DEFAULT_DELIMITER: str = ","
    DEFAULT_DATE_FORMAT: str = "M/d/yyyy h:mm:ss tt"
    
    # Cross-Platform Path Tracking Helpers
    BASE_DIR: Path = Path(__file__).resolve().parent.parent.parent
    
    @property
    def template_path(self) -> Path:
        """Mimics legacy .NET BaseDirectory + Constants.TemplatePath lookup"""
        return self.BASE_DIR / "config" / "ExtractTemplates"

    @property
    def sql_query_path(self) -> Path:
        """Mimics legacy .NET BaseDirectory + Constants.SqlQueryPath lookup"""
        return self.BASE_DIR / "config" / "SQLQueries"

    # Read seamlessly from local system environment or uncommitted .env files
    model_config = SettingsConfigDict(
        env_file=os.path.join(Path(__file__).resolve().parent.parent.parent, ".env"),
        env_file_encoding="utf-8",
        extra="ignore"
    )

# Instantiate a single, globally reusable settings object across all services
settings = Settings()
```

More
```python
# test_config.py
from app.core.config import settings

def test_environment_loading():
    print("==================================================")
    print("🛡️ RUNNING INITIAL PARITY CONFIGURATION VALIDATION")
    print("==================================================")
    print(f"Project Base Directory: {settings.BASE_DIR}")
    print(f"Target Fenergo URL:     {settings.FENERGO_BASE_URL}")
    print(f"Database Mapping State: {settings.DATABASE_URL}")
    print(f"Default CSV Delimiter:  '{settings.DEFAULT_DELIMITER}'")
    print(f"Default Date Pattern:   {settings.DEFAULT_DATE_FORMAT}")
    print(f"Legacy Template Path:   {settings.template_path}")
    print(f"Legacy SQL Query Path:  {settings.sql_query_path}")
    print("==================================================")
    print("✅ INITIAL WORKSPACE ENVIRONMENT SETUP: NO BUGS")
    print("==================================================")

if __name__ == "__main__":
    test_environment_loading()
```

app/core/logging.py

'''python
import sys
from pathlib import Path
from loguru import logger
from app.core.config import settings

def configure_logging():
    """
    Initializes a production-grade multi-target logging structure.
    Sinks informational tracks to Console (stdout) and system anomalies to an error log file.
    """
    # 1. Clear any default standard library logger configurations
    logger.remove()

    # 2. Add standard Console stdout handler for operational info tracking
    logger.add(
        sys.stdout,
        level="INFO",
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level:7}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>",
        colorize=True
    )

    # 3. Securely determine localized logs subdirectory within system base path
    log_dir = settings.BASE_DIR / "logs"
    log_dir.mkdir(parents=True, exist_ok=True)
    error_log_path = log_dir / "error.log"

    # 4. Bind file-system sink tracking exceptions and errors exclusively
    logger.add(
        str(error_log_path),
        level="WARNING",
        format="{time:YYYY-MM-DD HH:mm:ss} | {level:7} | {name}:{function}:{line} - {message}",
        rotation="10 MB",      # Prevents files from growing infinitely
        retention="30 days",   # Automatically purges stale server records
        compression="zip"      # Compresses historic tracking sets to optimize disk space
    )

    return logger

# Globally instantiate configured multi-sink execution logger
log = configure_logging()

'''

test_logging.py

```python
# test_logging.py
from pathlib import Path
from app.core.logging import log
from app.core.config import settings

def test_logger_routing():
    print("==================================================")
    print("📝 RUNNING STRUCTURAL LOGGING VALIDATION")
    print("==================================================")
    
    # 1. Fire informational trace (Should ONLY print to console, NOT file)
    log.info("System initializing... Connecting to Fenergo configuration layer.")
    
    # 2. Fire warning error traces (Should print to console AND write to logs/error.log)
    log.warning("System Alert: High network latency detected on endpoint handshake.")
    log.error("Parsing Exception: Core token identifier parsing failure mock error.")

    # 3. Verify file generation
    expected_log_file = settings.BASE_DIR / "logs" / "error.log"
    
    print("\n--------------------------------------------------")
    print("🔎 RUNNING LOG ROTATION FILE EXTRACT VERIFICATION:")
    if expected_log_file.exists():
        print(f"✅ PASS: File system log discovered at {expected_log_file}")
        with open(expected_log_file, "r") as f:
            lines = f.readlines()
            print(f"📁 Log File Record Count: {len(lines)} rows captured.")
    else:
        print("❌ FAIL: Log output file could not be generated on local drive.")
    print("==================================================")

if __name__ == "__main__":
    test_logger_routing()
```
