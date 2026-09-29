# libmongoc-windows

Prebuilt Windows DLLs of the [MongoDB C Driver](https://github.com/mongodb/mongo-c-driver) (`libmongoc` and `libbson`), 1.x and 2.x, for x64 and x86, built with MSVC and OpenSSL 3.5 LTS.

MongoDB publishes the C driver as source code only. This repository builds each upstream release on GitHub Actions from the signed tarball, tests it against a real `mongod` over TLS, and publishes the result as release assets.

This is an unofficial build, not affiliated with or endorsed by MongoDB, Inc.

## Which branch

| Releases | DLLs | For |
| --- | --- | --- |
| `v2.*` (marked Latest) | `mongoc2.dll`, `bson2.dll` | New code: 2.x is the current upstream branch |
| `v1.*` | `libmongoc-1.0.dll`, `libbson-1.0.dll` | Delphi FireDAC, and applications written for the 1.x API |

The 1.x DLLs keep the `lib`-prefixed names that FireDAC looks for. Upstream dropped that prefix in 1.16, so a default 1.x build does not work with FireDAC. FireDAC cannot use 2.x, whose DLL names and API differ.

## Download

Get the zip for your branch and architecture from the [releases](https://github.com/jamalofski/libmongoc-windows/releases):

| File | For |
| --- | --- |
| `libmongoc-windows-<version>-x64.zip` | 64-bit applications |
| `libmongoc-windows-<version>-x86.zip` | 32-bit applications, including the 32-bit Delphi IDE |

Each zip contains:

```
bin/        the two driver DLLs (see above)
            libssl-3-x64.dll, libcrypto-3-x64.dll   (x86: libssl-3.dll, libcrypto-3.dll)
include/    libbson and libmongoc headers
lib/        import libraries (1.x: libmongoc-1.0.lib, libbson-1.0.lib; 2.x: mongoc2.dll.lib, bson2.dll.lib),
            CMake and pkg-config files
licenses/   mongo-c-driver and its bundled components, OpenSSL
```

## Usage

Copy the four DLLs from `bin/` next to your executable. The driver DLL imports the OpenSSL DLLs when it loads, so they are needed even if you never use TLS, and they must be OpenSSL 3.5 or later. If your application already ships OpenSSL 3.5 or later under the same file names, you can keep yours.

The DLLs need the [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist) for the same architecture, which most machines already have.

For C and C++ projects, add the two directories under `include/` to the include path and link the two import libraries from `lib/`, or add the extracted folder to `CMAKE_PREFIX_PATH`.

## Verify a download

Each release comes with a `SHA256SUMS` file and a [build provenance attestation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds) for every zip and every DLL, which proves the file was produced by this repository's workflow:

```sh
sha256sum -c SHA256SUMS
gh attestation verify libmongoc-windows-2.5.5-x64.zip --repo jamalofski/libmongoc-windows
gh attestation verify mongoc2.dll --repo jamalofski/libmongoc-windows
```

## How it is built

[`build.yml`](.github/workflows/build.yml) runs on a GitHub-hosted `windows-2025` runner, with the MSVC 14.44 toolset from Visual Studio 2022 17.14:

1. It downloads the mongo-c-driver release and checks its signature against the [MongoDB C Driver release key](keys/mongo-c-driver.asc) (`6DB5 5D82 23FF 44E4 9DCB 9813 44E7 6C05 65AB C463`).
2. It builds OpenSSL from the latest 3.5 LTS release, after checking the tarball signature against the [OpenSSL signing certificate](keys/openssl.asc) (`B146 647E 45A7 B339 47AB 226B 2A2C 87D1 6169 2D40`), with OpenSSL's default Windows directories.
3. It builds libbson and libmongoc with `ENABLE_SSL=OPENSSL`, `ENABLE_SASL=SSPI`, `ENABLE_SRV=ON`, `ENABLE_ZLIB=BUNDLED`, `ENABLE_MONGODB_AWS_AUTH=ON`, `ENABLE_SNAPPY=OFF`, `ENABLE_ZSTD=OFF` and `ENABLE_CLIENT_SIDE_ENCRYPTION=OFF`. On 1.x it adds `BSON_OUTPUT_BASENAME=libbson` and `MONGOC_OUTPUT_BASENAME=libmongoc`.
4. It checks the architecture and version of every DLL, and that each dependency is either in the package or part of Windows.
5. It compiles a [smoke test](test/smoke.c) and runs it with only the package and Windows on the DLL search path, against a local `mongod` that requires TLS and a client certificate.

[`check.yml`](.github/workflows/check.yml) looks for new upstream 1.x and 2.x releases every week and builds and publishes them the same way.

## Versions

Release tags follow upstream: `v2.5.5` is mongo-c-driver 2.5.5 and `v1.30.12` is mongo-c-driver 1.30.12. Each release lists the OpenSSL and compiler versions it was built with.

Only new mongo-c-driver releases trigger a build, and each build takes the latest OpenSSL 3.5 release available at that time. A new OpenSSL release on its own does not produce a new package.

## License

The build scripts in this repository are under the [MIT license](LICENSE). The binaries are distributed under their own licenses, included in each zip: Apache License 2.0 for the MongoDB C Driver and for OpenSSL, plus the notices of the code bundled in the driver.

MongoDB is a trademark of MongoDB, Inc.
