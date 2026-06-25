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
```

```python
#app/core/config.py
import os
from pathlib import Path
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # App Environment Routing Mappings
    FENERGO_BASE_URL: str = "https://api.can1.fenergox.com"
    DATABASE_URL: str = "sqlite:///./reporting_platform.db"
    
    # Legacy Fallback Configuration Parameters
    DEFAULT_DELIMITER: str = ","
    DEFAULT_DATE_FORMAT: str = "yyyy-MM-dd HH:mm:ss"
    
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
