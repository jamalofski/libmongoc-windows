# libmongoc-windows

Prebuilt Windows DLLs of the [MongoDB C Driver](https://github.com/mongodb/mongo-c-driver) 1.x (`libmongoc` and `libbson`) for x64 and x86, built with MSVC and OpenSSL 3.5 LTS.

MongoDB publishes the C driver as source code only. This repository builds each 1.x release on GitHub Actions from the signed upstream tarball, tests it against a real `mongod` over TLS, and publishes the result as release assets.

The DLLs keep the `lib`-prefixed names that Delphi FireDAC looks for, `libmongoc-1.0.dll` and `libbson-1.0.dll`. Upstream dropped that prefix in 1.16, so a default build does not work with FireDAC.

This is an unofficial build, not affiliated with or endorsed by MongoDB, Inc.

## Download

Get the zip for your architecture from the [latest release](https://github.com/jamalofski/libmongoc-windows/releases/latest):

| File | For |
| --- | --- |
| `libmongoc-windows-<version>-x64.zip` | 64-bit applications |
| `libmongoc-windows-<version>-x86.zip` | 32-bit applications, including the 32-bit Delphi IDE |

Each zip contains:

```
bin/        libmongoc-1.0.dll, libbson-1.0.dll
            libssl-3-x64.dll, libcrypto-3-x64.dll   (x86: libssl-3.dll, libcrypto-3.dll)
include/    libbson-1.0 and libmongoc-1.0 headers
lib/        libmongoc-1.0.lib, libbson-1.0.lib, CMake and pkg-config files
licenses/   mongo-c-driver and its bundled components, OpenSSL
```

## Usage

Copy the four DLLs from `bin/` next to your executable. `libmongoc-1.0.dll` imports the OpenSSL DLLs when it loads, so they are needed even if you never use TLS, and they must be OpenSSL 3.5 or later. If your application already ships OpenSSL 3.5 or later under the same file names, you can keep yours.

The DLLs need the [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist) for the same architecture, which most machines already have.

For C and C++ projects, add `include/libbson-1.0` and `include/libmongoc-1.0` to the include path and link `libmongoc-1.0.lib` and `libbson-1.0.lib`, or add the extracted folder to `CMAKE_PREFIX_PATH`.

## Verify a download

Each release comes with a `SHA256SUMS` file and a [build provenance attestation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds) for every zip and every DLL, which proves the file was produced by this repository's workflow:

```sh
sha256sum -c SHA256SUMS
gh attestation verify libmongoc-windows-1.30.12-x64.zip --repo jamalofski/libmongoc-windows
gh attestation verify libmongoc-1.0.dll --repo jamalofski/libmongoc-windows
```

## How it is built

[`build.yml`](.github/workflows/build.yml) runs on a GitHub-hosted `windows-2025` runner, with the MSVC 14.44 toolset from Visual Studio 2022 17.14:

1. It downloads the mongo-c-driver release and checks its signature against the [MongoDB C Driver release key](keys/mongo-c-driver.asc) (`6DB5 5D82 23FF 44E4 9DCB 9813 44E7 6C05 65AB C463`).
2. It builds OpenSSL from the official source tarball, whose SHA-256 is pinned in the workflow, with OpenSSL's default Windows directories.
3. It builds libbson and libmongoc with `ENABLE_SSL=OPENSSL`, `ENABLE_SASL=SSPI`, `ENABLE_SRV=ON`, `ENABLE_ZLIB=BUNDLED`, `ENABLE_MONGODB_AWS_AUTH=ON`, `ENABLE_SNAPPY=OFF`, `ENABLE_ZSTD=OFF`, `ENABLE_CLIENT_SIDE_ENCRYPTION=OFF`, `BSON_OUTPUT_BASENAME=libbson` and `MONGOC_OUTPUT_BASENAME=libmongoc`.
4. It checks the architecture and version of every DLL, and that each dependency is either in the package or part of Windows.
5. It compiles a [smoke test](test/smoke.c) and runs it with only the package and Windows on the DLL search path, against a local `mongod` that requires TLS and a client certificate.

[`check.yml`](.github/workflows/check.yml) looks for a new upstream 1.x release every week and builds and publishes it the same way.

## Versions

Release tags follow upstream: `v1.30.12` is mongo-c-driver 1.30.12. Each release lists the OpenSSL and compiler versions it was built with.

Only the 1.x branch is built, for FireDAC and other applications that expect its DLL names and API.

## License

The build scripts in this repository are under the [MIT license](LICENSE). The binaries are distributed under their own licenses, included in each zip: Apache License 2.0 for the MongoDB C Driver and for OpenSSL, plus the notices of the code bundled in the driver.

MongoDB is a trademark of MongoDB, Inc.
