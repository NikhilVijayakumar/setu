# Functional Documentation: Project Setu

| |               |
| :--- |:--------------|
| **Version:** | 1.0           |
| **Date:** | July 30, 2025 |
| **Author:** | Nikhil        |
| **Status:** | Draft         |

-----

### Table of Contents

1.  [Introduction]()
2.  [Core Problem Statement]()
3.  [Goals and Scope]()
4.  [Core Features (Functional Requirements)]()
      - [4.1. Authentication Service (`IAuthService`)]()
      - [4.2. Storage Service (`IStorageService`)]()
      - [4.3. Notification Service (`INotificationService`)]()
      - [4.4. API Gateway Service (`IGatewayService`)]()
5.  [User Roles and Personas]()
6.  [Assumptions and Dependencies]()
#  Non Functional Documentation for Sutradhar
7.  [Out of Scope]()

-----

### 1\. Introduction

This document outlines the functional requirements and scope for **Project Setu (सेतु)**. Setu is a centralized middleware support library designed to provide a standardized, abstract interface for common backend services. The primary goal of Setu is to simplify integration, improve maintainability, and reduce code duplication across multiple applications by creating a unified "bridge" to essential infrastructure components.

The intended audience for this document includes project stakeholders, software developers who will use the library, and architects.

### 2\. Core Problem Statement

Modern distributed applications often rely on a common set of external services for handling authentication, file storage, notifications, and API routing. Integrating these services directly into each application leads to several challenges:

  - **Code Duplication:** The same boilerplate integration and configuration logic is rewritten in multiple projects.
  - **Inconsistency:** Slight variations in implementation across projects can lead to subtle bugs and security vulnerabilities.
  - **High Maintenance Overhead:** Updating a service (e.g., upgrading a Keycloak version or migrating from Minio to S3) requires finding and modifying code across every single project repository.
  - **Vendor Lock-in:** Applications become tightly coupled to specific service implementations, making future migrations difficult and costly.

> Project Setu aims to solve these problems by providing a single, well-tested, and versioned library that abstracts these integrations away from the application's core business logic.

### 3\. Goals and Scope

#### 3.1. Goals

  - To provide a simple, unified API for interacting with core backend services.
  - To abstract away the specific implementation details of third-party services.
  - To centralize integration logic for easier updates and maintenance.
  - To enable seamless swapping of backend services with minimal changes to the consuming application.
  - To ensure all service integrations are consistent and follow best practices.

#### 3.2. Scope

The initial scope of **Project Setu** will cover abstractions for the following four domains:

1.  **Authentication & Authorization**
2.  **File Storage**
3.  **User Notifications**
4.  **API Gateway Management**

### 4\. Core Features (Functional Requirements)

This section details the specific functions the library will expose to the consuming application.

#### 4.1. Authentication Service (`IAuthService`)

The Authentication Service will provide functionalities for managing user identity and access control.

  - **FR-AUTH-01: User Login:** The library must provide a function to authenticate a user with their credentials (e.g., username/password) and return an access token or user session object.
  - **FR-AUTH-02: Token Validation:** The library must provide a function to validate an access token to ensure it is authentic and not expired.
  - **FR-AUTH-03: Token Refresh:** The library must provide a function to refresh an expired access token using a refresh token.
  - **FR-AUTH-04: User Logout:** The library must provide a function to invalidate a user's session or token (e.g., by logging them out from the identity provider).
  - **FR-AUTH-05: Fetch User Profile:** The library must provide a function to retrieve the profile information of the currently authenticated user from the identity provider.
  - **FR-AUTH-06: Check Permissions:** The library must provide a function to check if an authenticated user has a specific role or permission.

#### 4.2. Storage Service (`IStorageService`)

The Storage Service will provide functionalities for object storage operations.

  - **FR-STOR-01: Upload File:** The library must provide a function to upload a file (from a byte stream or file path) to a specified bucket or container.
  - **FR-STOR-02: Download File:** The library must provide a function to download a file as a byte stream.
  - **FR-STOR-03: Get Public URL:** The library must provide a function to get a publicly accessible, permanent URL for a given file object.
  - **FR-STOR-04: Get Pre-signed URL:** The library must provide a function to generate a temporary, time-limited URL for securely downloading or uploading a private file.
  - **FR-STOR-05: Delete File:** The library must provide a function to permanently delete a file from storage.
  - **FR-STOR-06: Check File Existence:** The library must provide a function to check if a file with a given name exists in a bucket.

#### 4.3. Notification Service (`INotificationService`)

The Notification Service will provide functionalities for sending communications to users. The initial focus is on email.

  - **FR-NOTIF-01: Send Email:** The library must provide a function to send an email to a single recipient, accepting parameters for the recipient's address, subject, and body (both plain text and HTML).
  - **FR-NOTIF-02: Send Bulk Email:** The library must provide a function to send a templated email to multiple recipients.

#### 4.4. API Gateway Service (`IGatewayService`)

The API Gateway Service will provide functionalities for programmatically configuring the API gateway.

  - **FR-GW-01: Register Service:** The library must provide a function to register a new downstream service with the gateway.
  - **FR-GW-02: Create Route:** The library must provide a function to create a new route that maps a public-facing path (e.g., `/api/users`) to a registered downstream service.
  - **FR-GW-03: Add Middleware/Plugin:** The library must provide a function to attach a plugin (e.g., rate-limiting, authentication) to a specific route or service.

### 5\. User Roles and Personas

  - **Application Developer:** This is the primary user of the Setu library. They will install the library in their application and use its functions to integrate with the backend services without needing to know the underlying implementation details.

### 6\. Assumptions and Dependencies

  - The consuming application is responsible for providing all necessary configurations (e.g., API endpoints, credentials, bucket names) to the Setu library at runtime, typically via environment variables.
  - The Setu library itself will not store any secrets or sensitive configuration data.
  - Network access from the application environment to the respective backend services (Keycloak, Minio, etc.) is available and correctly configured.

### 7\. Out of Scope

  - **Providing the Services:** Setu is a library, not a service provider. It does not include a Keycloak server or a Minio instance; it only provides the tools to connect to them.
  - **Business Logic:** The library will not contain any application-specific business logic. Its role is strictly limited to infrastructure integration.
  - **User Interface (UI):** Setu is a backend library and has no UI components.