Of course. Here is the technical documentation formatted as a standard Markdown response.

# Technical Documentation: Project Setu

| | |
| :--- | :--- |
| **Version:** | 1.0 |
| **Date:** | July 30, 2025 |
| **Author:** | Gemini |
| **Status:** | Draft |

-----

### Table of Contents

1.  [Introduction](https://www.google.com/search?q=%231-introduction)
2.  [High-Level Architecture](https://www.google.com/search?q=%232-high-level-architecture)
      - [2.1. Core Philosophy: Clean Architecture](https://www.google.com/search?q=%2321-core-philosophy-clean-architecture)
      - [2.2. The Dependency Rule](https://www.google.com/search?q=%2322-the-dependency-rule)
3.  [Detailed Directory and Code Structure](https://www.google.com/search?q=%233-detailed-directory-and-code-structure)
4.  [Core Components Deep Dive](https://www.google.com/search?q=%234-core-components-deep-dive)
      - [4.1. The `domain` Layer](https://www.google.com/search?q=%2341-the-domain-layer)
      - [4.2. The `adapters` Layer](https://www.google.com/search?q=%2342-the-adapters-layer)
      - [4.3. The `application` Layer](https://www.google.com/search?q=%2343-the-application-layer)
5.  [Configuration Management](https://www.google.com/search?q=%235-configuration-management)
6.  [Usage Example: Integrating the Storage Service](https://www.google.com/search?q=%236-usage-example-integrating-the-storage-service)
7.  [Development and Contribution Guidelines](https://www.google.com/search?q=%237-development-and-contribution-guidelines)
8.  [Technology Stack](https://www.google.com/search?q=%238-technology-stack)

-----

### 1\. Introduction

This document provides a detailed technical overview of **Project Setu**. It describes the internal architecture, design decisions, and implementation guidelines. The primary audience is software developers who will be building, using, or contributing to the Setu library.

### 2\. High-Level Architecture

#### 2.1. Core Philosophy: Clean Architecture

Setu is built upon the principles of **Clean Architecture**. The entire design is centered around creating a system that is:

  - **Independent of Frameworks:** The core logic does not depend on external libraries or frameworks.
  - **Testable:** The business rules can be tested without any external dependencies.
  - **Independent of UI:** The core logic can be used with any type of interface (e.g., CLI, web app).
  - **Independent of Database/Services:** The core logic is decoupled from specific data sources or services, making them swappable.

This is achieved by separating the code into concentric layers:

1.  **Domain (Entities & Interfaces):** The innermost layer. Contains the core business logic and abstract definitions (interfaces).
2.  **Application (Use Cases):** Orchestrates the flow of data using the interfaces from the Domain layer.
3.  **Adapters:** The outermost layer. Contains the concrete implementations of the interfaces, connecting the application to external services like Keycloak or Minio.

#### 2.2. The Dependency Rule

The fundamental rule that makes this architecture work is the **Dependency Inversion Principle**.

> Source code dependencies can only point **inwards**.

  - The `adapters` layer depends on the `application` and `domain` layers.
  - The `application` layer depends on the `domain` layer.
  - The `domain` layer depends on **nothing**.

This ensures that a change in an external service (e.g., replacing Minio with S3) only affects the specific adapter in the outer layer and has zero impact on the core application logic.

### 3\. Detailed Directory and Code Structure

The project follows a `src` layout for clear separation between source code and project files.

```
setu/
├── .gitignore
├── LICENSE
├── pyproject.toml
├── README.md
│
├── docs/
│   ├── functional.md
│   ├── non_functional.md
│   └── technical_architecture.md
│
├── docker/
│   ├── keycloak/
│   │   └── docker-compose.yml
│   ├── minio/
│   │   └── docker-compose.yml
│   └── postal/
│       └── docker-compose.yml
│
└── src/
    └── setu/
        ├── __init__.py
        │
        ├── application/
        │   ├── __init__.py
        │   └── use_cases.py
        │
        ├── domain/
        │   ├── __init__.py
        │   ├── auth/
        │   │   ├── __init__.py
        │   │   ├── entities.py
        │   │   └── interfaces.py      # IAuthService contract
        │   ├── storage/
        │   │   ├── __init__.py
        │   │   ├── entities.py
        │   │   └── interfaces.py      # IStorageService contract
        │   ├── notification/
        │   │   └── ...
        │   └── gateway/
        │       └── ...
        │
        ├── adapters/
        │   ├── __init__.py
        │   ├── auth/
        │   │   ├── __init__.py
        │   │   └── keycloak/          # Keycloak implementation
        │   │       ├── __init__.py
        │   │       └── adapter.py
        │   ├── storage/
        │   │   ├── __init__.py
        │   │   ├── minio/             # Minio implementation
        │   │   │   ├── __init__.py
        │   │   │   └── adapter.py
        │   │   └── s3/                # Future S3 implementation
        │   │       ├── __init__.py
        │   │       └── adapter.py
        │   └── ...
        │
        ├── config/
        │   ├── __init__.py
        │   └── settings.py
        ├── exceptions/
        │   ├── __init__.py
        │   └── general.py
        └── utils/
            ├── __init__.py
            └── helpers.py
```

### 4\. Core Components Deep Dive

#### 4.1. The `domain` Layer

This is the heart of the library. It contains only pure Python code with no external dependencies.

  - **`domain/{context}/interfaces.py`**: These files define the abstract base classes (ABCs) or Protocols that act as "contracts."

*Example: `domain/storage/interfaces.py`*

```python
from abc import ABC, abstractmethod

class IStorageService(ABC):
    @abstractmethod
    def upload_file(self, source_path: str, destination_name: str) -> str:
        """Uploads a file and returns its identifier or URL."""
        pass

    @abstractmethod
    def get_presigned_url(self, file_name: str) -> str:
        """Generates a temporary URL for a private file."""
        pass
```

#### 4.2. The `adapters` Layer

This layer provides the concrete implementations for the interfaces defined in the domain.

  - **`adapters/{context}/{implementation}/adapter.py`**: Each file contains a class that inherits from a domain interface and implements its methods using a specific SDK (e.g., `minio`, `python-keycloak-client`).

*Example: `adapters/storage/minio/adapter.py`*

```python
from minio import Minio
from setu.domain.storage.interfaces import IStorageService
from setu.config.settings import MinioSettings

class MinioStorageAdapter(IStorageService):
    def __init__(self, settings: MinioSettings):
        self._client = Minio(
            endpoint=settings.endpoint,
            access_key=settings.access_key,
            secret_key=settings.secret_key,
            secure=settings.secure
        )
        self._bucket_name = settings.bucket_name

    def upload_file(self, source_path: str, destination_name: str) -> str:
        # Implementation using self._client
        # ...
        return f"Uploaded {destination_name}"

    def get_presigned_url(self, file_name: str) -> str:
        # Implementation using self._client
        # ...
        return "http://minio-presigned-url"
```

#### 4.3. The `application` Layer

This layer is the orchestrator. It uses the interfaces to perform actions, without knowing the concrete implementation. This layer is thin by design.

*Example: `application/use_cases.py`*

```python
from setu.domain.storage.interfaces import IStorageService
from setu.domain.auth.interfaces import IAuthService

class UserUseCases:
    def __init__(self, auth_service: IAuthService, storage_service: IStorageService):
        self._auth_service = auth_service
        self._storage_service = storage_service

    def upload_user_avatar(self, user_id: str, avatar_path: str) -> str:
        # 1. Check if user is valid (using auth service)
        user = self._auth_service.get_user_by_id(user_id)
        if not user:
            raise ValueError("User not found")

        # 2. Upload the file (using storage service)
        destination = f"avatars/{user_id}.png"
        result = self._storage_service.upload_file(avatar_path, destination)
        
        return result
```

### 5\. Configuration Management

  - **Pydantic for Settings:** The `config` package will use `pydantic` to define settings models for each service (e.g., `MinioSettings`, `KeycloakSettings`).
  - **Runtime Injection:** The consuming application will be responsible for instantiating these settings models (e.g., from environment variables) and passing them into the adapter constructors. This upholds the "No Secret Storage" security principle.

### 6\. Usage Example: Integrating the Storage Service

An application developer using Setu would perform the following steps:

1.  **Install the library:**

    ```bash
    pip install setu
    ```

2.  **Configure and initialize the adapter in their application:**

    ```python
    # In the main application (e.g., a FastAPI app)
    import os
    from setu.adapters.storage.minio import MinioStorageAdapter
    from setu.config.settings import MinioSettings

    # 1. Load settings from environment variables
    minio_settings = MinioSettings(
        endpoint=os.getenv("MINIO_ENDPOINT"),
        access_key=os.getenv("MINIO_ACCESS_KEY"),
        secret_key=os.getenv("MINIO_SECRET_KEY"),
        bucket_name="my-app-bucket"
    )

    # 2. Instantiate the concrete adapter
    storage_service = MinioStorageAdapter(settings=minio_settings)

    # 3. Use the service to perform an action
    @app.post("/upload")
    def upload_image(file: UploadFile):
        # The application code only knows about the interface methods
        url = storage_service.upload_file(file.file, file.filename)
        return {"url": url}
    ```

    If they later decide to switch to S3, they would only change the initialization step to use `S3StorageAdapter`—the rest of their application code remains unchanged.

### 7\. Development and Contribution Guidelines

  - **Setup:** Clone the repository. Run `docker-compose up` from the `docker/` subdirectories to start the required services (Minio, Keycloak).
  - **Dependencies:** Create a virtual environment and run `pip install -r requirements-dev.txt`.
  - **Code Quality:** Before committing, run `black .`, `isort .`, and `flake8`.
  - **Testing:** Run all tests with `pytest`. All new code must be accompanied by unit tests.

### 8\. Technology Stack

  - **Language:** Python 3.8+
  - **Configuration:** Pydantic
  - **Core Dependencies:** `requests`, `minio`, `python-keycloak-client`
  - **Development Tools:** `pytest`, `black`, `flake8`, `isort`, `mypy`