# 프로젝트 설정

아직 설치하지 않았다면,
[Rust 설치](https://www.rust-lang.org/tools/install)를 컴퓨터에 설치하세요.
(몇 분만 걸립니다).
그 후, 터미널을 열고 애플리케이션 코드를 저장할 디렉토리로 이동하세요.

[Rust 설치]: https://www.rust-lang.org/tools/install

`cargo new grrs`를 실행하여 시작하세요.
프로젝트 코드를 저장하는 디렉토리에서 실행하세요.
만약 `grrs` 디렉토리에서 `cargo run`을 실행하고 "Hello World"를 볼 수 있다면, 설정이 완료되었습니다.

## 어떻게 보일까요?

```console
$ cargo new grrs
     Created binary (application) `grrs` package
$ cd grrs/
$ cargo run
   Compiling grrs v0.1.0 (/Users/pascal/code/grrs)
    Finished dev [unoptimized + debuginfo] target(s) in 0.70s
     Running `target/debug/grrs`
Hello, world!
```
