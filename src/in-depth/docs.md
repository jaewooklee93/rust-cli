# CLI 앱에 대한 문서 렌더링

CLI의 문서는 일반적으로 명령어의 `--help` 섹션과 매뉴얼 (`man`) 페이지로 구성됩니다.

`clap`](https://crates.io/crates/clap)을 사용할 때, `clap_mangen`](https://crates.io/crates/clap_mangen) crate를 통해 자동으로 생성할 수 있습니다.

```rust,ignore
#[derive(Parser)]
pub struct Head {
    /// file to load
    pub file: PathBuf,
    /// how many lines to print
    #[arg(short = "n", default_value = "5")]
    pub count: usize,
}
```

두 번째로, 컴파일 시간에 코드 정의에서 수동 파일을 생성하기 위해 `build.rs`를 사용해야 합니다.

몇 가지 주의 사항이 있지만 (예: 바이너리 패키징 방식) 현재는 `man` 파일을 `src` 폴더 옆에 놓습니다.

```rust,ignore
use clap::CommandFactory;

#[path="src/cli.rs"]
mod cli;

fn main() -> std::io::Result<()> {
    let out_dir = std::path::PathBuf::from(std::env::var_os("OUT_DIR").ok_or_else(|| std::io::ErrorKind::NotFound)?);
    let cmd = cli::Head::command();

    let man = clap_mangen::Man::new(cmd);
    let mut buffer: Vec<u8> = Default::default();
    man.render(&mut buffer)?;

    std::fs::write(out_dir.join("head.1"), buffer)?;

    Ok(())
}
```

이제 애플리케이션을 컴파일하면
프로젝트 디렉토리에 `head.1` 파일이 생성됩니다.

그 파일을 `man`으로 열면
자유롭게 사용할 수 있는 문서를 볼 수 있습니다.
