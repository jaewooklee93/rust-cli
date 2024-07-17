# Exit 코드

프로그램이 항상 성공하는 것은 아닙니다.
오류가 발생하면,
필요한 정보를 올바르게 전달해야 합니다.
[사용자에게 오류를 알리는 방법](human-communication.html) 외에도,
대부분의 시스템에서,
프로세스가 종료될 때,
종료 코드(대부분의 플랫폼에서 호환되는 0과 255 사이의 정수)를 출력합니다.
프로그램 상태에 따라 올바른 코드를 출력하려고 노력해야 합니다.
예를 들어,
프로그램이 성공적으로 실행될 때는 `0`으로 종료해야 합니다.

오류가 발생하면 상황이 조금 더 복잡해집니다.
실제로는,
많은 도구가 일반적인 오류 발생 시 `1`로 종료합니다.
현재 Rust는 프로세스가 panic할 때 `101`의 종료 코드를 설정합니다.
그 외에도 사람들은 프로그램에서 다양한 작업을 수행했습니다.

그렇다면 어떻게 해야 할까요?
BSD 생태계는 종료 코드에 대한 일반적인 정의를 수집했습니다.
(여기에서 찾을 수 있습니다: [`sysexits.h`][`sysexits.h`]).
Rust 라이브러리 [`exitcode`]는 이러한 코드를 제공하며,
응용 프로그램에서 사용할 수 있습니다.
가능한 사용 값에 대한 API 문서를 참조하십시오.

`Cargo.toml`에 `exitcode` 의존성을 추가한 후,
다음과 같이 사용할 수 있습니다:

```rust,ignore
fn main() {
    // ...actual work...
    match result {
        Ok(_) => {
            println!("Done!");
            std::process::exit(exitcode::OK);
        }
        Err(CustomError::CantReadConfig(e)) => {
            eprintln!("Error: {}", e);
            std::process::exit(exitcode::CONFIG);
        }
        Err(e) => {
            eprintln!("Error: {}", e);
            std::process::exit(exitcode::DATAERR);
        }
    }
}
```


[`exitcode`]: https://crates.io/crates/exitcode
[`sysexits.h`]: https://www.freebsd.org/cgi/man.cgi?query=sysexits&apropos=0&sektion=0&manpath=FreeBSD+11.2-stable&arch=default&format=html
