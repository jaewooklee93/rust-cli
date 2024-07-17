# 출력

## "Hello World" 출력하기

```rust
println!("Hello World");
```

잘했어요.
좋아요, 다음 주제로 넘어가요.

## `println!` 사용하기

`println!` 매크로를 사용하면 좋아하는 것들을 거의 모두 출력할 수 있습니다.
이 매크로는 꽤 놀라운 기능을 가지고 있지만,
특별한 문법을 가지고 있습니다.
첫 번째 매개변수로 문자열 리터럴을 작성해야 하며,
이 문자열 리터럴에는 다음과 같은 매개변수 값으로 채워질 홀더가 포함되어 있어야 합니다.

예를 들어:

```rust
let x = 42;
println!("My lucky number is {}.", x);
```

출력될 내용

```console
My lucky number is 42.
```

위 문자열에서 중괄호 (`{}`)는 이러한 占位符 중 하나입니다.

이는 주어진 값을 인간이 읽기 쉽게 출력하려는 기본 占位符 유형입니다.
숫자와 문자열의 경우 이 방법이 매우 잘 작동하지만,
모든 유형에 대해서는 그렇지 않습니다.
이 때문에 `{:?}`와 같이 중괄호를 채워서 "디버그 표현"을 얻을 수 있습니다.

예를 들어,

```rust
let xs = vec![1, 2, 3];
println!("The list is: {:?}", xs);
```

출력될 내용

```console
The list is: [1, 2, 3]
```

디버깅 및 로그 기록을 위해 자신의 데이터 유형을 인쇄하려면,
대부분의 경우 정의 위에 `#[derive(Debug)]`를 추가할 수 있습니다.

<aside>

**참고:**
"사용자 친화적인" 인쇄는 [`Display`] 트레이트를 사용하여 수행되며,
디버그 출력 (개발자를 위한 가독성이 있는)은 [`Debug`] 트레이트를 사용합니다.
`println!`에서 사용할 수 있는 구문에 대한 자세한 내용은 [ `std::fmt` 모듈 문서][std::fmt]를 참조하십시오.

[`Display`]: https://doc.rust-lang.org/1.39.0/std/fmt/trait.Display.html
[`Debug`]: https://doc.rust-lang.org/1.39.0/std/fmt/trait.Debug.html
[std::fmt]: https://doc.rust-lang.org/1.39.0/std/fmt/index.html

</aside>

## 오류 출력

오류 출력은 `stderr`를 통해 수행되어야 하며,
이를 통해 사용자와 다른 도구가 출력을 파일이나 다른 도구로 파이프할 수 있도록 돕습니다.

<aside>

**참고:**
대부분의 운영 체제에서 프로그램은 `stdout`와 `stderr`라는 두 개의 출력 스트림으로 쓰기 가능합니다.
`stdout`는 프로그램의 실제 출력을 위해 사용되며,
`stderr`는 오류 및 기타 메시지를 `stdout`에서 분리하여 표시할 수 있도록 합니다.
이렇게 하면 출력을 파일로 저장하거나 다른 프로그램으로 파이프할 수 있으면서도 오류가 사용자에게 표시됩니다.

</aside>

Rust에서는 `println!`과 `eprintln!`을 사용하여 이를 달성합니다.
`println!`은 `stdout`로 출력하고,
`eprintln!`은 `stderr`로 출력합니다.

```rust
println!("This is information");
eprintln!("This is an error! :(");
```

<aside>

**주의**: 이스케이프 코드를 출력하는 것은 위험할 수 있습니다.
터미널 상태를 이상하게 만들 수 있습니다.
수동으로 출력할 때 항상 주의하십시오!

[이스케이프 코드]: https://ko.wikipedia.org/wiki/ANSI_escape_code

가능하다면 `ansi_term`과 같은 crate를 사용하여 원본 이스케이프 코드를 처리하는 것이 좋습니다.
그렇게 하면 (당신과 사용자 모두의) 삶이 더 쉬워집니다.

</aside>

## 출력 성능에 대한 주의 사항

터미널에 출력하는 것은 놀랍게도 느리다!
`println!`과 같은 함수를 루프에서 호출하면,
그렇지 않은 경우에도 빠른 프로그램에서 쉽게 병목 현상이 발생할 수 있습니다.
이를 가속화하려면,
두 가지 작업을 수행할 수 있습니다.

첫째,
실제로 "터미널에 플러시"하는 쓰기 횟수를 줄일 수 있습니다.
`println!`은 시스템에 매번 터미널에 플러시하도록 알립니다.
bởi vì 각 새 줄을 출력하는 것이 일반적입니다.
필요하지 않으면,
`stdout` 핸들을 [`BufWriter`]
에 감싸면 됩니다.
이는 기본적으로 8 kB까지 버퍼링합니다.
(여전히 `BufWriter`에 `.flush()`를 호출하여 즉시 출력할 수 있습니다.)

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = io::BufWriter::new(stdout); // optional: wrap that handle in a buffer
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

두 번째로, `stdout` (또는 `stderr`) 에 대한 잠금을 획득하고 `writeln!` 을 사용하여 직접 출력하는 것이 도움이 됩니다.
이렇게 하면 시스템이 `stdout` 를 반복적으로 잠금 해제하는 것을 방지할 수 있습니다.

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = stdout.lock(); // acquire a lock on it
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

두 가지 접근 방식을 결합할 수도 있습니다.

[`BufWriter`]: https://doc.rust-lang.org/1.39.0/std/io/struct.BufWriter.html

## 진행 상황 표시줄 표시하기

일부 CLI 애플리케이션은 1초 미만으로 실행되지만,
다른 애플리케이션은 몇 분 또는 몇 시간이 걸릴 수 있습니다.
만약 후자 유형의 프로그램을 작성하고 있다면,
사용자에게 무언가가 일어나고 있음을 알려주는 것이 좋습니다.
이를 위해 유용한 상태 업데이트를 출력해야 하며,
가능하면 쉽게 소비할 수 있는 형식으로 출력해야 합니다.

[indicatif] crate를 사용하면 프로그램에 진행 상황 표시줄과 작은 스피너를 추가할 수 있습니다.
다음은 간단한 예입니다:

```rust,ignore
{{#include output-progressbar.rs:1:9}}
```

더 자세한 내용은 [문서][indicatif docs]
및 [예제][indicatif examples]
을 참조하십시오.

[indicatif]: https://crates.io/crates/indicatif
[indicatif docs]: https://docs.rs/indicatif
[indicatif examples]: https://github.com/console-rs/indicatif/tree/main/examples

## 로그 기록

프로그램에서 일어나는 일을 더 쉽게 이해하기 위해,
로그 문을 추가하는 것이 좋습니다.
일반적으로 애플리케이션을 작성하는 동안에는 쉽습니다.
하지만 6개월 후에 이 프로그램을 다시 실행할 때 매우 유용합니다.
어느 정도로 로그 기록은 `println!`과 동일합니다.
단, 메시지의 중요도를 지정할 수 있습니다.
일반적으로 사용할 수 있는 레벨은 _오류_, _경고_, _정보_, _디버그_, 및 _추적_입니다.
(_오류_가 가장 높은 우선순위를 가지고, _추적_은 가장 낮습니다).

간단한 로그 기록을 애플리케이션에 추가하려면,
두 가지가 필요합니다.
[log] crate (로그 레벨에 따라 이름이 지정된 매크로를 포함) 및 실제로 로그 출력을 유용한 곳으로 작성하는 _어댑터_입니다.
로그 어댑터를 사용할 수 있는 것은 매우 유연합니다.
예를 들어, 로그를 콘솔뿐만 아니라 [syslog] 또는 중앙 로그 서버로 작성할 수 있습니다.

[syslog]: https://ko.wikipedia.org/wiki/Syslog

현재 우리가 CLI 애플리케이션을 작성하고 있기 때문에,
사용하기 쉬운 어댑터는 [env_logger]입니다.
```rust,ignore
{{#include output-log.rs}}
```

 `src/bin/output-log.rs` 파일이 있다고 가정하면,
Linux와 macOS에서 다음과 같이 실행할 수 있습니다:
```console
$ env RUST_LOG=info cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

Windows PowerShell에서 실행하면 다음과 같습니다.
```console
$ $env:RUST_LOG="info"
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

Windows CMD에서 실행하면 다음과 같습니다.
```console
$ set RUST_LOG=info
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

`RUST_LOG` 환경 변수를 사용하여 로그 설정을 변경할 수 있습니다.
`env_logger`는 프로그래밍적으로 설정을 조정할 수 있는 빌더도 제공합니다.
예를 들어, 기본적으로 _info_ 레벨 메시지를 표시하도록 설정할 수 있습니다.

다양한 대체 로그 어댑터와 `log`의 대안 또는 확장 기능이 있습니다.
응용 프로그램에 많은 로그가 필요하다면, 이러한 옵션을 검토하고 사용자의 작업을 더욱 쉽게 만들어 주세요.

<aside>

**팁:**
경험에 따르면, 심지어 약간 유용한 CLI 프로그램도 오랫동안 사용될 수 있습니다.
(특히 일시적인 해결책으로 만들어졌을 때)
응용 프로그램이 작동하지 않고 누군가(예: 미래의 당신)가 원인을 파악해야 하는 경우,
`--verbose`를 전달하여 추가 로그 출력을 얻을 수 있도록 하는 것은
디버깅 시간을 분에서 시간으로 줄이는 데 도움이 될 수 있습니다.
[clap-verbosity-flag] crate는 `clap`를 사용하여 프로젝트에 `--verbose`를 빠르게 추가하는 방법을 제공합니다.

[clap-verbosity-flag]: https://crates.io/crates/clap-verbosity-flag

</aside>
