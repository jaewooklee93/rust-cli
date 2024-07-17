# 명령줄 인수 분석

우리의 CLI 도구의 일반적인 호출은 다음과 같습니다.

```console
$ grrs foobar test.txt
```

우리 프로그램이 `test.txt`를 살펴보고 `foobar`를 포함하는 줄을 출력하도록 예상합니다.
하지만 이 두 값은 어떻게 얻을까요?

프로그램 이름 뒤에 있는 텍스트는 종종 "명령줄 인수" 또는 "명령줄 플래그"라고 불립니다.
(특히 `--this`와 같은 모양일 때)
실제로 운영 체제는 일반적으로 이를 문자열 목록으로 표현합니다.
대략적으로 말하면 공백으로 구분됩니다.

이러한 인수에 대해 생각할 수 있는 방법은 많으며,
더 쉽게 처리할 수 있도록 분석하는 방법도 많습니다.
프로그램 사용자에게 어떤 인수를 제공해야 하는지
그리고 어떤 형식으로 기대되는지도 알려야 합니다.

## 인수 가져오기

표준 라이브러리에는 주어진 인수의 [이터레이터](https://doc.rust-lang.org/1.39.0/std/iter/index.html)를 제공하는 `std::env::args()` 함수가 있습니다.
첫 번째 항목(인덱스 `0`에서)은 프로그램이 호출된 이름(예: `grrs`)이며,
그 다음은 사용자가 나중에 입력한 것입니다.

[`std::env::args()`]: https://doc.rust-lang.org/1.39.0/std/env/fn.args.html
[이터레이터]: https://doc.rust-lang.org/1.39.0/std/iter/index.html

이렇게 원본 인수를 가져오는 것은 매우 쉽습니다( `src/main.rs` 파일에서):

```rust,ignore
{{#include cli-args-vars.rs}}
```

 `cargo run` 을 사용하여 실행할 수 있습니다.
인수는 `--` 다음에 작성하여 전달합니다.

```console
$ cargo run -- some-pattern some-file
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/grrs some-pattern some-file`
pattern: "some-pattern", path: "some-file"
```

## 명령줄 인자를 데이터 유형으로 생각하기

단순히 텍스트로만 생각하는 것보다,
명령줄 인자를 프로그램의 입력을 나타내는
맞춤형 데이터 유형으로 생각하는 것이 유리합니다.

`grrs foobar test.txt`를 살펴보세요:
두 개의 인자가 있습니다.
첫 번째는 `pattern` (찾을 문자열)이고,
두 번째는 `path` (찾을 파일 경로)입니다.

이에 대해 더 무엇을 말할 수 있을까요?
첫째, 두 개 모두 필수입니다.
기본값에 대해서는 아직 이야기하지 않았으므로,
사용자는 항상 두 값을 제공해야 합니다.
둘째, 유형에 대해 약간 말할 수 있습니다.
`pattern`은 문자열로 기대되며,
두 번째 인자는 파일 경로로 기대됩니다.

Rust에서는 처리하는 데이터를 중심으로 프로그램을 구조화하는 것이 일반적이므로,
이러한 명령줄 인자에 대한 시각은 매우 잘 맞습니다. `src/main.rs` 파일에서 `fn main() {` 전에 다음과 같이 시작해 보겠습니다.

```rust,ignore
{{#include cli-args-struct.rs:1:4}}
```

이것은 두 개의 필드를 가진 새로운 구조 (`struct`)를 정의합니다. `pattern` 과 `path` 에 데이터를 저장합니다.

[`struct`]: https://doc.rust-lang.org/1.39.0/book/ch05-00-structs.html

<aside>

**참고:**
[`PathBuf`]는 파일 시스템 경로를 위한 [`String`]과 유사하지만, 범용적으로 작동합니다.

[`PathBuf`]: https://doc.rust-lang.org/1.39.0/std/path/struct.PathBuf.html
[`String`]: https://doc.rust-lang.org/1.39.0/std/string/struct.String.html

</aside>

이제 우리는 프로그램이 실제로 받은 인수를 이 형태로 가져오는 방법이 필요합니다.
한 가지 옵션은 운영 체제에서 받은 문자열 목록을 수동으로 분석하고 구조를 직접 만드는 것입니다.
다음과 같이 보일 것입니다.

```rust,ignore
{{#include cli-args-struct.rs:6:16}}
```

이것은 작동하지만, 매우 편리하지 않습니다.
`--pattern="foo"` 또는 `--pattern "foo"`와 같은 요구 사항을 지원하는 방법은 무엇입니까?
`--help`를 구현하는 방법은 무엇입니까?

## Clap을 사용하여 CLI 인수 분석하기

훨씬 더 좋은 방법은 많은 라이브러리 중 하나를 사용하는 것입니다.
명령줄 인수를 분석하는 가장 인기 있는 라이브러리는 [`clap`]이라고 합니다.

`clap`는 서브 명령어, [셸 완성], 훌륭한 도움 메시지 등 기대하는 모든 기능을 제공합니다.

[`clap`]: https://docs.rs/clap/
[셸 완성]: https://docs.rs/clap_complete/

먼저 `Cargo.toml` 파일의 `[dependencies]` 섹션에 `clap = { version = "4.0", features = ["derive"] }`를 추가하여 `clap`를 가져옵니다.

이제 코드에서 `use clap::Parser;`를 작성하고 `struct Cli` 바로 위에 `#[derive(Parser)]`를 추가할 수 있습니다.
동시에 문서 코멘트도 작성해 보겠습니다.

`src/main.rs` 파일에서 `fn main() {` 앞에 다음과 같이 작성됩니다.

```rust,ignore
{{#include cli-args-clap.rs:1:10}}
```

<aside class="node">

**참고:**
필드에 추가할 수 있는 사용자 정의 속성이 많습니다.
예를 들어,
`-o` 또는 `--output` 다음에 사용되는 필드를 원한다면,
`#[arg(short = 'o', long = "output")]`를 추가합니다.
더 자세한 내용은 [clap 문서][`clap` ]를 참조하십시오.

</aside>

`Cli` 구조체 바로 아래에 템플릿에 `main` 함수가 포함되어 있습니다.
프로그램이 시작될 때 이 함수가 호출됩니다:

```rust,ignore
{{#include cli-args-clap.rs:12:16}}
```

이것은 인수를 우리의 `Cli` 구조체로 파싱하려고 시도합니다.

하지만 만약 실패하면 어떨까요?
이 접근 방식의 아름다움은 다음과 같습니다.
Clap은 기대하는 필드와 기대되는 형식을 알고 있습니다.
자동으로 멋진 `--help` 메시지를 생성할 수 있으며,
`--putput`라고 작성했을 때 `--output`를 전달하라고 제안하는 훌륭한 오류 메시지를 제공할 수 있습니다.

<aside class="note">

**참고:**
`parse` 메서드는 `main` 함수에서 사용하도록 설계되었습니다.
실패하면 오류 또는 도움 메시지를 출력하고 프로그램을 즉시 종료합니다.
다른 곳에서 사용하지 마세요!

</aside>

## 마무리

 코드가 이제 다음과 같이 보일 것입니다.

```rust,ignore
{{#include cli-args-clap.rs}}
```

아무런 인수 없이 실행하면:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 10.16s
     Running `target/debug/grrs`
error: The following required arguments were not provided:
    <pattern>
    <path>

USAGE:
    grrs <pattern> <path>

For more information try --help
```

인수를 전달하여 실행하는 경우:

```console
$ cargo run -- some-pattern some-file
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/grrs some-pattern some-file`
pattern: "some-pattern", path: "some-file"
```

출력은 프로그램이 `Cli` 구조체로 인수를 성공적으로 분석했다는 것을 보여줍니다.
