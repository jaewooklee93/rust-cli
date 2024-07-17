# 구성 파일 사용

구성 파일을 다루는 것은 특히 여러 운영 체제를 지원하는 경우
모든 운영 체제가
단기 및 장기 파일을 위한 고유한 위치를 가지고 있기 때문에
지루할 수 있습니다.

이에 대한 몇 가지 해결책이 있으며,
일부는 다른 것보다 더 저수준입니다.

이를 위해 사용하기 가장 쉬운 crate는 [`confy`]입니다.
그것은 애플리케이션 이름을 요청하고
`struct` (즉 `Serialize`, `Deserialize` 인)를 통해 구성 레이아웃을 지정하도록 요구하며
나머지는 스스로 처리합니다!

```rust,ignore
#[derive(Debug, Serialize, Deserialize)]
struct MyConfig {
    name: String,
    comfy: bool,
    foo: i64,
}

fn main() -> Result<(), io::Error> {
    let cfg: MyConfig = confy::load("my_app")?;
    println!("{:#?}", cfg);
    Ok(())
}
```

이것은 사용하기 매우 쉬운
물론 구성 가능성을 포기해야 하는 경우입니다.
하지만 간단한 구성만 원하는 경우
이 crate가 적합할 수 있습니다!

[`confy`]: https://docs.rs/confy/0.3.1/confy/

## 구성 환경

<aside class="todo">

**TODO**

1. 존재하는 crate를 평가
2. Cli-args + 여러 구성 파일 + 환경 변수
3. [`configure`]가 이 모든 것을 할 수 있나요? 둘러싼 멋진 래퍼가 있나요?

</aside>

[`configure`]: https://docs.rs/configure/0.1.1/configure/
