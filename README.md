# Competency Proof

Competency Proof demonstrates how a worker can receive a digital competency
credential and use it to show an employer that they meet a job requirement.
The example follows an electrical installation technician, from assessment by a
certification body to verification by an employer.

The application connects an Android wallet and three web portals to real walt.id
issuer, wallet and verifier services. Credentials use synthetic data. The demo
organizations are fictional and make no accreditation claims.

## How it works

| Organization | Responsibility | Interface |
| --- | --- | --- |
| LSP Nusantara | Record assessments and issue competency credentials | Issuer portal |
| NusaCred Wallet | Manage custodial wallets, participant access and account recovery | Android wallet and custodian portal |
| Karya Instalasi | Publish job requirements, approve trusted issuers and verify credentials | Employer portal and public job page |

1. A participant signs in to the Android wallet and obtains their recipient code.
2. An issuer operator records a synthetic assessment, binds it to that participant
   and creates a credential offer.
3. The participant scans the offer QR code or opens its App Link, reviews the
   details and approves receipt. The credential is issued and stored in the
   server-side wallet; the application reads it back before confirming success.
4. The employer configures the job requirement and reviews and approves the
   issuer's identity in **Trusted issuers**.
5. On the public job page, the participant selects **Check my competency** and
   opens the verification request in the wallet.
6. The participant selects a stored credential, reviews the requested information
   and approves presentation. The walt.id verifier checks the proof, and the
   employer application evaluates issuer trust and the job requirement before
   recording the result.

A valid signature alone does not establish that an issuer is trusted or that a
credential meets the job requirement. Those are separate employer decisions.

## Architecture and integration

The application backends handle authorization, assessments, consent and business
decisions. The companion
[walt.id fork](https://github.com/nusacred-lab/waltid-identity) handles credential
issuance, storage and cryptographic verification through OpenID4VCI and
OpenID4VP. Keycloak provides OIDC authentication, with separate identity services
for the issuer, wallet provider and employer.

The wallet is custodial: holder signing keys and credentials stay on the server.
The Android app provides sign-in, QR scanning, credential review and explicit
consent. Account recovery preserves the existing wallet and credentials while
revoking earlier access.

| Module | Purpose |
| --- | --- |
| `android-wallet` | Native Android participant wallet built with Jetpack Compose |
| `web-portals` | Issuer, custodian and employer interfaces built with Compose Multiplatform for WebAssembly |
| `issuer-backend` | Assessment records and recipient-bound credential offers |
| `custodian-backend` | Wallet access, receiving, presentation, recovery and participant invitations |
| `company-backend` | Job requirements, issuer trust and verification results |
| `backend-common` | Shared Ktor authentication, persistence and protocol integration |
| `shared-contracts` | API models shared by JVM, Android and web clients |
| `integration-tests` | Browser/API contract checks |
| `keycloak-recovery-provider` | Guarded password recovery integration |

Application state uses SQLite; identity services use PostgreSQL. The
[API contract](docs/openapi.json) describes the application endpoints.

## Run the demo

Deployment lives in
[platform-infra](https://github.com/nusacred-lab/platform-infra), which provides
Docker Compose, release verification, private configuration preparation and
operational scripts. Start with its
[installation guide](https://github.com/nusacred-lab/platform-infra/blob/main/release/README.md).
The installation uses prebuilt images and does not require building this repository.

Install the matching signed APK separately on Android 11 or newer. Public phone
access requires trusted HTTPS and the App Link association for that APK's signing
certificate. The installer generates fresh synthetic accounts for an isolated
installation; participant invitations require additional operational configuration.

## Build from source

Use JDK 21, Python 3, and an Android SDK with platform 37 and Build Tools 37.0.0.
The Gradle wrapper is included. From the repository root:

```sh
python3 ci/check_policy.py
python3 ci/verify_dependency_review.py
python3 ci/check_api_contract.py
./gradlew :shared-contracts:jvmTest :backend-common:test \
  :custodian-backend:test :integration-tests:test \
  :issuer-backend:installDist :company-backend:installDist \
  :custodian-backend:installDist :android-wallet:assembleDebug \
  :android-wallet:lintDebug :web-portals:wasmJsBrowserDistribution \
  --dependency-verification strict --no-daemon
```

These checks build the application and exercise its local contracts. Running the
credential journey also requires the deployed identity and walt.id services.
See the [release guide](docs/p6-release.md) for APK verification and offline signing.

## Scope and limitations

The demo covers one issuer, one employer, one job opening and a fixed competency
credential profile. Credential revocation, multiple issuers or employers, CVs and
a complete recruitment process are outside its scope. Presentation uses the
profile's required claims; arbitrary selective disclosure is not supported.

Custody makes the wallet provider responsible for holder keys and service
availability. This setup is a synthetic demonstration, not a production deployment.

## Further reading

- [Wallet and portal behavior](docs/p5-experience.md)
- [Credential flows and verification policies](docs/P4-credential-business.md)
- [Account recovery](docs/P2-recovery-proof.md)
- [Security and review policy](SECURITY-PROJECT.md)

Licensed under [Apache-2.0](LICENSE). This project has no official affiliation
with walt.id.
