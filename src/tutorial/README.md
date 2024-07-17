# 15분 안에 Rust로 명령줄 앱 만들기

이 튜토리얼에서는 [Rust]로 명령줄 인터페이스(CLI) 앱을 작성하는 방법을 안내합니다.
약 15분 안에 실행 가능한 프로그램을 만들 수 있습니다(약 1.3장).
그 후에는 작은 도구를 배포할 수 있도록 프로그램을 조정합니다.

[Rust]: https://rust-lang.org/

Rust의 기본적인 개념을 배우고, 어디에서 더 많은 정보를 얻을 수 있는지 알게 됩니다.
필요하지 않은 부분은 건너뛰거나 언제든지 시작할 수 있습니다.

<aside>

**사전 요구 사항:**
이 튜토리얼은 프로그래밍에 대한 일반적인 소개를 대체하지 않으며, 몇 가지 일반적인 개념에 익숙해야 합니다.
명령줄/터미널 사용에 익숙해야 합니다.
다른 언어를 이미 알고 있다면, 이는 Rust와의 첫 만남이 될 수 있습니다.

**도움 요청:**
Rust의 기능에 압도되거나 혼란스러울 때는 공식 문서를 참조하십시오.
Rust의 공식 문서는 매우 상세하며, 특히 책인 [The Rust Programming Language]를 추천합니다.
대부분의 Rust 설치에 포함되어 있습니다(`rustup doc`),
그리고 [doc.rust-lang.org]에서 온라인으로 확인할 수 있습니다.

[doc.rust-lang.org]: https://doc.rust-lang.org

또한 Rust 커뮤니티에 질문을 하는 것도 좋습니다.
Rust 커뮤니티는 친절하고 도움이 되는 것으로 유명합니다.
[커뮤니티 페이지]에서 Rust를 논의하는 곳 목록을 확인하십시오.

[커뮤니티 페이지]: https://www.rust-lang.org/community

</aside>

어떤 종류의 프로젝트를 만들고 싶으신가요?
간단한 것부터 시작해 보는 건 어떨까요?
`grep` 클론을 작게 만들어 보겠습니다.
즉, 문자열과 경로를 제공하면 해당 문자열이 포함된 줄만 출력하는 도구입니다.
`grrs`라고 부르겠습니다(발음은 “grass”).

결국, 다음과 같이 도구를 실행할 수 있도록 하겠습니다:

```console
$ cat test.txt
foo: 10
bar: 20
baz: 30
$ grrs foo test.txt
foo: 10
$ grrs --help
[some help text explaining the available options]
```

<aside class="note">

**참고:**
이 책은 [Rust 2018]을 위한 것입니다.
코드 예제는 Rust 2015에서도 사용할 수 있지만,
일부 수정이 필요할 수 있습니다; 예를 들어 `extern crate foo;`를 추가해야 할 수 있습니다.

Rust 1.31.0 (또는 이후 버전)을 실행하고 있으며 `edition = "2018"`이 `Cargo.toml` 파일의 `[package]` 섹션에 설정되어 있는지 확인하십시오.

[Rust 2018]: https://doc.rust-lang.org/edition-guide/index.html

</aside>
