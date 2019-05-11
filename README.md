![Supported Platforms](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-blue.svg)
[![Build Status](https://travis-ci.org/gulrak/filesystem.svg?branch=master)](https://travis-ci.org/gulrak/filesystem)
[![Build status](https://ci.appveyor.com/api/projects/status/t07wp3k2cddo0hpo/branch/master?svg=true)](https://ci.appveyor.com/project/gulrak/filesystem)
[![Latest Release Tag](https://img.shields.io/github/tag/gulrak/filesystem.svg)](https://github.com/gulrak/filesystem/tree/v1.1.4)

# Filesystem

This is a header-only single-file std::filesystem compatible helper library,
based on the C++17 specs, but implemented for C++11 or C++14 (so not 100%
conforming to the C++17 standard). It is currently tested on macOS 10.12, Windows 10,
and Ubuntu 18.04 but should work on other versions too, as long as you have a
C++11 compatible compiler. It is of course in its own namespace `ghc::filesystem`
to not interfere with a regular `std::filesystem` should you use it in a mixed C++17
environment.

*It could still use some polishing, test coverage is above 90%, I didn't benchmark
much yet, but I'll try to optimize some parts and refactor others, so I'm striving
to improve it as long as it doesn't introduce additional C++17 compatibility
issues. Feedback is always welcome. Simply open an issue if you see something missing
or wrong or not behaving as expected and I'll comment.*


## Motivation

I'm often in need of filesystem functionality, mostly `fs::path`, but directory
access too, and when beginning to use C++11, I used that language update
to try to reduce my third-party dependencies. I could drop most of what
I used, but still missed some stuff that I started implementing for the
fun of it. Originally I based these helpers on my own coding- and naming
conventions. When C++17 was finalized, I wanted to use that interface,
but it took a while, to push myself to convert my classes.

The implementation is closely based on chapter 30.10 from the C++17 standard
and a draft close to that version is
[Working Draft N4687](https://github.com/cplusplus/draft/raw/master/papers/n4687.pdf).
It is from after the standardization of C++17 but it contains the latest filesystem
interface changes compared to the
[Working Draft N4659](https://github.com/cplusplus/draft/raw/master/papers/n4659.pdf).

I want to thank the people working on improving C++, I really liked how the language
evolved with C++11 and the following standards. Keep on the good work!

Oh, and if you ask yourself, what `ghc` is standing for, it is simply
`gulraks helper classes`, yeah, I know, not very imaginative, but I wanted a
short namespace and I use it in some of my private classes (so it has nothing
to do with Haskell).

## Platforms

`ghc::filesystem` is developed on macOS but tested on Windows and Linux.
It should work on any of these with a C++11-capable compiler. I currently
don't have a BSD derivate besides macOS, so the preprocessor checks will
cry out if you try to use it there, but if there is demand, I can try to
help. Also there are some checks to hopefully better work on Android, but
as I currently don't test with the Android NDK, I wouldn't call it a
supported platform yet. All in all, I don't see it replacing `std::filesystem`
where full C++17 is available, it doesn't try to be a "better"
`std::filesystem`, just a drop-in if you can't use it (with the exception
of the UTF-8 preference on Windows).


Unit tests are currently run with:

* macOS 10.12: XCode 9.2 (clang-900.0.39.2), GCC 8.1.0, Clang 7.0.0
* Windows 10: Visual Studio 2017 15.8.5, MingW GCC 5.3
* Linux (Ubuntu): GCC (5.5, 6.5, 7.4, 8.1, 8.2), Clang (5.0, 6.0, 7.1, 8.0)


## Tests

The header comes with a set of unit-tests and uses [CMake](https://cmake.org/)
as a build tool and [Catch2](https://github.com/catchorg/Catch2) as test framework.

All tests agains this implementation should succeed, depending on your environment
it might be that there are some warnings, e.g. if you have no rights to create
Symlinks on Windows or at least the test thinks so, but these are just informative.

To build the tests from inside the project directory under macOS or Linux just:

```cpp
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

This generates `filesystem_test`, the binary that runs all tests.

If the default compiler is a GCC 8 or newer, or Clang 7 or newer, it
additionally tries to build a version of the test binary compiled against GCCs/Clangs
`std::filesystem` implementation, named `std_filesystem_test`
as an additional test of conformance. Ideally all tests should compile and
succeed with all filesystem implementations, but in reality, there are
some differences in behavior, sometimes due to room for interpretation in
in the standard, and there might be issues in these implementations too.


## Usage

### Downloads

The latest release version is [v1.1.4](https://github.com/gulrak/filesystem/tree/v1.1.4) and
source archives can be found [here](https://github.com/gulrak/filesystem/releases/tag/v1.1.4).

### Using it as Single-File-Header

As `ghc::filesystem` is at first a header-only library, it should be enough to copy the header
or the `include/ghc` directory into your project folder oder point your include path to this place and
simply include the `filesystem.hpp` header (or `ghc/filesystem.hpp` if you use the subdirectory).

Everything is in the namespace `ghc::filesystem`, so one way to use it only as
a fallback could be:

```cpp
#if defined(__cplusplus) && __cplusplus >= 201703L && defined(__has_include) && __has_include(<filesystem>)
#include <filesystem>
namespace fs = std::filesystem;
#else
#include <ghc/filesystem.hpp>
namespace fs = ghc::filesystem;
#endif
```

If you want to also use the `fstream` wrapper with `path` support as fallback,
you might use:

```cpp
#if defined(__cplusplus) && __cplusplus >= 201703L && defined(__has_include) && __has_include(<filesystem>)
#include <filesystem>
namespace fs {
using namespace std::filesystem;
using ifstream = std::ifstream;
using ofstream = std::ofstream;
using fstream = std::fstream;
}
#else
#include <ghc/filesystem.hpp>
namespace fs {
using namespace ghc::filesystem;
using ifstream = ghc::filesystem::ifstream;
using ofstream = ghc::filesystem::ofstream;
using fstream = ghc::filesystem::fstream;
} 
#endif
```

Now you have e.g. `fs::ofstream out(somePath);` and it is either the wrapper or
the C++17 `std::ofstream`.

**Note, that on MSVC this detection only works starting from version 15.7 on and when setting
the `/Zc:__cplusplus` compile switch, as the compiler allways reports `199711L`
without that switch ([see](https://blogs.msdn.microsoft.com/vcblog/2018/04/09/msvc-now-correctly-reports-__cplusplus/)).**

Be aware too, as a header-only library, it is not hiding the fact, that it
uses system includes, so they "pollute" your global namespace.

There is an additional header named `ghc/fs_std.hpp` that implements this
dynamic selection of a filesystem implementation, that you can include
instead of `ghc/filesystem.hpp` when you want std::filesystem where
available and ghc::filesystem where not.


### Using it as Forwarding-/Implementation-Header

Alternatively, starting from v1.1.0 `ghc::filesystem` can also be used by
including one of two additional wrapper headers. These allow to include
a forwarded version in most places (`ghc/fs_fwd.hpp`) while hiding the
implementation details in a single cpp that includes `ghc/fs_impl.hpp` to
implement the needed code. That way system includes are only visible from
inside the cpp, all other places are clean. 

Be aware, that it is currently not supported to hide the implementation
into a Windows-DLL, as a DLL interface with C++ standard templates in interfaces
is a different beast. If someone is willing to give it a try, I might integrate
a PR but currently working on that myself is not a priority.

If you use the forwarding/implementation approach, you can still use the dynamic
switching like this:

```cpp
#if defined(__cplusplus) && __cplusplus >= 201703L && defined(__has_include) && __has_include(<filesystem>)
#include <filesystem>
namespace fs {
using namespace std::filesystem;
using ifstream = std::ifstream;
using ofstream = std::ofstream;
using fstream = std::fstream;
}
#else
#include <ghc/fs-fwd.hpp>
namespace fs {
using namespace ghc::filesystem;
using ifstream = ghc::filesystem::ifstream;
using ofstream = ghc::filesystem::ofstream;
using fstream = ghc::filesystem::fstream;
} 
#endif
```

and in the implementation hiding cpp, you might use (before any include that includes `ghc/fs_fwd.hpp`
to take precedence:

```cpp
#if !(defined(__cplusplus) && __cplusplus >= 201703L && defined(__has_include) && __has_include(<filesystem>))
#include <ghc/fs_impl.hpp>
#endif
```

There are additional helper headers, named `ghc/fs_std_fwd.hpp` and `ghc/fs_std_impl.hpp`
that use this technique, so you can simply include them if you want to dynamically select
the filesystem implementation.


### Git Submodule and CMake

Starting from v1.1.0, it is possible to add `ghc::filesystem`
as a git submodule, add the directory to your `CMakeLists.txt` with
`add_subdirectory()` and then simply use `target_link_libraries(your-target ghc_filesystem)`
to ensure correct include path that allow `#include <ghc/filesystem.hpp>`
to work.

### Versioning

There is a version macro `GHC_FILESYSTEM_VERSION` defined in case future changes
might make it needed to react on the version, but I don't plan to break anything.
It's the version as decimal number `(major * 10000 + minor * 100 + patch)`.

**Note:** Starting from v1.0.2 only even path versions will be used for releases
and odd patch version will only be used for in between commits while working on
the next version.


## Documentation

There is almost no documentation in this release, as any `std::filesystem` documentation
would work, besides the few differences explained in the next section. So you might
head over to https://en.cppreference.com/w/cpp/filesystem for a description of
the components of this library.

The only additions to the standard are documented here:


### `ghc::filesystem::ifstream`, `ghc::filesystem::ofstream`, `ghc::filesystem::fstream`

These are simple wrappers around `std::ifstream`, `std::ofstream` and `std::fstream`.
They simply add an `open()` method and a constuctor with an `ghc::filesystem::path`
argument as the `fstream` variants in C++17 have them.

### `ghc::filesystem::u8arguments`

This is a helper class that currently checks for UTF-8 encoding on non-Windows platforms but on Windows it
fetches the command line arguments als Unicode strings from the OS with

```cpp
::CommandLineToArgvW(::GetCommandLineW(), &argc)
```

and then converts them to UTF-8, and replaces `argc` and `argv`. It is a guard-like
class that reverts its changes when going out of scope.

So basic usage is:

```cpp
namespace fs = ghc::filesystem;

int main(int argc, char* argv[])
{
    fs::u8arguments u8guard(argc, argv);
    if(u8guard.valid()) {
        std::cerr << "Bad encoding, needs UTF-8." << std::endl;
        exit(EXIT_FAILURE);
    }

    // now use argc/argv as usual, they have utf-8 enconding on windows
    // ...

    return 0;
}
```

That way `argv` is UTF-8 encoded as long as the scope from `main` is valid.

**Note:** On macOS, while debugging under Xcode the code currently will return
`false` as Xcode starts the application with `US-ASCII` as encoding, no matter what
encoding is actually used and even setting `LC_ALL` in the product scheme doesn't
change anything. I still need to investigate this.


## Differences

As this implementation is based on existing code from my private helper
classes, it derived some constraints of it, leading to some differences
between this and the standard C++17 API.


### LWG Defects

This implementation has switchable behavior for the LWG defects
[#2682](https://wg21.cmeerw.net/lwg/issue2682),
[#2935](http://www.open-std.org/jtc1/sc22/wg21/docs/lwg-defects.html#2935) and
[#2937](http://www.open-std.org/jtc1/sc22/wg21/docs/lwg-defects.html#2937).
The currently selected behavior is following
[#2682](https://wg21.cmeerw.net/lwg/issue2682),
[#2937](http://www.open-std.org/jtc1/sc22/wg21/docs/lwg-defects.html#2937) but
not following [#2935](http://www.open-std.org/jtc1/sc22/wg21/docs/lwg-defects.html#2935),
as I feel it is a bug to report no error on a `create_directory()` or `create_directories()`
where a regular file of the same name prohibits the creation of a directory and forces
the user of those functions to double-check via `fs::is_directory` if it really worked.
The more intuitive approach to directory creation of treating a file with that name as an
error is also advocated by the newer paper
[WG21 P1164R0](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1164r0.pdf), the revison
P1161R1 was agreed upon on Kona 2019 meeting [see merge](https://github.com/cplusplus/draft/issues/2703)
and GCC by now switched to following its proposal
([GCC #86910](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=86910)). 

### Not Implemented

Besides this still being work-in-progress, there are a few cases where
there will be no implementation in the close future:

```cpp
// methods in ghc::filesystem::path:
path& operator+=(basic_string_view<value_type> x);
int compare(basic_string_view<value_type> s) const;
```

These are not implemented under C++11 and C++14, as there is no
`std::basic_string_view` available and I did want to keep this
implementation self-contained and not write a full C++17-upgrade for
C++11/14. Starting with v1.1.0 these are supported when compiling
ghc::filesystem under C++17.


### Differences in API

```cpp
filesystem::path::string_type
filesystem::path::value_type
```

In Windows, an implementation should use `std::wstring` and `wchar_t` as types used
for the native representation, but as I'm a big fan of the
["UTF-8 Everywhere" philosophy](https://utf8everywhere.org/), I decided
agains it for now. If you need to call some Windows API, use the W-variant
with the `path::wstring()` member
(e.g. `GetFileAttributesW(p.wstring().c_str())`). This gives you the
Unicode variant independant of the `UNICODE` macro and makes sharing code
between Windows, Linux and macOS easier.

```cpp
const path::string_type& path::native() const /*noexcept*/;
const path::value_type *path::c_str() const /*noexcept*/;
```

These two can not be `noexcept` with the current implementation. This due
to the fact, that internally path is working on the generic path version
only, and the getters need to do a conversion to native path format.

```cpp
const path::string_type& path::generic_string() const;
```

This returns a const reference, instead of a value, because it can. This
implementation uses the generic representation for internal workings, so
it's "free" to return that.


### Differences in Behavior

#### fs.path ([ref](https://en.cppreference.com/w/cpp/filesystem/path))

As the complete inner mechanics of this implementation `fs::path` are working
on the generic format, it is the internal representation. So creating any mixed
slash `fs::path` object under Windows (e.g. with `"C:\foo/bar"`) will lead to a
unified path with `"C:\foo\bar"` via `native()` and `"C:/foo/bar"` via
`generic_string()` API.

Additionally this implementation follows the standards suggestion to handle
posix paths of the form `"//host/path"` and USC path on windows also as having
a root-name (e.g. `"//host"`). The GCC implementation didn't choose to do that
while testing on Ubuntu 18.04 and macOS with GCC 8.1.0 or Clang 7.0.0. This difference
will show as warnings under std::filesystem. This leads to a change in the
algorithm described in the standard for `operator/=(path& p)` where any path
`p` with `p.is_absolute()` will degrade to an assignment, while this implementation
has the exception where `*this == *this.root_name()` and `p == preferred_seperator`
a normal append will be done, to allow:

```cpp
fs::path p1 = "//host/foo/bar/file.txt";
fs::path p2;
for (auto p : p1) p2 /= p;
ASSERT(p1 == p2);
```

For all non-host-leading paths the behaviour will match the one described by
the standard.

#### fs.op.copy ([ref](https://en.cppreference.com/w/cpp/filesystem/copy))

Then there is `fs::copy`. The tests in the suite fail partially with C++17 `std::filesystem`
on GCC/Clang. They complain about a copy call with `fs::copy_options::recursive` combined
with `fs::copy_options::create_symlinks` or `fs::copy_options::create_hard_links` if the
source is a directory. There is nothing in the standard that forbids this combination
and it is the only way to deep-copy a tree while only create links for the files.
There is [LWG #2682](https://wg21.cmeerw.net/lwg/issue2682) that supports this
interpretation, but the issue ignores the usefulness of the combination with recursive
and part of the justification for the proposed solution is "we did it so for almost two years".
But this makes `fs::copy` with `fs::copy_options::create_symlinks` or `fs::copy_options::create_hard_links`
just a more complicated syntax for the `fs::create_symlink` or `fs::create_hardlink` operation
and I don't want to believe, that this was the intention of the original writing.
As there is another issue related to copy, with a different take on the description.

*Note:* With v1.1.2 I decided to integrate a behavior switch for this and make the LWG #2682
the default.

## Open Issues

### General Known Issues

There are still some methods that break the `noexcept` clause, some
are related to LWG defects, some are due to my implementation. I
work on fixing the later ones, and might in cases where there is no
way of implementing the feature without risk of an exception, break
conformance and remove the `noexcept`.

### Windows

#### Symbolic Links on Windows

As symbolic links on Windows, while being supported more or less since
Windows Vista (with some strict security constraints) and fully since some earlier
build of Windows 10, when "Developer Mode" is activated, are at time of writing
(2018) rarely used, still they are supported with this implementation.

#### Permissions

The Windows ACL permission feature translates badly to the POSIX permission
bit mask used in the interface of C++17 filesystem. The permissions returned
in the `file_status` are therefore currently synthesized for the `user`-level
and copied to the `group`- and `other`-level. There is still some potential
for more interaction with the Windows permission system, but currently setting
or reading permissions with this implementation will most certainly not lead
to the expected behavior.


## Release Notes

### [v1.1.4](https://github.com/gulrak/filesystem/releases/tag/v1.1.4)

* Additional Bugfix for ([#12](https://github.com/gulrak/filesystem/issues/12)),
  error in old unified `readdir/readdir_r` code of `fs::directory_iterator`;
  as `readdir_r` is now depricated, I decided to drop it and the resulting
  code is much easier, shorter and due to more refactoring faster
* Fix for crashing unit tests against MSVC C++17 std::filesystem
* Travis-CI now additionally test with Xcode 10.2 on macOS
* Some minor refactorings

### [v1.1.2](https://github.com/gulrak/filesystem/releases/tag/v1.1.2)

* Bugfix for ([#11](https://github.com/gulrak/filesystem/issues/11)),
  `fs::path::lexically_normal()` had some issues with `".."`-sequences.
* Bugfix for ([#12](https://github.com/gulrak/filesystem/issues/12)),
  `fs::recursive_directory_iterator` could run into endless loops,
  the methods depth() and pop() had issues and the copy behaviour and
  `input_iterator_tag` conformance was broken, added tests
* Restructured some CMake code into a macro to ease the support for
  C++17 std::filesystem builds of tests and examples for interoperability
  checks.
* Some fixes on Windows tests to ease interoperability test runs.
* Reduced noise on `fs::weakly_canonical()` tests against `std::fs`
* Added simple `du` example showing the `recursive_directory_iterator`
  used to add the sizes of files in a directory tree.
* Added error checking in `fs::file_time_type` test helpers
* `fs::copy()` now conforms LWG #2682, disallowing the use of
  `copy_option::create_symlinks' to be used on directories

### [v1.1.0](https://github.com/gulrak/filesystem/releases/tag/v1.1.0)

* Restructuring of the project directory. The header files are now using
  `hpp` as extension to be marked as c++ and they where moved to
  `include/ghc/` to be able to include by `<ghc/filesystem.hpp>` as the
  former include name might have been to generic and conflict with other
  files.
* Better CMake support: `ghc::filesystem` now can be used as a submodul
  and added with `add_subdirectory` and will export itself as `ghc_filesystem`
  target. To use it, only `target_link_libraries(your-target ghc_filesystem)`
  is needed and the include directories will be set so `#include <ghc/filesystem.hpp>`
  will be a valid directive.
  Still you can simply only add the header file to you project and include it
  from there.
* Enhancement ([#10](https://github.com/gulrak/filesystem/issues/10)),
  support for separation of implementation and forwarded api: Two
  additional simple includes are added, that can be used to forward
  `ghc::filesystem` declarations (`fs_fwd.hpp`) and to wrap the
  implementation into a single cpp (`fs_impl.hpp`)
* The `std::basic_string_view` variants of the `fs::path` api are
  now supported when compiling with C++17. 
* Added CI integration for Travis-CI and Appveyor.
* Fixed MingW compilation issues.
* Added long filename support for Windows.

### [v1.0.10](https://github.com/gulrak/filesystem/releases/tag/v1.0.10)

* Bugfix for ([#9](https://github.com/gulrak/filesystem/issues/9)), added
  missing return statement to `ghc::filesystem::path::generic_string()`
* Added checks to hopefully better compile against Android NDK. There where
  no tests run yet, so feedback is needed to actually call this supported.
* `filesystem.h` was renamed `filesystem.hpp` to better reflect that it is
  a c++ language header.

### [v1.0.8](https://github.com/gulrak/filesystem/releases/tag/v1.0.8)

* Bugfix for ([#6](https://github.com/gulrak/filesystem/issues/6)), where
  `ghc::filesystem::remove()` and `ghc::filesystem::remove_all()` both are
  now able to remove a single file and both will not raise an error if the
  path doesn't exist.
* Merged pull request ([#7](https://github.com/gulrak/filesystem/pull/7)),
  a typo leading to setting error code instead of comparing it in
  `ghc::filesystem::remove()` under Windows.
* Bugfix for (([#8](https://github.com/gulrak/filesystem/issues/8)), the
  Windows version of `ghc::filesystem::directory_iterator` now releases
  resources when reaching `end()` like the POSIX one does.


### [v1.0.6](https://github.com/gulrak/filesystem/releases/tag/v1.0.6)

* Bugfix for ([#4](https://github.com/gulrak/filesystem/issues/4)), missing error_code
  propagation in `ghc::filesystem::copy()` and `ghc::filesystem::remove_all` fixed.
* Bugfix for ([#5](https://github.com/gulrak/filesystem/issues/5)), added missing std
  namespace in `ghc::filesystem::recursive_directory_iterator::difference_type`.

### [v1.0.4](https://github.com/gulrak/filesystem/releases/tag/v1.0.4)

* Bugfix for ([#3](https://github.com/gulrak/filesystem/issues/3)), fixed missing inlines
  and added test to ensure including into multiple implementation files works as expected.
* Building tests with `-Wall -Wextra -Werror` and fixed resulting issues.

### [v1.0.2](https://github.com/gulrak/filesystem/releases/tag/v1.0.2)

* Updated catch2 to v2.4.0.
* Refactored `fs.op.permissions` test to work with all tested `std::filesystem`
  implementations (gcc, clang, msvc++).
* Added helper class `ghc::filesystem::u8arguments` as `argv` converter, to
  help follow the UTF-8 path on windows. Simply instantiate it with `argc` and
  `argv` and it will fetch the Unicode version of the command line and convert
  it to UTF-8. The destructor reverts the change.
* Added `examples` folder with hopefully some usefull example usage. Examples are
  tested (and build) with `ghc::filesystem` and C++17 `std::filesystem` when
  available.
* Starting with this version, only even patch level versions will be tagged and
  odd patch levels mark in-between non-stable wip states.
* Tests can now also be run against MS version of std::filesystem for comparison.
* Added missing `fstream` include.
* Removed non-conforming C99 `timespec`/`timeval` usage.
* Fixed some integer type mismatches that could lead to warnings.
* Fixed `chrono` conversion issues in test and example on clang 7.0.0.

### [v1.0.1](https://github.com/gulrak/filesystem/releases/tag/v1.0.1)

* Bugfix: `ghc::filesystem::canonical` now sees empty path as non-existant and reports
  an error. Due to this `ghc::filesystem::weakly_canonical` now returns relative
  paths for non-existant argument paths. ([#1](https://github.com/gulrak/filesystem/issues/1))
* Bugfix: `ghc::filesystem::remove_all` now also counts directories removed ([#2](https://github.com/gulrak/filesystem/issues/2))
* Bugfix: `recursive_directory_iterator` tests didn't respect equality domain issues
  and dereferencable constraints, leading to fails on `std::filesystem` tests.
* Bugfix: Some `noexcept` tagged methods and functions could indirectly throw exceptions
  due to UFT-8 decoding issues.
* `std_filesystem_test` is now also generated if LLVM/clang 7.0.0 is found.


### [v1.0.0](https://github.com/gulrak/filesystem/releases/tag/v1.0.0)

This was the first public release version. It implements the full range of
C++17 std::filesystem, as far as possible without other C++17 dependencies.

