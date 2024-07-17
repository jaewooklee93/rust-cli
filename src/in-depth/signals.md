# 신호 처리

프로세스
명령줄 응용 프로그램과 같이
운영 체제에서 보낸 신호에 반응해야 합니다.
가장 일반적인 예는 아마도 <kbd>Ctrl</kbd>+<kbd>C</kbd>일 것입니다.
이 신호는 일반적으로 프로세스가 종료하도록 알리는 신호입니다.
Rust 프로그램에서 신호를 처리하려면
이러한 신호를 수신하는 방법과
이에 반응하는 방법을 고려해야 합니다.

<aside>

**참고:**
응용 프로그램이 우아하게 종료하지 않아도 되는 경우
기본 처리가 괜찮습니다
(즉, 즉시 종료하고
OS가 열린 파일 핸들과 같은 리소스를 정리하도록 합니다).
그 경우:
이 챕터에서 설명하는 내용을 하지 않아도 됩니다!

그러나,
자신을 정리해야 하는 응용 프로그램의 경우
이 챕터는 매우 관련이 있습니다!
예를 들어,
응용 프로그램이
네트워크 연결을 제대로 닫아야 하는 경우
(다른 쪽 프로세스에 "안녕히 가세요"라고 말해야 합니다),
임시 파일을 제거하거나,
시스템 설정을 재설정해야 하는 경우,
더 읽어보세요.

</aside>

## 운영 체제 간 차이

Unix 시스템(Linux, macOS, FreeBSD와 같음)
프로세스는 [신호]를 받을 수 있습니다.
프로세스는 기본적인 방식(OS에서 제공)으로 반응하거나,
신호를 잡아 프로그램에서 정의된 방식으로 처리하거나,
신호를 완전히 무시할 수 있습니다.

[신호]: https://manpages.ubuntu.com/manpages/bionic/en/man7/signal.7.html

Windows는 신호를 가지고 있지 않습니다.
[콘솔 핸들러]를 사용하여 특정 이벤트가 발생했을 때 실행되는 콜백 함수를 정의할 수 있습니다.
또한 [구조화된 예외 처리]가 시스템 예외의 여러 유형(0으로 나누기, 무효 액세스 예외, 스택 오버플로우 등)을 처리합니다.

[콘솔 핸들러]: https://docs.microsoft.com/en-us/windows/console/console-control-handlers
[구조화된 예외 처리]: https://docs.microsoft.com/en-us/windows/desktop/debug/structured-exception-handling

## 먼저: Ctrl+C 처리

[ctrlc] crate는 이름에서 알 수 있듯이 다음과 같은 기능을 제공합니다:
사용자가 <kbd>Ctrl</kbd>+<kbd>C</kbd>를 누를 때, 플랫폼에 상관없이 반응할 수 있도록 합니다.
주요 사용 방법은 다음과 같습니다:

[ctrlc]: https://crates.io/crates/ctrlc

```rust,ignore
{{#include signals-ctrlc.rs}}
```

물론, 이것만으로는 도움이 되지 않습니다.
메시지를 출력하지만 프로그램을 중지하지 않습니다.

실제 프로그램에서는, 신호 처리기에서 변수를 설정하는 것이 좋습니다.
이 변수는 프로그램의 여러 곳에서 확인할 수 있습니다.
예를 들어,
신호 처리기에서 `Arc<AtomicBool>`(쓰레드 간 공유 가능한 불리언)
을 설정하고,
뜨거운 루프 또는 스레드를 기다릴 때,
주기적으로 그 값을 확인하고 참이 되면 중단할 수 있습니다.

## 다른 유형의 신호 처리하기

[ctrlc] crate는 <kbd>Ctrl</kbd>+<kbd>C</kbd>만 처리하거나, Unix 시스템에서 `SIGINT` ( "중지" 신호)라고 불립니다.
더 많은 Unix 신호에 반응하려면 [signal-hook]을 살펴보세요.
그 설계는 [이 블로그 게시물][signal-hook-post]에 설명되어 있으며, 현재 가장 많은 커뮤니티 지원을 받는 라이브러리입니다.

다음은 간단한 예입니다:

```rust,ignore
{{#include signals-hooked.rs}}
```

[시그널 훅 게시물]: https://vorner.github.io/2018/06/28/signal-hook.html

## 채널 사용

변수를 설정하고 프로그램의 다른 부분이 그것을 확인하는 대신,
채널을 사용할 수 있습니다.
채널을 생성하여 신호 처리기가 신호가 수신될 때마다 값을 전송합니다.
응용 프로그램 코드에서는 이 채널과 다른 채널을 스레드 간 동기화 지점으로 사용합니다.
[crossbeam-channel]을 사용하면 다음과 같습니다.

[crossbeam-channel]: https://crates.io/crates/crossbeam-channel

```rust,ignore
{{#include signals-channels.rs}}
```

## futures와 스트림 사용

[tokio]를 사용하는 경우,
대부분 비동기 패턴과 이벤트 기반 디자인으로 애플리케이션을 작성하고 있을 것입니다.
crossbeam의 채널을 직접 사용하는 대신,
signal-hook의 `tokio-support` 기능을 활성화할 수 있습니다.
이를 통해 signal-hook의 `Signals` 유형에 [`.into_async()`]
을 호출하여 `futures::Stream`을 구현하는 새로운 유형을 가져올 수 있습니다.

[signal-hook]: https://crates.io/crates/signal-hook
[tokio]: https://tokio.rs/
[`.into_async()`]: https://docs.rs/signal-hook/0.1.6/signal_hook/iterator/struct.Signals.html#method.into_async

## 첫 Ctrl+C 처리 중에 또 다른 Ctrl+C를 받았을 때 어떻게 해야 할까요?

대부분의 사용자는 <kbd>Ctrl</kbd>+<kbd>C</kbd>를 누르고,
프로그램이 종료될 때까지 몇 초를 기다리거나,
무슨 일이 일어나고 있는지 알려줄 것입니다.
만약 그렇지 않다면,
다시 <kbd>Ctrl</kbd>+<kbd>C</kbd>를 누를 것입니다.
일반적인 동작은 애플리케이션을 즉시 종료하는 것입니다.
