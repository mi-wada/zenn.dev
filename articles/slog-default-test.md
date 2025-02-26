---
title: "slog.Info, slog.Errorなどの出力をテストする"
emoji: "🪵"
type: "tech"
topics:
  - "Go"
  - "slog"
  - "Test"
published: true
---

## はじめに

`slog.Info`, `slog.Warn`, `slog.Error`などの`slog.Default()`を用いてロギングする以下のような関数のテストを書く。

```go
func myFunc() {
  slog.Info("myFunc called", slog.String("key", "value"))
}
```

## 結論

このようなテストヘルパー関数を用いる。テストを並列実行する場合を考慮してロックを取る。

```go
package testutil

var mu sync.Mutex

func CaptureSlog(t *testing.T, f func()) []byte {
  t.Helper()

  mu.Lock()
  defer mu.Unlock()
  original := slog.Default()

  var buf bytes.Buffer
  logger := slog.New(slog.NewJSONHandler(&buf, nil))

  slog.SetDefault(logger)
  defer slog.SetDefault(original)

  f()

  return buf.Bytes()
}
```

このように使う。

```go
func myFunc() {
  slog.Info("myFunc called", slog.String("key", "value"))
}

func TestMyFunc(t *testing.T) {
  t.Parallel()

  for i := range 2 {
    t.Run(fmt.Sprintf("%d", i), func(t *testing.T) {
      t.Parallel()

      logBytes := testutil.CaptureSlog(t, func() {
        myFunc()
      })

      var m map[string]interface{}
      if err := json.Unmarshal(logBytes, &m); err != nil {
        t.Fatalf("json.Unmarshal() = %v; want nil", err)
      }

      if got, want := m["msg"], "myFunc called"; got != want {
        t.Errorf("msg = %v; want %v", got, want)
      }
      if got, want := m["level"], slog.LevelInfo.String(); got != want {
        t.Errorf("level = %v; want %v", got, want)
      }
      if got, want := m["key"], "value"; got != want {
        t.Errorf("key = %v; want %v", got, want)
      }
    })
  }
}

```

## おわりに

`testutil.CaptureSlog`を通さずに`slog.SetDefault`が使われると並列実行時に落ちるので、そのあたりはLinterなどでチェックすると良い。
