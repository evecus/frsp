# frsp

**Fast RuleSet for Proxy** — 高性能域名 / IP / 端口规则集编译与匹配库。

支持从以下格式编译：

- 纯文本规则（domain / domain-suffix / domain-keyword / domain-regex / ip-cidr / port 等）
- AdGuard 过滤列表
- mihomo / Clash `rule-providers` YAML
- sing-box JSON 规则

编译后可序列化为紧凑的二进制格式（**`.frs`**，基于 [FST](https://docs.rs/fst)），支持 mmap 零拷贝加载，适合 geosite / geoip 等大规模规则集。

仓库：[fast_ruleset_for_proxy](https://github.com/YOUR_USERNAME/fast_ruleset_for_proxy)

## 快速开始

```toml
[dependencies]
frsp = "0.1"
```

```rust
use frsp::{CompiledRuleSet, LoadedRuleSet, MatchTarget, RuleSet};

fn main() -> frsp::Result<()> {
    // 从文本编译
    let compiled = CompiledRuleSet::from_text(
        r#"
        domain:example.com
        domain-suffix:google.com
        ip-cidr:10.0.0.0/8
        "#,
    )?;

    // 建立匹配引擎
    let loaded = LoadedRuleSet::from_compiled(compiled)?;
    let rs = RuleSet::from_loaded(loaded)?;

    assert!(rs.matches(&MatchTarget::Domain("www.google.com")));
    assert!(rs.matches(&MatchTarget::Ip("10.1.2.3".parse().unwrap())));
    assert!(!rs.matches(&MatchTarget::Domain("example.org")));

    Ok(())
}
```

## 主要 API

| 类型 | 作用 |
|------|------|
| `CompiledRuleSet` | 从文本 / JSON / YAML / AdGuard 编译出的中间表示 |
| `LoadedRuleSet` | 从二进制 `.frs` 或 `CompiledRuleSet` 加载的原始数据 |
| `RuleSet` | 最终匹配引擎（FST + Aho-Corasick + RegexSet + 区间树） |
| `MatchTarget` | 查询目标：`Domain` / `Ip` / `Port` |
| `AdGuardConvertReport` | AdGuard 列表转换结果 |

## 二进制格式 `.frs`

- Magic: `FRST`
- 基于 FST 存储域名精确匹配与后缀匹配，体积小、查询快
- 支持 mmap 零拷贝加载本地文件

## License

MIT OR Apache-2.0
