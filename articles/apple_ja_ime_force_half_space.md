---
title: "KarabinerでmacOSの日本語IMEのスペースキーを常に半角化する"
emoji: "⌨️"
type: "tech"
topics: ["Karabiner", "macOS", "IME"]
published: true
---

このruleをComplex Modificationsに追加する。`Shift + Space`で半角スペースが入力可能なので、`Space`を押したらそれが入力されるようにしている。

```json
{
  "description": "全角スペース->半角スペース",
  "manipulators": [
    {
      "type": "basic",
      "conditions": [
        {
          "input_sources": [
            {
              "language": "ja"
            }
          ],
          "type": "input_source_if"
        }
      ],
      "from": {
        "key_code": "spacebar"
      },
      "to": [
        {
          "key_code": "spacebar",
          "modifiers": [
            "left_shift"
          ]
        }
      ]
    }
  ]
}
```

今の所問題なく動いているが、もしかすると想定外の挙動があるかもしれない。
