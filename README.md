This fork was created for use with amrayn/licensepp and Bazel.

WORKSPACE

```
# devguy-com/cryptopp is needed by licensepp
http_archive(
    name = "cryptopp",
    build_file_content = all_content,
    strip_prefix = "cryptopp-master",
    url = "https://github.com/devguy-com/cryptopp/archive/master.zip",
)

# pe-commons/licensepp
new_git_repository(
    name = "licensepp",
    branch="master",
    build_file = "//third_party/licensepp:BUILD",
    remote = "git@bitbucket.org:pecommons/licensepp.git",
)

# devguy-com/ripe is needed by licenseepp
http_archive(
    name = "ripe",
    strip_prefix = "ripe-master",
    build_file = "//third_party/ripe:BUILD",
    url = "https://github.com/devguy-com/ripe/archive/master.zip",
)

# zlib is needed by licensepp
http_archive(
    name = "zlib",
    sha256 = "c3e5e9fdd5004dcb542feda5ee4f0ff0744628baf8ed2dd5d66f8ca1197cb1a1",
    strip_prefix = "zlib-1.2.11",
    urls = [
        "https://mirror.bazel.build/zlib.net/zlib-1.2.11.tar.gz",
        "https://zlib.net/zlib-1.2.11.tar.gz",
    ],
    build_file = "//third_party/zlib:BUILD"
)
```

third_party/zlib/BUILD

```
# copied from: https://github.com/bazelbuild/bazel/blob/master/third_party/zlib/BUILD

cc_library(
    name = "zlib",
    srcs = glob(["*.c"]),
    hdrs = glob(["*.h"]),
    # Use -Dverbose=-1 to turn off zlib's trace logging. (bazelbuild/bazel#3280)
    copts = ["-w", "-Dverbose=-1"],
    includes = ["."],
    visibility = ["//visibility:public"],
)
```

third_party/ripe/BUILD
```
cc_binary(
    name = "ripe",
    srcs = [
        "include/Ripe.h",
        "lib/Ripe.cc",
        "src/ripe.cc",
    ],
    deps = ["@exd_edge_compute//third_party/cryptopp", "@zlib"],
    defines = ['RIPE_VERSION=\\"1.0\\"']
)
```

third_party/cryptopp/BUILD
```
load("@rules_foreign_cc//tools/build_defs:make.bzl", "make")

make(
    name = "cryptopp",
    lib_source = "@cryptopp//:all",
    static_libraries =
        ["libcryptopp.a"],
    visibility = ["//visibility:public"],
)
```

third_party/licensepp/BUILD
```
load("@rules_foreign_cc//tools/build_defs:cmake.bzl", "cmake_external")

config_setting(
    name = "darwin_build",
    values = {"cpu": "darwin"},
)

filegroup(
    name = "all",
    srcs = glob(["**"]),
    visibility = ["//visibility:public"],
)

cmake_external(
    name = "licensepp",
    # Values to be passed as -Dkey=value on the CMake command line;
    # here are serving to provide some CMake script configuration options
    cache_entries = {
        "MAKEFLAGS": "-j5",
        "CRYPTOPP_ROOT_DIR": "$EXT_BUILD_DEPS/cryptopp",
    },
    defines = ["NDEBUG"],
    lib_source = "@licensepp//:all",
    shared_libraries =
        select({
            ":darwin_build": [
                "liblicensepp.dylib",
            ],
            "//conditions:default": [
                "liblicensepp.so",
            ],
        }),
    visibility = ["//visibility:public"],
    deps = [
        "//third_party/cryptopp",
    ],
)

filegroup(
    name = "cli_files",
    srcs = [
        "cli/licensing/license-manager.h",
        "cli/licensing/license-manager-key-register.cc",
        "cli/licensing/license-manager-key-register.h",
        "cli/main.cc",
    ],
    visibility = ["//visibility:public"],
)

cc_binary(
    name = "license-manager",
    srcs = ["@licensepp//:cli_files"],
    defines = ["NDEBUG"],
    deps = ["//third_party/licensepp"],
)

filegroup(
    name = "licensepp_verify_files",
    srcs = [
      "cli/validate-license.cc",
      "cli/licensing/license-manager-key-register.cc",
      "cli/licensing/license-manager.h",
      "cli/licensing/license-manager-key-register.h",
    ],
    visibility = ["//visibility:public"],
)

cc_library(
  name = "licensepp_verify",
  srcs = ["@licensepp//:licensepp_verify_files"],
  defines = ["NDEBUG"],
  deps = ["licensepp"],
  visibility = ["//visibility:public"],
)
```

Building the license manager executable:

```
bazel build -s //third_party/licensepp:license-manager
```

