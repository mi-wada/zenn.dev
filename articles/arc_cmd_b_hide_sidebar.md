---
title: "KarabinerでArcブラウザのサイドバーをCmd-Bで閉じられるようにする"
emoji: "👾"
type: "tech"
topics: ["Karabinar", "Arc", "macOS"]
published: true
---

Arcのサイドバーを閉じるショートカットをVSCodeと同じCmd-Bに割り当てたかったが、現在Cmd-Bを直接設定できない[^1]。そのため、Karabinerを使って無理やり設定した。

[^1]: <https://www.reddit.com/r/ArcBrowser/comments/19dv059/setting_commandb_as_a_shortcut/>

このruleをComplex Modificationsに追加すれば良い。

```json
{
  "description": "Arc / Remap Cmd-B to Cmd-S",
  "manipulators": [
    {
      "conditions": [
        {
          "bundle_identifiers": [
            "^company\\.thebrowser\\.Browser$"
          ],
          "type": "frontmost_application_if"
        }
      ],
      "from": {
        "key_code": "b",
        "modifiers": {
          "mandatory": [
            "command"
          ]
        }
      },
      "to": [
        {
          "key_code": "s",
          "modifiers": [
            "command"
          ]
        }
      ],
      "type": "basic"
    }
  ]
}
```
