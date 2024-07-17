# 더 나은 오류 보고

모두 오류가 발생할 수밖에 없다는 사실을 받아들일 수밖에 없습니다.
다른 많은 언어와 달리,
Rust를 사용할 때는 이 현실을 알아채고 처리하지 않을 수 없습니다.
예외가 없기 때문에,
모든 가능한 오류 상태가 종종 함수의 반환 유형에 암호화됩니다.

## 결과

[`read_to_string`]과 같은 함수는 문자열을 반환하지 않습니다.
대신, 성공 또는 실패 여부를 나타내는 [`Result`]를 반환합니다.

[`Result`]는 `String` 또는 어떤 유형의 오류 (`std::io::Error`와 같이)를 포함합니다.

[`read_to_string`]: https://doc.rust-lang.org/1.39.0/std/fs/fn.read_to_string.html
[`Result`]: https://doc.rust-lang.org/1.39.0/std/result/index.html
[`std::io::Error`]: https://doc.rust-lang.org/1.39.0/std/io/type.Result.html

어떤 경우인지 알아내는 방법은 무엇입니까?
`Result`는 `enum`이기 때문에 `match`를 사용하여 어떤 변형인지 확인할 수 있습니다.

```rust,no_run
let result = std::fs::read_to_string("test.txt");
match result {
    Ok(content) => { println!("File content: {}", content); }
    Err(error) => { println!("Oh noes: {}", error); }
}
```

<aside>

**참고:**

Rust에서 enum이 무엇인지 또는 어떻게 작동하는지 잘 모르시나요?
[Rust 책의 이 장을 확인](https://doc.rust-lang.org/1.39.0/book/ch06-00-enums.html)
 enum에 대해 자세히 알아보세요.

</aside>

## Unwrapping

이제 파일의 내용에 액세스할 수 있지만, `match` 블록 이후에는 그 내용을 사용할 수 없습니다.
이를 위해서는 오류 케이스를 어떻게 처리할지 알아야 합니다.
문제는 `match` 블록의 모든 팔이 동일한 유형의 것을 반환해야 한다는 것입니다.
하지만 이를 우회하는 멋진 방법이 있습니다.

```rust,no_run
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { panic!("Can't deal with {}, just exit here", error); }
};
println!("file content: {}", content);
```

매치 블록 후 `content`에서 String을 사용할 수 있습니다.
`result`가 오류라면 String은 존재하지 않을 것입니다.
그러나 프로그램이 `content`를 사용하는 지점에 도달하기 전에 종료되므로 괜찮습니다.

이것은 극단적으로 보일 수 있지만,
매우 편리합니다.
프로그램이 해당 파일을 읽어야 하며 파일이 없으면 아무것도 할 수 없다면,
종료는 유효한 전략입니다.
`Result`에 `unwrap`이라는 단축 메서드도 있습니다.

```rust,no_run
let content = std::fs::read_to_string("test.txt").unwrap();
```

## 당황하지 마세요

물론, 프로그램을 중단하는 것이 오류를 처리하는 유일한 방법은 아닙니다.
`panic!` 대신, `return`을 쉽게 작성할 수도 있습니다.

```rust,no_run
# fn main() -> Result<(), Box<dyn std::error::Error>> {
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { return Err(error.into()); }
};
# Ok(())
# }
```

그러나 이것은 함수가 반환해야 하는 반환 유형을 변경합니다.
사실, 우리의 예제에 항상 숨겨져 있던 것이 있었습니다.
이 코드가 있는 함수 시그니처입니다.
그리고 마지막 예제에서 `return`을 사용하는 경우,
그것이 중요해집니다.
다음은 _완전한_ 예제입니다:

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let result = std::fs::read_to_string("test.txt");
    let content = match result {
        Ok(content) => { content },
        Err(error) => { return Err(error.into()); }
    };
    println!("file content: {}", content);
    Ok(())
}
```

반환형이 `Result`입니다!
이유로 `return Err(error);`를 두 번째 `match` 팔레트에서 작성할 수 있습니다.
아래에 `Ok(())`가 있는 것을 보셨나요?
함수의 기본 반환값이며
"결과는 괜찮고, 내용이 없습니다"라는 의미입니다.

<aside>

**참고:**
왜 이것이 `return Ok(());`로 작성되지 않았을까요?
쉽게 `return Ok(());`로 작성할 수 있습니다. – 이것도 완전히 유효합니다.
Rust에서 어떤 블록의 마지막 표현식은 그 블록의 반환값이며,
불필요한 `return`을 생략하는 것이 관습입니다.

</aside>

## 물음표

`.unwrap()`을 호출하는 것처럼 `panic!`을 사용하는 `match` 문의 단축키와 같이,
`match` 문에서 `return`하는 `match` 문의 단축키도 있습니다:
`?`.

맞아요, 물음표입니다.
`Result` 유형의 값에 이 연산자를 추가하면,
Rust는 내부적으로 우리가 막 작성한 `match` 문과 매우 유사한 것을 확장합니다.

시도해 보세요:

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("test.txt")?;
    println!("file content: {}", content);
    Ok(())
}
```

매우 간결합니다!

<aside>

**참고:**
여기서 몇 가지 더 중요하지 않은 내용이 있습니다.
예를 들어,
`main` 함수에서의 오류 유형은 `Box<dyn std::error::Error>`입니다.
하지만 위에서 보았듯이 `read_to_string`은 [`std::io::Error`](https://doc.rust-lang.org/1.39.0/std/io/error.html)를 반환합니다.
이는 `?`가 오류 유형을 _변환_하는 코드로 확장되기 때문입니다.

`Box<dyn std::error::Error>` 또한 흥미로운 유형입니다.
`Box`는 _모든_ 유형을 포함할 수 있는데,
표준 [`Error`](https://doc.rust-lang.org/1.39.0/std/error/trait.Error.html) 추상화를 구현합니다.
즉, 거의 모든 오류를 이 상자에 넣을 수 있으므로,
`Result`를 반환하는 일반적인 함수에 `?`를 사용할 수 있습니다.

</aside>

## 맥락 제공

`main` 함수에서 `?`를 사용할 때 발생하는 오류는 괜찮지만, 좋지는 않습니다.
예를 들어:
`std::fs::read_to_string("test.txt")?`를 실행하지만 `test.txt` 파일이 존재하지 않으면,
다음과 같은 출력이 나타납니다:

```text
Error: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

코드에 파일 이름이 직접 포함되지 않은 경우,
어떤 파일이 `NotFound`인지 알기 어렵습니다.
이 문제를 해결하는 방법은 여러 가지가 있습니다.

예를 들어, 우리는 자신의 오류 유형을 만들고,
그것을 사용하여 사용자 정의 오류 메시지를 생성할 수 있습니다.

```rust,ignore
{{#include errors-custom.rs}}
```

이제,
이것을 실행하면 사용자 정의 오류 메시지를 얻습니다:

```text
Error: CustomError("Error reading `test.txt`: No such file or directory (os error 2)")
```

아주 예쁘지는 않지만,
나중에 우리 유형의 디버그 출력을 쉽게 적응할 수 있습니다.

이 패턴은 사실 매우 일반적입니다.
하지만 하나의 문제가 있습니다.
우리는 원래 오류를 저장하지 않고,
오직 문자열 표현만 저장합니다.
자주 사용되는 [`anyhow`] 라이브러리는 이 문제에 대한 깔끔한 해결책을 제공합니다.
우리의 `CustomError` 유형과 유사하게,
its [`Context`] 트레이트는 설명을 추가하는 데 사용될 수 있습니다.
또한 원래 오류를 유지하기 때문에,
원인을 파악하는 데 도움이 되는 오류 메시지의 "체인"을 얻습니다.

[`anyhow`]: https://docs.rs/anyhow
[`Context`]: https://docs.rs/anyhow/1.0/anyhow/trait.Context.html

먼저 `anyhow` crate를 가져오기 위해
`anyhow = "1.0"`을 `Cargo.toml` 파일의 `[dependencies]` 섹션에 추가합니다.

완전한 예제는 다음과 같습니다.

```rust,ignore
{{#include errors-exit.rs}}
```

이것은 오류를 출력합니다:

```text
Error: could not read file `test.txt`

Caused by:
    No such file or directory (os error 2)
```

## 마무리

이제 코드는 다음과 같이 보일 것입니다.

```rust,ignore
{{#include errors-impl.rs}}
```
