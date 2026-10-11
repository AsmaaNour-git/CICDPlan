# CICDPlan
4 days plan
# Apigee CI/CD POC — Hello Name API

## 1. Project Overview

This project demonstrates local Apigee API proxy development using VS Code, the Apigee Emulator, Docker, and GitHub.

The `hello-name` proxy accepts a name as a query parameter and returns a JSON greeting. It also validates that the name parameter is provided.

## 2. Local Development Setup

### Prerequisites

- Windows
- Visual Studio Code
- Git
- Docker Desktop with the Linux container engine running
- Apigee extension for Cloud Code in VS Code
- Apigee Emulator image: `gcr.io/apigee-release/hybrid/apigee-emulator:1.9.2`

### Workspace Location

The local workspace is:

`D:\Asmaa\Devops\cicd-workspace`

The proxy is located at:

`src/main/apigee/apiproxies/hello-name/apiproxy`

### Proxy Structure

```text
hello-name/
└── apiproxy/
    ├── policies/
    │   ├── AM-Hello.xml
    │   └── RF-MissingName.xml
    ├── proxies/
    │   └── default.xml
    └── targets/
```

The proxy uses `AssignMessage` to generate a response and `RaiseFault` to return an error when the name parameter is missing. No backend service is required.

## 3. Running the Apigee Emulator

### Step 1: Start Docker Desktop

Open Docker Desktop and wait until the engine is running.

### Step 2: Verify the emulator container

Run the following PowerShell command:

```powershell
docker ps
```

Confirm that the `apigee-local` container is running.

The emulator version used in this POC is `1.9.2`.

### Step 3: Deploy the local environment

1. Open the workspace in VS Code.
2. Open the Cloud Code Apigee panel.
3. Select the local `dev` environment.
4. Deploy the environment to the `apigee-local` emulator.
5. Confirm that the deployment succeeds in the Output panel.

### Step 4: Verify the listener

The proxy is accessed through port `8998`:

`http://localhost:8998`

The root path `/` is not a configured proxy endpoint, so a 404 response at the root is expected.

## 4. API Endpoint Behavior

### Successful Request

**Method:** GET

**Endpoint:**

`http://localhost:8998/v1/hello?name=Asmaa`

Test with PowerShell:

```powershell
curl.exe -i "http://localhost:8998/v1/hello?name=Asmaa"
```

**Expected result:** HTTP 200 OK

```json
{
  "message": "Hello, Asmaa!"
}
```

### Missing Name Parameter

**Endpoint:**

`http://localhost:8998/v1/hello`

Test with PowerShell:

```powershell
curl.exe -i "http://localhost:8998/v1/hello"
```

**Expected result:** HTTP 400 Bad Request

```json
{
  "error": "The name query parameter is required."
}
```

### Different Name

```powershell
curl.exe -i "http://localhost:8998/v1/hello?name=Ahmed"
```

**Expected result:** HTTP 200 OK

```json
{
  "message": "Hello, Ahmed!"
}
```

## 5. Known Limitations

- The proxy runs locally in the Apigee Emulator; it is not deployed to a production Apigee organization.
- Docker Desktop must be running before the emulator can be used.
- The API generates its response directly and does not integrate with a real backend.
- The current implementation checks whether the name parameter is missing. It does not validate empty strings, input length, special characters, or other malformed values.
- The proxy does not currently implement authentication, OAuth 2.0, JWT verification, quotas, or SpikeArrest.
- The emulator may not reproduce every behavior or feature of a fully provisioned Apigee environment.
- Port `8998` must remain available on the local machine.
- Local deployment is performed through VS Code Cloud Code; automated CI/CD deployment has not yet been configured.

## 6. Troubleshooting

### Docker connection error

Start Docker Desktop and verify the container status:

```powershell
docker ps
```

### Unable to identify proxy

Verify that the `hello-name` proxy is included in the local `dev` environment deployment and that its base path is `/v1/hello`.

### Deployment fails because of `targetURL`

The original generated backend target used an unresolved `{{targetURL}}` variable. This POC does not need a backend, so the unused `backend.xml` file was removed to allow deployment.

### Deployment successful but API returns 404

Confirm that the request uses port `8998`, the correct base path, and the deployed proxy.

## 7. Repository and Future Work

Source code repository:

https://github.com/AsmaaNour-git/CICDPlan

Planned next steps:

1. Version-control the proxy using Git.
2. Add automated API tests.
3. Create a Cloud Build pipeline.
4. Automate proxy validation and deployment.
5. Explore promotion from the development organization to the production organization.

## 8. Current Status

The `hello-name` proxy has been deployed successfully to the local Apigee Emulator and tested with both successful and missing-parameter requests.