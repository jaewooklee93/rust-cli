# 기계와의 소통

명령줄 도구의 힘은 다른 도구와 결합할 때 가장 빛납니다.

이것은 새로운 아이디어가 아닙니다:
사실, 이는 [Unix 철학]에서 나온 문장입니다:

> 모든 프로그램의 출력이 아직 알려지지 않은 다른 프로그램의 입력이 되기를 기대하십시오.

[Unix 철학]: https://ko.wikipedia.org/wiki/Unix_철학

우리의 프로그램이 이 기대를 충족한다면,
사용자들은 행복할 것입니다.
이것이 잘 작동하도록 하기 위해,
우리는 인간을 위한 아름다운 출력뿐만 아니라 다른 프로그램이 필요로 하는 버전도 제공해야 합니다.
우리가 이것을 어떻게 할 수 있는지 살펴보겠습니다.

<aside>

**참고:**
튜토리얼의 [CLI 출력 챕터][output]를 먼저 읽으십시오.
터미널에 출력을 작성하는 방법을 설명합니다.

[output]: ../tutorial/output.html

</aside>

## 누가 이것을 읽고 있나요?

가장 먼저 묻고 싶은 질문은 다음과 같습니다.
우리의 출력은 화려한 터미널 앞에 있는 사람을 위한 것인가요,
 아니면 다른 프로그램을 위한 것인가요?
이를 답하기 위해,
[is-terminal]과 같은 crate를 사용할 수 있습니다.

[is-terminal]: https://crates.io/crates/is-terminal

```rust,ignore
use is_terminal::IsTerminal as _;

if std::io::stdout().is_terminal() {
    println!("I'm a terminal");
} else {
    println!("I'm not");
}
```

출력을 읽을 사람에 따라 추가 정보를 추가할 수 있습니다.
예를 들어, 사람들은 색상을 좋아합니다.
`ls`를 임의의 Rust 프로젝트에서 실행하면 다음과 같은 내용을 볼 수 있습니다.

```console
$ ls
CODE_OF_CONDUCT.md   LICENSE-APACHE       examples
CONTRIBUTING.md      LICENSE-MIT          proptest-regressions
Cargo.lock           README.md            src
Cargo.toml           convey_derive        target
```

이 스타일은 사람을 위해 만들어졌기 때문에,
대부분의 구성에서
일부 이름 (예: `src`)을 색상으로 표시하여
그것들이 디렉토리임을 보여줍니다.
대신 이를 파일이나 `cat`와 같은 프로그램에 연결하면,
`ls`는 출력을 조정합니다.
터미널 창에 맞는 열을 사용하는 대신
각 항목을 하나씩 줄에 출력합니다.
또한 색상을 출력하지 않습니다.

```console
$ ls | cat
CODE_OF_CONDUCT.md
CONTRIBUTING.md
Cargo.lock
Cargo.toml
LICENSE-APACHE
LICENSE-MIT
README.md
convey_derive
examples
proptest-regressions
src
target
```

## 기계가 쉽게 이해할 수 있는 출력 형식

과거에는 명령줄 도구가 생성하는 유일한 출력 유형이 문자열이었습니다.
이는 일반적으로 터미널 앞에 있는 사람들에게는 괜찮습니다.
사람들은 텍스트를 읽고 그 의미를 추론할 수 있기 때문입니다.
그러나 다른 프로그램은 그렇지 않습니다.
프로그램의 작성자가 `ls`와 같은 도구의 출력을 이해하도록 작동하는 파서를 포함하지 않는 한,
`ls`와 같은 도구의 출력을 이해할 수 없습니다.

이로 인해 출력이 일반적으로 파싱하기 쉬운 형식으로 제한되었습니다.
각 레코드가 고유한 줄에 있고,
각 줄은 탭으로 구분된 콘텐츠를 포함하는 TSV(탭으로 구분된 값)와 같은 형식이 매우 인기가 있습니다.
이러한 텍스트 줄 기반의 간단한 형식은 `ls`와 같은 도구의 출력에 `grep`과 같은 도구를 사용할 수 있도록 합니다.
`| grep Cargo`는 줄이 `ls`에서 나왔는지 파일에서 나왔는지 상관없이 줄별로 필터링합니다.

단점은 `grep` 명령어를 사용하여 `ls`가 제공한 모든 디렉토리를 필터링할 수 없다는 것입니다.
이를 위해 각 디렉토리 항목에는 추가 데이터가 필요합니다.

## 기계용 JSON 출력

탭 구분 값은 구조화된 데이터를 출력하는 간단한 방법이지만,
다른 프로그램이 어떤 필드를 기대하는지(그리고 어떤 순서로 기대하는지)
알아야 하며, 다른 유형의 메시지를 출력하는 것은 어렵습니다.
예를 들어,
프로그램이 현재 다운로드를 기다리고 있다는 메시지를 소비자에게 전달하고,
그 후에 가져온 데이터를 설명하는 메시지를 출력하고 싶다고 가정해 보겠습니다.
이러한 메시지는 매우 다르며,
TSV 출력에 통합하려면 그들을 구분하는 방법을 고안해야 합니다.
두 개의 목록이 포함된 메시지를 출력하고 싶을 때도 마찬가지입니다.
이 목록은 길이가 다른 항목을 포함합니다.

그러나, 대부분의 프로그래밍 언어/환경에서 쉽게 분석할 수 있는 형식을 선택하는 것이 좋습니다.
최근 몇 년 동안 많은 응용 프로그램이 데이터를 [JSON] 형식으로 출력할 수 있게 되었습니다.
실제로 모든 언어에서 파서가 존재하는 만큼 간단하면서도 많은 경우에 유용할 만큼 강력합니다.
사람이 읽을 수 있는 텍스트 형식이지만,
JSON 데이터를 분석하고 JSON으로 데이터를 시리얼화하는 데 매우 빠른 구현이 많이 개발되었습니다.

[JSON]: https://www.json.org/

위 설명에서,
우리의 프로그램이 "메시지"를 작성한다고 말했습니다.
이것은 출력에 대한 좋은 생각입니다.
프로그램이 반드시 하나의 데이터 블록만 출력하는 것은 아니며,
실행 중에 많은 양의 다른 정보를 출력할 수 있습니다.
JSON을 사용하여 출력할 때 이러한 접근 방식을 지원하는 쉬운 방법은 메시지 하나당 하나의 JSON 문서를 작성하고,
각 JSON 문서를 새 줄에 넣는 것입니다(때로는 [Line-delimited JSON][jsonlines]이라고 합니다).
이는 일반적인 `println!`을 사용하는 것만큼 간단하게 구현할 수 있습니다.

[jsonlines]: https://en.wikipedia.org/wiki/JSON_streaming#Line-delimited_JSON

다음은 [serde_json]에서 제공하는 `json!` 매크로를 사용하여 Rust 소스 코드에서 유효한 JSON을 빠르게 작성하는 간단한 예입니다.

[serde_json]: https://crates.io/crates/serde_json

```rust,ignore
{{#include machine-communication.rs}}
```

그리고 여기 출력입니다:

```console
$ cargo run -q
Hello world
$ cargo run -q -- --json
{"content":"Hello world","type":"message"}
```

( `cargo` 를 `-q` 플래그로 실행하면 일반적인 출력이 숨겨집니다.
`--` 뒤에 있는 인수는 프로그램에 전달됩니다.)

### 실제 예시: ripgrep

_[ripgrep]_는 Rust로 작성된 _grep_ 또는 _ag_의 대체품입니다.
기본적으로 다음과 같은 출력을 생성합니다.

[ripgrep]: https://github.com/BurntSushi/ripgrep

```console
$ rg default
src/lib.rs
37:    Output::default()

src/components/span.rs
6:    Span::default()
```

하지만 `--json`을 지정하면 다음과 같이 출력합니다.

```console
$ rg default --json
{"type":"begin","data":{"path":{"text":"src/lib.rs"}}}
{"type":"match","data":{"path":{"text":"src/lib.rs"},"lines":{"text":"    Output::default()\n"},"line_number":37,"absolute_offset":761,"submatches":[{"match":{"text":"default"},"start":12,"end":19}]}}
{"type":"end","data":{"path":{"text":"src/lib.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":137622,"human":"0.000138s"},"searches":1,"searches_with_match":1,"bytes_searched":6064,"bytes_printed":256,"matched_lines":1,"matches":1}}}
{"type":"begin","data":{"path":{"text":"src/components/span.rs"}}}
{"type":"match","data":{"path":{"text":"src/components/span.rs"},"lines":{"text":"    Span::default()\n"},"line_number":6,"absolute_offset":117,"submatches":[{"match":{"text":"default"},"start":10,"end":17}]}}
{"type":"end","data":{"path":{"text":"src/components/span.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":22025,"human":"0.000022s"},"searches":1,"searches_with_match":1,"bytes_searched":5221,"bytes_printed":277,"matched_lines":1,"matches":1}}}
{"data":{"elapsed_total":{"human":"0.006995s","nanos":6994920,"secs":0},"stats":{"bytes_printed":533,"bytes_searched":11285,"elapsed":{"human":"0.000160s","nanos":159647,"secs":0},"matched_lines":2,"matches":2,"searches":2,"searches_with_match":2}},"type":"summary"}
```

각 JSON 문서는 `type` 필드를 포함하는 객체(맵)입니다.
이를 통해 `rg`에 대한 간단한 프론트엔드를 작성하여
이 문서가 들어오면서 매치를 표시할 수 있습니다.
(검색 중인 파일도 함께)
_ripgrep_가 검색을 완료하기 전에도.

<aside>

**참고:**
이것은 Visual Studio Code가 코드 검색에 _ripgrep_을 사용하는 방식입니다.

</aside>

## 우리에게 파이프로 전달된 입력을 처리하는 방법

예를 들어, 파일의 단어 수를 읽는 프로그램이 있다고 가정해 보겠습니다.

``` rust,ignore
{{#include machine-communication-wc.rs}}
```

파일 경로를 입력받아 줄 단위로 읽고 공백으로 구분된 단어의 개수를 세는 프로그램입니다.

실행 시 파일의 총 단어 수를 출력합니다.

``` console
$ cargo run README.md
Words in README.md: 47
```

하지만 프로그램에 송수신된 단어 수를 세고 싶다면 어떻게 하나요?
Rust 프로그램은 [Stdin
구조체](https://doc.rust-lang.org/std/io/struct.Stdin.html)를 통해 [stdin 함수](https://doc.rust-lang.org/std/io/fn.stdin.html)를 사용하여 표준 라이브러리에서 가져올 수 있습니다. 파일의 줄을 읽는 것과 유사하게, stdin에서 줄을 읽을 수 있습니다.

다음은 stdin을 통해 전달된 내용의 단어를 세는 프로그램입니다.

``` rust,ignore
{{#include machine-communication-stdin.rs}}
```

만약 그 프로그램을 텍스트를 입력으로 파이프 라인을 사용하여 실행하면, `-`가 `stdin`에서 읽는 의도를 나타내는 경우, 단어 수를 출력합니다.

``` console
$ echo "hi there friend" | cargo run -- -
Words from stdin: 3
```

입력이 파이프를 통해 프로그램에 전달되는 것으로 기대하기 때문에, stdin이 비주얼 인터랙티브가 아니어야 합니다. stdin이 tty라면, 왜 작동하지 않는지 명확하게 알 수 있도록 도움말 문서를 출력합니다.
