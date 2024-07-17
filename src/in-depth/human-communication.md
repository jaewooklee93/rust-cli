# 인간과의 소통

먼저 [CLI 출력에 대한 튜토리얼 챕터][output]를 읽어보세요.
터미널에 출력을 작성하는 방법을 설명합니다.
이 챕터에서는 _어떤 내용을 출력해야 하는지_에 대해 논의할 것입니다.

[output]: ../tutorial/output.html

## 모든 것이 잘 되는 경우

모든 것이 잘 되더라도 응용 프로그램의 진행 상황을 보고하는 것은 유용합니다.
정보가 풍부하고 간결한 메시지를 전달하려고 노력하십시오.
로그에 과도하게 기술적인 용어를 사용하지 마십시오.
기억하세요:
응용 프로그램이 충돌하지 않으므로
사용자는 오류를 찾아보지 않아도 됩니다.

가장 중요한 것은 의사소통 방식의 일관성입니다.
로그를 쉽게 스크롤하여 읽을 수 있도록 동일한 접두사와 문장 구조를 사용하십시오.

응용 프로그램의 출력이 사용자에게 어떤 영향을 미치는지에 대한 이야기를 전달하도록 시도하십시오.
이에는 필요한 단계의 시간 순서를 보여주거나
장시간 실행되는 작업에 대한 진행 상황 표시줄과 지표를 포함할 수 있습니다.
사용자는 응용 프로그램이 이해하기 어려운 신비로운 작업을 수행하고 있다는 느낌을 받지 않아야 합니다.

## 어떤 상황이 혼란스러울 때

비정상 상태를 전달할 때 일관성이 중요합니다.
엄격한 로그 레벨을 따르지 않는, 많은 로그를 기록하는 애플리케이션은
로그를 기록하지 않는 애플리케이션만큼이나, 심지어는 더 적은 정보를 제공합니다.

따라서,
사건의 심각성과 관련된 메시지의 심각성을 정의하고,
일관된 로그 레벨을 사용하는 것이 중요합니다.
이렇게 하면 사용자는 `--verbose` 플래그 또는 환경 변수(예: `RUST_LOG`)를 통해 로그의 양을 스스로 선택할 수 있습니다.

흔히 사용되는 `log` crate
[로그 레벨을 정의](https://docs.rs/log/0.4.4/log/enum.Level.html)
(심각성이 증가하는 순서로):

- trace
- debug
- info
- warning
- error

_info_를 기본 로그 레벨로 생각하는 것이 좋습니다.
정보적인 출력에 사용하십시오.
(조용한 출력 스타일을 선호하는 일부 애플리케이션은 기본적으로 경고 및 오류만 표시할 수 있습니다.)

또한,
로그 메시지에 걸쳐 유사한 접두사와 문장 구조를 사용하는 것이 항상 좋은 방법입니다.
이는 `grep`과 같은 도구를 사용하여 메시지를 필터링하는 데 유용합니다.
메시지는 스스로 필터링된 로그에서 유용하도록 충분한 맥락을 제공해야 하지만, 동시에 *너무* 상세하지 않아야 합니다.

[로그 레벨을 정의]: https://docs.rs/log/0.4.4/log/enum.Level.html

### 예시 로그 메시지

```console
error: could not find `Cargo.toml` in `/home/you/project/`
```

```console
=> Downloading repository index
=> Downloading packages...
```

다음 로그 출력은 [wasm-pack]에서 가져왔습니다.

```console
 [1/7] Adding WASM target...
 [2/7] Compiling to WASM...
 [3/7] Creating a pkg directory...
 [4/7] Writing a package.json...
 > [WARN]: Field `description` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `repository` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `license` is missing from Cargo.toml. It is not necessary, but recommended
 [5/7] Copying over your README...
 > [WARN]: origin crate has no README
 [6/7] Installing WASM-bindgen...
 > [INFO]: wasm-bindgen already installed
 [7/7] Running WASM-bindgen...
 Done in 1 second
```

## 패닉 발생 시

자주 잊히는 부분 중 하나는 프로그램이 오류 발생 시에도 무언가를 출력한다는 것입니다.
Rust에서 "오류 발생"은 대부분 "패닉"으로 표현됩니다.
(즉, "제어된 오류 발생"은 "운영 체제가 프로세스를 종료시켰다"와 대비됩니다).
기본적으로,
패닉이 발생하면 "패닉 핸들러"가 콘솔에 몇 가지 정보를 출력합니다.

예를 들어,
`cargo new --bin foo`로 새로운 이진 프로젝트를 만들고 `fn main`의 내용을 `panic!("Hello World")`로 바꾸면 프로그램 실행 시 다음과 같은 출력이 나타납니다:

```console
thread 'main' panicked at 'Hello, world!', src/main.rs:2:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

이 정보는 개발자에게 유용합니다.
(놀라운 점: 프로그램이 `main.rs` 파일의 2행 때문에 충돌했습니다).
그러나 소스 코드에 액세스할 수 없는 사용자에게는
이 정보는 매우 가치가 없습니다.
사실, 대부분 혼란스러울 뿐입니다.
그렇기 때문에 사용자 중심의 출력을 제공하는 사용자 정의 panic 처리기를 추가하는 것이 좋습니다.

그러한 작업을 수행하는 라이브러리가 [human-panic]입니다.
CLI 프로젝트에 추가하려면
라이브러리를 가져와서 `main` 함수의 처음에 `setup_panic!()` 매크로를 호출합니다.

```rust,ignore
use human_panic::setup_panic;

fn main() {
   setup_panic!();

   panic!("Hello world")
}
```

이제 매우 친절한 메시지가 나타나며,
사용자에게 할 수 있는 일을 알려줍니다:

```console
Well, this is embarrassing.

foo had a problem and crashed. To help us diagnose the problem you can send us a crash report.

We have generated a report file at "/var/folders/n3/dkk459k908lcmkzwcmq0tcv00000gn/T/report-738e1bec-5585-47a4-8158-f1f7227f0168.toml". Submit an issue or email with the subject of "foo Crash Report" and include the report as an attachment.

- Authors: Your Name <your.name@example.com>

We take privacy seriously, and do not perform any automated error collection. In order to improve the software, we rely on people to submit reports.

Thank you kindly!
```

[human-panic]: https://crates.io/crates/human-panic
[wasm-pack]: https://crates.io/crates/wasm-pack
