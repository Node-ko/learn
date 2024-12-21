---
title: 다양한 파일시스템에서 작업하는 법
layout: learn
---

> ❗️ _번역 날짜: 2024년 12월 21일_ <br />
> 공식 문서 원문은 아래를 참고하세요.<br /> > [How to Work with Different Filesystems](https://nodejs.org/en/learn/manipulating-files/working-with-different-filesystems)

# 다양한 파일시스템에서 작업하는 법

Node.js 는 파일 시스템의 많은 기능을 제공합니다. 하지만 모든 파일 시스템이 동일하지는 않습니다.

다양한 파일 시스템을 다룰 때 코드를 간단하고 안전하게 유지하기 위한 권장 모범 사례는 다음과 같습니다.

## 파일 시스템 동작 (Filesystem behavior)

파일 시스템을 사용하기 전에, 해당 파일 시스템이 어떻게 동작하는지 알아야 합니다. 파일 시스템마다 동작 방식이 다르고, 지원하는 기능도 다를 수 있습니다.

예를 들면

- 대소문자 구분 여부
- 대소문자 비구분 및 대소문자 보존 여부
- 유니코드 형식 보존 여부
- 타임스탬프의 해상도
- 확장 속성
- 인덱스 노드(inodes)
- 유닉스 권한(permissions)
- 대체 데이터 스트림 등

`process.platform` 값을 통해 파일 시스템의 동작을 추론하는 것은 위험할 수 있습니다. 예를 들면, Darwin(Mac OS) 에서 실행된다고 해서 반드시 대소문자를 구분하지 않는 파일시스템(HFS+)를 사용한다고 단정지을 수 없습니다. 사용자가 대소문자를 구분하는 파일시스템(HFSX)를 사용하고 있을 수도 있기 때문입니다.

또는, Linux에서 실행된다고 해서 유닉스 권한과 인덱스 노드를 지원하는 파일시스템을 사용한다고 단정지을 수 없습니다. 외부 드라이브, USB 또는 네트워크 드라이브처럼 해당 기능을 지원하지 않는 파일 시스템일 수도 있기 때문입니다.

운영체제는 파일 시스템 동작을 추론하기 어렵게 만들 수 있지만, 모든 것이 불가능한 것은 아닙니다. 모든 파일 시스템과 동작 목록을 유지하는 대신(항상 불완전할 수 있음), 파일 시스템의 실제 동작을 프로브(probe)하여 확인할 수 있습니다. 기능이 존재하는 지 여부는 프로브하기 쉽습니다. 또한
The presence or absence of certain features which are
easy to probe, are often enough to infer the behavior of other features which
are more difficult to probe.

Remember that some users may have different filesystems mounted at various paths
in the working tree.

## Avoid a Lowest Common Denominator Approach

You might be tempted to make your program act like a lowest common denominator
filesystem, by normalizing all filenames to uppercase, normalizing all filenames
to NFC Unicode form, and normalizing all file timestamps to say 1-second
resolution. This would be the lowest common denominator approach.

Do not do this. You would only be able to interact safely with a filesystem
which has the exact same lowest common denominator characteristics in every
respect. You would be unable to work with more advanced filesystems in the way
that users expect, and you would run into filename or timestamp collisions. You
would most certainly lose and corrupt user data through a series of complicated
dependent events, and you would create bugs that would be difficult if not
impossible to solve.

What happens when you later need to support a filesystem that only has 2-second
or 24-hour timestamp resolution? What happens when the Unicode standard advances
to include a slightly different normalization algorithm (as has happened in the
past)?

A lowest common denominator approach would tend to try to create a portable
program by using only "portable" system calls. This leads to programs that are
leaky and not in fact portable.

## Adopt a Superset Approach

Make the best use of each platform you support by adopting a superset approach.
For example, a portable backup program should sync btimes (the created time of a
file or folder) correctly between Windows systems, and should not destroy or
alter btimes, even though btimes are not supported on Linux systems. The same
portable backup program should sync Unix permissions correctly between Linux
systems, and should not destroy or alter Unix permissions, even though Unix
permissions are not supported on Windows systems.

Handle different filesystems by making your program act like a more advanced
filesystem. Support a superset of all possible features: case-sensitivity,
case-preservation, Unicode form sensitivity, Unicode form preservation, Unix
permissions, high-resolution nanosecond timestamps, extended attributes etc.

Once you have case-preservation in your program, you can always implement
case-insensitivity if you need to interact with a case-insensitive filesystem.
But if you forego case-preservation in your program, you cannot interact safely
with a case-preserving filesystem. The same is true for Unicode form
preservation and timestamp resolution preservation.

If a filesystem provides you with a filename in a mix of lowercase and
uppercase, then keep the filename in the exact case given. If a filesystem
provides you with a filename in mixed Unicode form or NFC or NFD (or NFKC or
NFKD), then keep the filename in the exact byte sequence given. If a filesystem
provides you with a millisecond timestamp, then keep the timestamp in
millisecond resolution.

When you work with a lesser filesystem, you can always downsample appropriately,
with comparison functions as required by the behavior of the filesystem on which
your program is running. If you know that the filesystem does not support Unix
permissions, then you should not expect to read the same Unix permissions you
write. If you know that the filesystem does not preserve case, then you should
be prepared to see `ABC` in a directory listing when your program creates `abc`.
But if you know that the filesystem does preserve case, then you should consider
`ABC` to be a different filename to `abc`, when detecting file renames or if the
filesystem is case-sensitive.

## Case Preservation

You may create a directory called `test/abc` and be surprised to see sometimes
that `fs.readdir('test')` returns `['ABC']`. This is not a bug in Node. Node
returns the filename as the filesystem stores it, and not all filesystems
support case-preservation. Some filesystems convert all filenames to uppercase
(or lowercase).

## Unicode Form Preservation

_Case preservation and Unicode form preservation are similar concepts. To
understand why Unicode form should be preserved , make sure that you first
understand why case should be preserved. Unicode form preservation is just as
simple when understood correctly._

Unicode can encode the same characters using several different byte sequences.
Several strings may look the same, but have different byte sequences. When
working with UTF-8 strings, be careful that your expectations are in line with
how Unicode works. Just as you would not expect all UTF-8 characters to encode
to a single byte, you should not expect several UTF-8 strings that look the same
to the human eye to have the same byte representation. This may be an
expectation that you can have of ASCII, but not of UTF-8.

You may create a directory called `test/café` (NFC Unicode form with byte
sequence `<63 61 66 c3 a9>` and `string.length === 5`) and be surprised to see
sometimes that `fs.readdir('test')` returns `['café']` (NFD Unicode form with
byte sequence `<63 61 66 65 cc 81>` and `string.length === 6`). This is not a
bug in Node. Node.js returns the filename as the filesystem stores it, and not
all filesystems support Unicode form preservation.

HFS+, for example, will normalize all filenames to a form almost always the same
as NFD form. Do not expect HFS+ to behave the same as NTFS or EXT4 and
vice-versa. Do not try to change data permanently through normalization as a
leaky abstraction to paper over Unicode differences between filesystems. This
would create problems without solving any. Rather, preserve Unicode form and use
normalization as a comparison function only.

## Unicode Form Insensitivity

Unicode form insensitivity and Unicode form preservation are two different
filesystem behaviors often mistaken for each other. Just as case-insensitivity
has sometimes been incorrectly implemented by permanently normalizing filenames
to uppercase when storing and transmitting filenames, so Unicode form
insensitivity has sometimes been incorrectly implemented by permanently
normalizing filenames to a certain Unicode form (NFD in the case of HFS+) when
storing and transmitting filenames. It is possible and much better to implement
Unicode form insensitivity without sacrificing Unicode form preservation, by
using Unicode normalization for comparison only.

## Comparing Different Unicode Forms

Node.js provides `string.normalize('NFC' / 'NFD')` which you can use to normalize a
UTF-8 string to either NFC or NFD. You should never store the output from this
function but only use it as part of a comparison function to test whether two
UTF-8 strings would look the same to the user.

You can use `string1.normalize('NFC') === string2.normalize('NFC')` or
`string1.normalize('NFD') === string2.normalize('NFD')` as your comparison
function. Which form you use does not matter.

Normalization is fast but you may want to use a cache as input to your
comparison function to avoid normalizing the same string many times over. If the
string is not present in the cache then normalize it and cache it. Be careful
not to store or persist the cache, use it only as a cache.

Note that using `normalize()` requires that your version of Node.js include ICU
(otherwise `normalize()` will just return the original string). If you download
the latest version of Node.js from the website then it will include ICU.

## Timestamp Resolution

You may set the `mtime` (the modified time) of a file to `1444291759414`
(millisecond resolution) and be surprised to see sometimes that `fs.stat`
returns the new mtime as `1444291759000` (1-second resolution) or
`1444291758000` (2-second resolution). This is not a bug in Node. Node.js returns
the timestamp as the filesystem stores it, and not all filesystems support
nanosecond, millisecond or 1-second timestamp resolution. Some filesystems even
have very coarse resolution for the atime timestamp in particular, e.g. 24 hours
for some FAT filesystems.

## Do Not Corrupt Filenames and Timestamps Through Normalization

Filenames and timestamps are user data. Just as you would never automatically
rewrite user file data to uppercase the data or normalize `CRLF` to `LF`
line-endings, so you should never change, interfere or corrupt filenames or
timestamps through case / Unicode form / timestamp normalization. Normalization
should only ever be used for comparison, never for altering data.

Normalization is effectively a lossy hash code. You can use it to test for
certain kinds of equivalence (e.g. do several strings look the same even though
they have different byte sequences) but you can never use it as a substitute for
the actual data. Your program should pass on filename and timestamp data as is.

Your program can create new data in NFC (or in any combination of Unicode form
it prefers) or with a lowercase or uppercase filename, or with a 2-second
resolution timestamp, but your program should not corrupt existing user data by
imposing case / Unicode form / timestamp normalization. Rather, adopt a superset
approach and preserve case, Unicode form and timestamp resolution in your
program. That way, you will be able to interact safely with filesystems which do
the same.

## Use Normalization Comparison Functions Appropriately

Make sure that you use case / Unicode form / timestamp comparison functions
appropriately. Do not use a case-insensitive filename comparison function if you
are working on a case-sensitive filesystem. Do not use a Unicode form
insensitive comparison function if you are working on a Unicode form sensitive
filesystem (e.g. NTFS and most Linux filesystems which preserve both NFC and NFD
or mixed Unicode forms). Do not compare timestamps at 2-second resolution if you
are working on a nanosecond timestamp resolution filesystem.

## Be Prepared for Slight Differences in Comparison Functions

Be careful that your comparison functions match those of the filesystem (or
probe the filesystem if possible to see how it would actually compare).
Case-insensitivity for example is more complex than a simple `toLowerCase()`
comparison. In fact, `toUpperCase()` is usually better than `toLowerCase()`
(since it handles certain foreign language characters differently). But better
still would be to probe the filesystem since every filesystem has its own case
comparison table baked in.

As an example, Apple's HFS+ normalizes filenames to NFD form but this NFD form
is actually an older version of the current NFD form and may sometimes be
slightly different from the latest Unicode standard's NFD form. Do not expect
HFS+ NFD to be exactly the same as Unicode NFD all the time.
