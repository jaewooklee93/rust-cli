# Rust 도구 포장 및 배포

프로그램이 다른 사람들이 사용하기에 준비되었다고 생각되면 포장 및 배포할 시간입니다!

몇 가지 접근 방식이 있으며, 가장 빠르게 설정할 수 있는 방법부터 사용자에게 가장 편리한 방법까지 세 가지를 살펴보겠습니다.

## 가장 빠른 방법: `cargo publish`

앱을 게시하는 가장 쉬운 방법은 cargo를 사용하는 것입니다.
외부 의존성을 프로젝트에 추가하는 방법을 기억하시나요?
cargo는 기본 "crate 레지스트리"인 [crates.io]에서 해당 의존성을 다운로드했습니다.
`cargo publish`를 사용하면,
당신도 [crates.io]에 crate를 게시할 수 있습니다.
이는 이진 대상을 포함한 모든 crate에 적용됩니다.

[crates.io]에 crate를 게시하는 것은 매우 간단합니다:
아직 계정을 만들지 않았다면, [crates.io]에 계정을 만드세요.
현재는 GitHub에서 인증을 통해 계정을 생성하므로,
GitHub 계정이 필요합니다.
(그리고 GitHub에 로그인해야 합니다).
다음으로, 로컬 머신에서 cargo를 사용하여 로그인합니다.
[crates.io 계정 페이지]로 이동하여,
새 토큰을 생성하고, `cargo login <your-new-token>`을 실행합니다.
이 작업은 컴퓨터당 한 번만 수행하면 됩니다.
cargo의 [게시 가이드]에서 자세한 내용을 확인할 수 있습니다.

이제 cargo와 crates.io가 당신을 알고 있으므로,
crate를 게시할 준비가 되었습니다.
새로운 crate (버전)을 급하게 게시하기 전에,
`Cargo.toml`을 다시 열어서 필요한 메타데이터를 추가했는지 확인하는 것이 좋습니다.
[cargo의 manifest 형식]의 설명서에서 설정할 수 있는 모든 가능한 필드를 찾을 수 있습니다.
다음은 몇 가지 일반적인 항목에 대한 간략한 개요입니다:

```toml
[package]
name = "grrs"
version = "0.1.0"
authors = ["Your Name <your@email.com>"]
license = "MIT OR Apache-2.0"
description = "A tool to search files"
readme = "README.md"
homepage = "https://github.com/you/grrs"
repository = "https://github.com/you/grrs"
keywords = ["cli", "search", "demo"]
categories = ["command-line-utilities"]
```

<aside class="note">

**참고:**
이 예제에는 필수 라이선스 필드가 포함되어 있습니다.
Rust 프로젝트에 대한 일반적인 선택인
컴파일러 자체에도 사용되는 동일한 라이선스를 참조합니다.
또한 `README.md` 파일을 참조합니다.
프로젝트에 대한 간략한 설명을 포함해야 하며,
crates.io 페이지뿐만 아니라 GitHub에서 기본적으로 레포지토리 페이지에 표시됩니다.

</aside>

[crates.io]: https://crates.io/
[crates.io 계정 페이지]: https://crates.io/me
[게시 가이드]: https://doc.rust-lang.org/1.39.0/cargo/reference/publishing.html
[cargo의 manifest 형식]: https://doc.rust-lang.org/1.39.0/cargo/reference/manifest.html

### crates.io에서 바이너리를 설치하는 방법

crates.io에 crate를 게시하는 방법을 알아보았습니다.
그리고 설치하는 방법에 대해 궁금하실 수 있습니다.
라이브러리와 달리,
`cargo build` (또는 유사한 명령어)를 실행하면 cargo가 다운로드하고 컴파일해 줄 것입니다.
하지만 바이너리를 설치하려면 명시적으로 알려줘야 합니다.

이는 `cargo install <crate-name>`을 사용하여 수행됩니다.
기본적으로 crate를 다운로드하고,
그 안에 있는 모든 바이너리 대상을 컴파일합니다.
( "release" 모드이므로 잠시 걸릴 수 있습니다)
그리고 `~/.cargo/bin/` 디렉토리로 복사합니다.
(셸이 바이너리를 찾을 수 있도록 설정해야 합니다!)

또한 git 저장소에서 crate를 설치하거나,
crate의 특정 바이너리만 설치하거나,
설치할 대체 디렉토리를 지정할 수 있습니다.
자세한 내용은 `cargo install --help`를 참조하십시오.

### 언제 사용해야 할까요?

`cargo install`은 바이너리 크레이트를 설치하는 간단한 방법입니다.
Rust 개발자에게는 매우 편리하지만,
몇 가지 중요한 단점이 있습니다.

원본 코드를 항상 처음부터 컴파일하기 때문에,
도구를 사용하는 사용자는 Rust, cargo, 그리고 프로젝트가 필요로 하는 모든 시스템 의존성을
컴퓨터에 설치해야 합니다.

대형 Rust 코드베이스를 컴파일하는 것도 시간이 걸릴 수 있습니다.

다른 Rust 개발자를 대상으로 하는 도구를 배포하는 경우에 가장 적합합니다.
예를 들어:
`cargo-tree` 또는 `cargo-outdated`와 같은 많은 cargo 서브명은
이를 사용하여 설치할 수 있습니다.

## 바이너리 배포

Rust는 기본적으로 모든 의존성을 정적 연결하는 네이티브 코드로 컴파일되는 언어입니다.
`cargo build`를 사용하여 `grrs`라는 바이너리를 포함하는 프로젝트를 빌드하면 `grrs`라는 바이너리 파일이 생성됩니다.

시도해보세요:
`cargo build`를 사용하면 `target/debug/grrs`가 되고, `cargo build --release`를 실행하면 `target/release/grrs`가 됩니다.
타겟 시스템에 외부 라이브러리가 설치되어 있어야 하는 crate을 사용하지 않는 한(예: OpenSSL의 시스템 버전 사용)
이 바이너리는 일반 시스템 라이브러리에만 의존합니다.
즉,
그 한 파일을 가져와 같은 운영 체제를 사용하는 사람들에게 보내면
그들은 바로 실행할 수 있습니다.

이것은 이미 매우 강력합니다!
`cargo install`에서 보았던 두 가지 단점을 해결합니다.
사용자 머신에 Rust가 설치되어 있어야 하는 필요성이 없으며,
컴파일하는 데 1분이 걸리는 대신
즉시 바이너리를 실행할 수 있습니다.

따라서 우리가 보았듯이,
`cargo build`는 이미 바이너리를 빌드합니다.
유일한 문제는
모든 플랫폼에서 작동할 수 있는지 보장되지 않는다는 것입니다.
Windows 머신에서 `cargo build`를 실행하면
기본적으로 Mac에서 작동하는 바이너리가 생성되지 않습니다.
모든 흥미로운 플랫폼에 대해
이러한 바이너리를 자동으로 생성하는 방법이 있을까요?

### CI에서 이진 파일 릴리스 빌드

만약 프로젝트가 오픈 소스이며 GitHub에 호스팅된다면,
무료 CI(지속적 통합) 서비스를 설정하는 것이 매우 쉽습니다.
[Travis CI]와 같이.
(다른 플랫폼에서도 작동하는 다른 서비스들이 있지만, Travis는 매우 인기가 많습니다.)
이는 각각의 변경 사항을 리포지토리에 푸시할 때 가상 머신에서 설정 명령을 실행하는 것입니다.
어떤 명령어가 실행되고, 어떤 종류의 머신에서 실행되는지는 구성 가능합니다.
예를 들어:
`cargo test`를 Rust와 몇 가지 일반적인 빌드 도구가 설치된 머신에서 실행하는 것은 좋은 생각입니다.
만약 이것이 실패하면,
최근 변경 사항에 문제가 있다는 것을 알 수 있습니다.

[Travis CI]: https://travis-ci.com/

우리는 이를 사용하여 이진 파일을 빌드하고 GitHub에 업로드할 수도 있습니다!
사실, 만약
`cargo build --release`를 실행하고 이진 파일을 어딘가 업로드한다면,
모두 준비가 되어 있을 것입니다, 맞지 않나요?
아니요.
우리가 빌드하는 이진 파일이 가능한 한 많은 시스템과 호환되는지 확인해야 합니다.
예를 들어,
Linux에서는 현재 시스템을 대상으로 빌드하는 대신,
`x86_64-unknown-linux-musl` 타겟을 사용하여 기본 시스템 라이브러리에 의존하지 않도록 할 수 있습니다.
macOS에서는 `MACOSX_DEPLOYMENT_TARGET`를 `10.7`로 설정하여 10.7 버전 이상에서만 존재하는 시스템 기능에만 의존하도록 할 수 있습니다.

Linux와 macOS를 위한 이진 파일을 빌드하는 방법의 한 예는
[여기][wasm-pack-travis]와 Windows를 위한 [여기][wasm-pack-appveyor](AppVeyor를 사용)를 참조하십시오.

[wasm-pack-travis]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.travis.yml#L74-L91
[wasm-pack-appveyor]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.appveyor.yml

또는, 필요한 모든 도구가 포함된 미리 빌드된 (Docker) 이미지를 사용하는 방법도 있습니다.
이를 통해 더욱 특이한 플랫폼을 쉽게 대상으로 할 수도 있습니다.
[trust] 프로젝트에는
프로젝트에 포함할 수 있는 스크립트와 이를 설정하는 방법에 대한 지침이 포함되어 있습니다.
Windows를 위한 AppVeyor 지원도 포함되어 있습니다.

만약 로컬에서 설정하고 자신의 머신에서 릴리스 파일을 생성하는 것을 선호한다면,
여전히 trust를 살펴보세요.
이는 [cross]를 내부적으로 사용하며,
cargo와 유사하게 작동하지만 Docker 컨테이너 내부의 cargo 프로세스로 명령을 전달합니다.
이미지의 정의 또한
[cross] 레포지토리에 있습니다.

[trust]: https://github.com/japaric/trust
[cross]: https://github.com/rust-embedded/cross

### 이러한 바이너리 설치 방법

사용자를 [이와 같은 방식][wasm-pack-release]으로 출시 페이지로 안내합니다.
그리고 그들은 우리가 방금 만든 아티팩트를 다운로드할 수 있습니다.
우리가 방금 생성한 출시 아티팩트는 특별한 것이 아닙니다.
결국, 그들은 우리의 바이너리가 포함된 아카이브 파일일 뿐입니다!
이 덕분에 도구의 사용자는
웹 브라우저로 다운로드하여
(종종 자동으로) 압축 해제하고 바이너리를 원하는 위치로 복사할 수 있습니다.

[wasm-pack-release]: https://github.com/rustwasm/wasm-pack/releases/tag/v0.5.1

이렇게 하려면 프로그램을 수동으로 "설치"하는 데 대한 몇 가지 경험이 필요하므로
README 파일의 섹션에 이 프로그램을 설치하는 방법을 추가해야 합니다.

<aside class="note">

**참고:**
[trust]를 사용하여 바이너리를 빌드하고 GitHub 출시에 추가한 경우,
사람들에게 다음과 같은 명령을 실행하도록 알릴 수도 있습니다.
`curl -LSfs https://japaric.github.io/trust/install.sh | sh -s -- --git your-name/repo-name`
만약 그렇게 하면 더 쉬울 것이라고 생각한다면.

</aside>

### 언제 사용할까요?

이진 릴리스를 제공하는 것은 일반적으로 좋은 생각입니다.
단점이 거의 없습니다.
사용자가 수동으로
설치하고 업데이트해야 하는 문제는 해결하지는 않지만,
Rust를 설치하지 않고도 최신 릴리스 버전을 쉽게 가져올 수 있습니다.

### 바이너리 외에 패키지에 포함할 항목

지금은,
사용자가 우리의 릴리즈 빌드를 다운로드하면,
`.tar.gz` 파일을 받습니다.
이 파일에는 바이너리 파일만 포함되어 있습니다.
따라서 우리의 예제 프로젝트의 경우,
사용자는 실행할 수 있는 `grrs` 파일 하나만 받게 됩니다.
하지만 우리 리포지토리에 이미 있는 몇 가지 다른 파일도 사용자에게 유용할 수 있습니다.
예를 들어, 이 도구를 사용하는 방법을 설명하는 README 파일과 라이선스 파일.
우리가 이미 가지고 있기 때문에 쉽게 추가할 수 있습니다.

명령줄 도구의 경우 특히 의미 있는 몇 가지 파일이 있습니다.
README 파일 외에도 man 페이지를 포함하고, 쉘에서 가능한 플래그의 완성을 추가하는 구성 파일을 포함하는 것은 어떨까요?
수동으로 작성할 수 있지만, 우리가 사용하는 인수 분석 라이브러리인 _clap_ (clap가 기반을 둔 라이브러리)
은 이러한 모든 파일을 자동으로 생성하는 방법을 제공합니다.
자세한 내용은 [이 심층적인 챕터](../in-depth/docs.html)를 참조하십시오.


[clap-man-pages]: ../in-depth/docs.html


## 패키지 저장소에 앱을 배포하는 방법

지금까지 살펴본 두 가지 방법은 일반적으로 컴퓨터에 소프트웨어를 설치하는 방법이 아닙니다.
특히 명령줄 도구는 대부분의 운영 체제에서 글로벌 패키지 관리자를 사용하여 설치합니다.
사용자에게는 다음과 같은 장점이 있습니다.
* 프로그램 설치 방법에 대해 생각할 필요가 없습니다. 다른 도구와 동일한 방식으로 설치할 수 있기 때문입니다.
* 패키지 관리자는 새로운 버전이 출시되면 사용자가 프로그램을 업데이트할 수 있도록 해줍니다.

하지만 다양한 시스템을 지원하려면 각 시스템의 작동 방식을 이해해야 합니다.
일부 시스템에서는 저장소에 파일을 추가하는 것만으로 충분할 수 있습니다(예: macOS의 `brew`를 위한 Formula 파일을 추가하는 것과 같이 [이것][rg-formula]), 다른 시스템에서는 패치를 직접 보내서 도구를 저장소에 추가해야 할 수도 있습니다.
[cargo-bundle](https://crates.io/crates/cargo-bundle), [cargo-deb](https://crates.io/crates/cargo-deb), [cargo-aur](https://crates.io/crates/cargo-aur)와 같은 유용한 도구가 있지만, 이러한 도구가 작동하는 방식과 다양한 시스템에 대해 도구를 올바르게 패키징하는 방법에 대한 설명은 이 장의 범위를 벗어납니다.

[rg-formula]: https://github.com/BurntSushi/ripgrep/blob/31adff6f3c4bfefc9e77df40871f2989443e6827/pkg/brew/ripgrep-bin.rb

대신, Rust로 작성되어 여러 패키지 관리자에서 사용 가능한 도구를 살펴보겠습니다.

### 예시: ripgrep

[ripgrep]는 `grep`/`ack`/`ag`의 대안이며 Rust로 작성되었습니다.
꽤 성공적이며 많은 운영 체제에 패키지화되어 있습니다:
README의 "설치" 섹션을 참조하세요!

참고로, 설치 방법에 대한 몇 가지 다른 옵션이 나열되어 있습니다:
GitHub에서의 리리즈 링크를 통해 바이너리를 직접 다운로드할 수 있습니다;
다양한 패키지 관리자를 사용하여 설치하는 방법이 나열되어 있습니다;
마지막으로 `cargo install`을 사용하여 설치할 수도 있습니다.

이것은 매우 좋은 생각처럼 보입니다:
여기서 제시된 접근 방식 중 하나를 선택하지 않고,
`cargo install`로 시작하여,
바이너리 리리즈를 추가하고,
마지막으로 시스템 패키지 관리자를 사용하여 도구를 배포하십시오.

[ripgrep]: https://github.com/BurntSushi/ripgrep
[rg-install]: https://github.com/BurntSushi/ripgrep/tree/31adff6f3c4bfefc9e77df40871f2989443e6827#installation
