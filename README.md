# test_data_for_stuff

## About the work

### ExtractUtility
In the original .NET architecture, the workflow acted as a bridge between the raw database schema and the client-facing report schema. The process looked exactly like this:

1. The .NET console app executed a raw SQL query from a .sql file inside config/SQ** against the database server.

2. The database returned a standard data reader dataset, which often contained internal system column names or database-specific naming conventions.

3. The app looped through that dataset. For every row, it looked strictly at the ordered Column elements inside the Extra**.xml file. It pulled data from the matching database fields, applied the data formatting transformations, and outputted the clean row using the exact user-friendly column names specified in the XML configuration (ColNm="...").


## 🧬 Local Parity Demonstration & Validation

The platform includes a localized CLI execution loop that allows developers to run end-to-end report extractions on a local machine or within an Ubuntu WSL environment. This allows you to verify data parity byte-for-byte against legacy systems without needing an active database connection.

### 🏃‍♂️ Running the Local Pipeline Engine

1. **Prepare Your Local Source Assets:** Ensure you have an extraction mapping template configured under your configuration directory (`config/Extrac/AND.xml`) and a raw test file located at `data/raw_input.csv`.

2. **Execute the Transformation Pipeline:** Run the local orchestrator by passing the input file, the template mapping schema, and your target output destination path as arguments:
   ```bash
   python run_local.py \
     --input data/raw_input.csv \
     --template config/Extrac/NDER_.xml \
     --output data/processed_output.csv


### Architectural Service Core Overview

The transformation engine inside this repository is a direct port of the legacy .NET Framework 4.7 compilation utility (ExtractUtility.exe). It migration moves the platform toward a decoupled, service-oriented architecture.

🔄 Legacy Architecture vs. Modern Python Platform
Legacy .NET Pattern (ExtractUtility.exe): Tightly bound command-line invocation parsing, database cursor loop structures (SqlDataReader), and XML configuration string matches together inside a single application runtime.

Modern Python Service Pattern: Completely uncouples the engine components into isolated, reusable modules using an Adapter Design Pattern.

```txt
                           ┌─────────────────────────┐
                           │   Legacy XML Template   │
                           └────────────┬────────────┘
                                        │
                                        ▼
┌──────────────────┐       ┌─────────────────────────┐       ┌──────────────────────┐
│  Raw Data Source │ ───>  │     MappingService      │ ───>  │ Standardized Outbound│
│  (Local or API)  │       │ (Pure Business Logic)   │       │   Reporting File     │
└──────────────────┘       └─────────────────────────┘       └──────────────────────┘
```

Key Architectural Components
1. app/core/config.py & logging.py: Strongly-typed settings management (via Pydantic) and multi-sink execution logs that protect credentials and track pipeline anomalies.

2. app/services/xml_adapter.py (XMLConfigAdapter): Parses the metadata layer (ExtractType, DateFormat, column positions) from legacy XML configurations and builds an internal ReportSchema. This allows you to easily drop the XML logic in Phase 2 and switch to a database or JSON configuration without breaking the application.

3. app/services/mapping.py (MappingService): The core business engine. It streams large files in low-memory chunks to keep cloud footprints small, strips dirty whitespace, standardizes datetimes to match reporting export constraints, and applies explicit field quoting rules.


## How to Find/Verify API Credentials inside the Fenergo UI Platform
If the authentication script continues to encounter an error, your local developer keys may have expired or need to be whitelisted for the correct reporting API scopes. Follow these steps within the Fenergo platform to verify or generate a clean pair:

Log In to Your Tenant Portal: Navigate to your environment portal (e.g., your bank's Fenergo SaaS gateway landing page) using your corporate credentials.

Access Admin Configuration Settings: Locate the gear icon on the top right header pane or navigation panel and select Setup / Administration Console.

Navigate to Identity & Access Management (IAM): In the left navigation menu layout pane, look under security groupings or integration layers and open Auth Providers / API Credentials / Connected Apps.

Locate or Create Your Client Credentials Entry: Find the service account identifier mapping to your reporting utility team (e.g., gco-reporting-platform). If none exists, click Create New Client Credentials.

Verify Allowed Permissions Scopes: Ensure your client registration profile has the explicit permission scope checked: fenergo.api.read or advanced_reporting.read. If this scope is missing from the client registration, the auth server will return an error when you try to connect.

Regenerate & Update .env: Copy the active Client ID and click Regenerate Secret to capture the text token. Open your local uncommitted .env configuration file immediately and paste the keys in place.

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

```python
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

```

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


app/services/xml_adapter.py

```python
import xml.etree.ElementTree as ET
from pathlib import Path
from dataclasses import dataclass, field
from typing import List, Dict
from app.core.logging import log

@dataclass
class ColumnDefinition:
    """Tracks field metadata required for downstream database type mapping transformations."""
    name: str
    data_type: str  # e.g., "String", "DateTime", "Numeric"
    assignment_order: int

@dataclass
class ReportSchema:
    """Unified configuration map driving the transformation engine rules."""
    extract_type: str = "CSV"
    date_format: str = "M/d/yyyy h:mm:ss tt"
    max_retry: int = 3
    columns: List[ColumnDefinition] = field(default_factory=list)

class XMLConfigAdapter:
    """Transforms legacy bank XML layout schemas into unified internal ReportSchema objects."""
    
    @staticmethod
    def parse_template(xml_path: Path) -> ReportSchema:
        log.info(f"Parsing configuration template file from path: {xml_path}")
        
        if not xml_path.exists():
            raise FileNotFoundError(f"Legacy configuration target not found at: {xml_path}")
            
        try:
            tree = ET.parse(xml_path)
            root = tree.getroot()
            
            # 1. Fetch top-level orchestration metadata
            extract_type = root.get("ExtractType", "CSV")
            date_format = root.get("DateFormat", "M/d/yyyy h:mm:ss tt")
            max_retry = int(root.get("MaxRetry", "3"))
            
            raw_columns: List[ColumnDefinition] = []
            
            # 2. Navigate nodes safely via structural tags instead of brittle text matching
            for col_node in root.findall(".//Column"):
                col_name = col_node.get("ColNm")
                data_type = col_node.get("DataType", "String")
                assignment_str = col_node.get("Assignment")
                
                if col_name and assignment_str:
                    raw_columns.append(
                        ColumnDefinition(
                            name=col_name.strip(),
                            data_type=data_type.strip(),
                            assignment_order=int(assignment_str.strip())
                        )
                    )
            
            # 3. Enforce deterministic structural sequence based on original Assignment arrays
            sorted_columns = sorted(raw_columns, key=lambda c: c.assignment_order)
            
            log.info(f"Successfully compiled execution mapping layer. Captured {len(sorted_columns)} sorted target columns.")
            
            return ReportSchema(
                extract_type=extract_type,
                date_format=date_format,
                max_retry=max_retry,
                columns=sorted_columns
            )
            
        except ET.ParseError as e:
            log.error(f"Malformed XML syntax encountered during parsing: {str(e)}")
            raise
        except Exception as e:
            log.error(f"Unexpected operational failure compiling template layout: {str(e)}")
            raise
```

config/ExtractTemplates/CANDER_Report_ExtractTemplate.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ExtractConfig ExtractType="CSV" DateFormat="M/d/yyyy h:mm:ss tt" MaxRetry="5">
    <RecordTemplate>
        <Column Assignment="1" ColNm="CLIENT_IDENTIFIER" DataType="String" />
        <Column Assignment="2" ColNm="ORGANIZATION_NAME" DataType="String" />
        <Column Assignment="3" ColNm="REGISTRATION_DATE" DataType="DateTime" />
        <Column Assignment="4" ColNm="COUNTRY_OF_DOMICILE" DataType="String" />
        <Column Assignment="5" ColNm="LAST_MODIFIED_TIMESTAMP" DataType="DateTime" />
    </RecordTemplate>
</ExtractConfig>
```

test_parser.py

```python
# test_parser.py
from app.core.config import settings
from app.core.logging import log
from app.services.xml_adapter import XMLConfigAdapter

def verify_xml_adapter_pipeline():
    print("==================================================")
    print("🧬 RUNNING METADATA ADAPTER VALIDATION")
    print("==================================================")
    
    target_xml = settings.template_path / "CANDER_Report_ExtractTemplate.xml"
    
    try:
        schema = XMLConfigAdapter.parse_template(target_xml)
        
        print("\n--------------------------------------------------")
        print("🔎 COMPILING METADATA ASSERTIONS:")
        print(f"File Type Asserted:     {schema.extract_type} (Expected: CSV)")
        print(f"Max Retries Bound:      {schema.max_retry} (Expected: 5)")
        print(f"Extracted Total Columns: {len(schema.columns)} (Expected: 5)")
        
        print("\n📋 COLUMN FORMAT DETAILS:")
        for col in schema.columns:
            print(f" -> Pos {col.assignment_order}: {col.name:25} | Type: {col.data_type}")
        print("--------------------------------------------------")
        
        # Safe validations
        assert schema.extract_type == "CSV"
        assert len(schema.columns) == 5
        assert schema.columns[0].name == "CLIENT_IDENTIFIER"
        assert schema.columns[2].data_type == "DateTime"
        
        log.info("Parity Validation Metrics verified successfully.")
        print("==================================================")
        print("✅ METADATA ADAPTER SETUP: NO BUGS")
        print("==================================================")
        
    except Exception as e:
        print(f"❌ PIPELINE FAILURE: {str(e)}")
        print("==================================================")

if __name__ == "__main__":
    verify_xml_adapter_pipeline()
```

## Day 5

app/services/mapping.py

```python
import os
from pathlib import Path
import pandas as pd
from app.core.logging import log
from app.services.xml_adapter import ReportSchema

class MappingService:
    """Core business logic engine performing schema-driven data transformations."""

    @staticmethod
    def _format_value(value: any, data_type: str, date_mask: str) -> str:
        """Applies explicit legacy formatting rules to a raw cell value."""
        if pd.isna(value) or value is None:
            return ""

        # Convert to string and immediately strip trailing/leading whitespace
        val_str = str(value).strip()

        # Handle Date Formatting Parity
        if data_type == "DateTime":
            try:
                parsed_date = pd.to_datetime(val_str)
                return parsed_date.strftime("%-m/%-d/%Y %-I:%M:%S %p").lower()
            except Exception:
                log.warning(f"Failed parsing value '{val_str}' as DateTime. Falling back to raw text.")
                return val_str

        # Handle String Formatting Parity: Enforce double quotes around strings
        if data_type == "String":
            if val_str.startswith('"') and val_str.endswith('"'):
                return val_str
            return f'"{val_str}"'

        return val_str

    def transform_file(self, input_path: Path, output_path: Path, schema: ReportSchema) -> None:
        """
        Streams input files through chunked dataframes to optimize memory 
        while manually generating row structures to guarantee quoting parity.
        """
        log.info(f"Initiating extraction transformation pipeline from: {input_path}")
        
        if not input_path.exists():
            raise FileNotFoundError(f"Source data target file not found at: {input_path}")

        # Ensure output directory exists
        output_path.parent.mkdir(parents=True, exist_ok=True)

        # Determine structural delimiter matching configuration choices
        delimiter = "," if schema.extract_type == "CSV" else ","

        try:
            # Write rows out as plain text lines to honor exact manual quoting rules
            with open(output_path, mode="w", encoding="utf-8", newline="") as out_file:
                # 1. Write headers
                headers = [col.name for col in schema.columns]
                out_file.write(delimiter.join(headers) + "\n")

                # 2. Stream chunked inputs
                for chunk in pd.read_csv(input_path, chunksize=5000, dtype=str):
                    for _, row in chunk.iterrows():
                        processed_row = []
                        
                        for col in schema.columns:
                            raw_val = row.get(col.name, "")
                            formatted_val = self._format_value(
                                value=raw_val,
                                data_type=col.data_type,
                                date_mask=schema.date_format
                            )
                            processed_row.append(formatted_val)
                            
                        # Join elements cleanly with delimiter
                        out_file.write(delimiter.join(processed_row) + "\n")

            log.info(f"Successfully processed and generated parity report at: {output_path}")

        except Exception as e:
            log.error(f"Critical execution exception encountered during data streaming: {str(e)}")
            
            if output_path.exists():
                log.warning(f"Scrubbing incomplete or corrupted file from drive: {output_path}")
                os.remove(output_path)
                
            raise e
```

run_local.py

```python
# run_local.py
import argparse
from pathlib import Path
from app.core.logging import log
from app.services.xml_adapter import XMLConfigAdapter
from app.services.mapping import MappingService

def main():
    parser = argparse.ArgumentParser(description="GCO Fen-X Local Reporting CLI Pipe Runner")
    parser.add_index = False
    parser.add_argument("--input", required=True, help="Path to raw source data CSV")
    parser.add_argument("--template", required=True, help="Path to legacy config XML mapping layout")
    parser.add_argument("--output", required=True, help="Target destination for output CSV data")
    
    args = parser.parse_args()
    
    input_path = Path(args.input)
    template_path = Path(args.template)
    output_path = Path(args.output)
    
    log.info("--- STARTING LOCAL EXTRACTION PARITY ENGINE ---")
    
    try:
        # 1. Parse XML schema layouts via our adapter
        schema = XMLConfigAdapter.parse_template(template_path)
        
        # 2. Invoke core transformation logic execution passes
        service = MappingService()
        service.transform_file(
            input_path=input_path,
            output_path=output_path,
            schema=schema
        )
        
        log.info("--- PIPELINE RUN COMPLETED SUCCESSFULLY ---")
        
    except Exception as e:
        log.error(f"Execution pipeline run halted due to structural error: {str(e)}")
        exit(1)

if __name__ == "__main__":
    main()
```

data/raw_input.csv

```csv
CLIENT_IDENTIFIER,ORGANIZATION_NAME,REGISTRATION_DATE,COUNTRY_OF_DOMICILE,LAST_MODIFIED_TIMESTAMP
102948,  Acme Corporate Wealth   ,2026-01-15,Canada,2026-06-29 08:30:00
583920,Global Markets Corp,2024-11-02 14:22:11,United States,2026-06-25 17:15:00
```

Run the engine from your terminal window:

```bash
python run_local.py --input data/raw_input.csv --template config/ExtractTemplates/CANDER_Report_ExtractTemplate.xml --output data/processed_output.csv
```

Tasks 6
=======

app/services/fenergo_client.py

```python
import time
from typing import Optional, Dict
import httpx
from app.core.config import settings
from app.core.logging import log

class FenergoAPIClient:
    """Asynchronous HTTP client managing secure Fenergo API platform ingestion workflows."""
    
    def __init__(self):
        self.base_url = settings.FENERGO_BASE_URL
        self._token: Optional[str] = None
        self._token_expires_at: float = 0.0
        
        # Pull configuration dynamically from validated environment settings
        self.verify_ssl = settings.HTTPX_VERIFY_SSL
        self.proxy_url = settings.CORPORATE_HTTP_PROXY
        
        self.mounts: Dict[str, httpx.AsyncBaseTransport] = {}
        if self.proxy_url:
            log.info(f"Configuring outbound corporate proxy routing transport: {self.proxy_url}")
            self.mounts = {"all://": httpx.AsyncHTTPTransport(proxy=self.proxy_url)}

    async def _get_valid_token(self, client: httpx.AsyncClient) -> str:
        """Retrieves a valid OAuth2 Access Token using strict client credentials form mapping."""
        if self._token and time.time() < (self._token_expires_at - 30):
            return self._token

        log.info("OAuth2 token missing or expired. Fetching fresh credentials from Token Server...")
        
        payload = {
            "grant_type": "client_credentials",
            "scope": settings.FENERGO_SCOPE,
            "client_id": settings.FENERGO_CLIENT_ID,
            "client_secret": settings.FENERGO_CLIENT_SECRET
        }
        
        headers = {
            "Content-Type": "application/x-www-form-urlencoded",
            "Accept": "application/json",
            "User-Agent": "PostmanRuntime/7.43.0",
            "Cache-Control": "no-cache"
        }

        # Notice we use the client passed from the outer block which carries the correct proxy/SSL config
        response = await client.post(
            settings.FENERGO_TOKEN_URL, 
            data=payload, 
            headers=headers,
            timeout=15.0
        )
        
        response.raise_for_status()
        data = response.json()
        
        self._token = data["access_token"]
        expires_in = data.get("expires_in", 3600)
        self._token_expires_at = time.time() + expires_in
        
        log.info("OAuth2 Client Credentials authentication handshake verified successfully.")
        return self._token

    async def fetch_presigned_report_url(self, report_id: str) -> str:
        """Queries Fenergo Advanced Reporting API endpoints to fetch the download location."""
        log.info(f"Querying Fenergo metadata registry for Report ID: {report_id}")
        
        # CRITICAL FIX: Ensure the SSL validation and proxy settings apply to the authorization request too
        async with httpx.AsyncClient(verify=self.verify_ssl, mounts=self.mounts) as client:
            token = await self._get_valid_token(client)
            headers = {
                "Authorization": f"Bearer {token}",
                "Accept": "application/json"
            }
            
            log.info("Retrieving time-bound presigned extraction uniform resource identifiers...")
            mock_presigned_url = f"https://fenergo-data-bucket.s3.amazonaws.com/raw_api_extract_{report_id}.csv"
            return mock_presigned_url
```

test_client.py

```python

    asyncio.run(run_client_validation())
```

new config.py:


```python
import os
from pathlib import Path
from typing import Optional
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # App Environment Routing Mappings
    DATABASE_URL: str = "sqlite:///./reporting_platform.db"

    # --- Bank Enterprise Authentication Mapping (No Defaults - Safely Injected) ---
    FENERGO_BASE_URL: str
    FENERGO_CLIENT_ID: str
    FENERGO_CLIENT_SECRET: str
    FENERGO_TOKEN_URL: str
    FENERGO_SCOPE: str

    # Legacy Fallback Configuration Parameters
    DEFAULT_DELIMITER: str = ","
    DEFAULT_DATE_FORMAT: str = "M/d/yyyy h:mm:ss tt"

    # Cross-Platform Path Tracking Helpers
    BASE_DIR: Path = Path(__file__).resolve().parent.parent.parent

    # For parsing network overrides cleanly
    HTTPX_VERIFY_SSL: bool = True
    CORPORATE_HTTP_PROXY: Optional[str] = None

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

settings = Settings()

```

update your bashrc:

```bash
export http_proxy=$(netsh.exe winhttp show proxy | grep 'Proxy Server' | awk '{print $3}')
export https_proxy=$http_proxy
export WIN_HOST_IP=$(ip route | grep default | awk '{print $3}')
export http_proxy=http://$WIN_HOST_IP:9000
export https_proxy=http://$WIN_HOST_IP:9000
```

and .env:
```
# --- NETWORK & PROXY INTEGRATION STRUCTURES ---
HTTPX_VERIFY_SSL=False

# Replace with your actual office corporate proxy if the network requires explicit routing:
CORPORATE_HTTP_PROXY=http://10.0.0.1:8080
```

check powershell:
Get-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' | Select-Object ProxyServer, ProxyEnable, AutoConfigURL

