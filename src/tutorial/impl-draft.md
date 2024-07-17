# _grrs_의 첫 번째 구현

지난 챕터에서 명령줄 인수에 대해 배운 후,
입력 데이터를 가지게 되었고,
실제 도구를 작성하기 시작할 수 있습니다.
`main` 함수에는 현재 다음과 같은 줄만 포함되어 있습니다:

```rust,ignore
{{#include impl-draft.rs:13:13}}
```

(우리는 단순히 프로그램이 예상대로 작동하는지 보여주기 위해 임시로 넣었던 `println` 문을 삭제합니다.)

지금부터 우리가 얻은 파일을 열어보겠습니다.

```rust,ignore
{{#include impl-draft.rs:14:14}}
```

<aside>

**참고:**
여기 [`.expect`] 메서드를 보셨나요?
이는 프로그램이 입력 파일을 읽을 수 없는 경우
즉시 종료되는 단축 함수입니다.
매우 보기 좋지 않으며,
다음 챕터인 [더 나은 오류 보고]
에서 이를 개선하는 방법을 살펴보겠습니다.

[`.expect`]: https://doc.rust-lang.org/1.39.0/std/result/enum.Result.html#method.expect
[더 나은 오류 보고]:./errors.html

</aside>

이제 줄을 반복하고 패턴을 포함하는 각 줄을 출력해 보겠습니다:

```rust,ignore
{{#include impl-draft.rs:16:20}}
```

## 마무리

이제 코드는 다음과 같이 보일 것입니다.

```rust,ignore
{{#include impl-draft.rs}}
```

시도해보세요: `cargo run -- main src/main.rs`가 이제 작동해야 합니다!

<aside class="exercise">

**독자를 위한 연습:**
이것은 최적화된 구현이 아닙니다:
파일 전체를 메모리에 읽어들일 것입니다.
– 파일이 얼마나 크더라도.
최적화하는 방법을 찾으세요!
(한 가지 아이디어는 `read_to_string()` 대신 [`BufReader`]
을 사용하는 것입니다.)

[`BufReader`]: https://doc.rust-lang.org/1.39.0/std/io/struct.BufReader.html

</aside>
