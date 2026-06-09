# fizz-session-tracker

Fizz の**配信セッション トラッカ**。配信の開始/終了境界を明示化し、今の配信枠で
誰がどれだけ喋ったか(per-user participation)を集計する。

「前回の配信」「この枠での常連度」を recall が出すための土台。配信終了時の要約
(`session.summary`)は L3 summarizer が後から埋める前提で、ここでは枠だけ持つ。

## I/O

stdin に 1 行 1 コマンド(NDJSON):

```jsonc
{"op":"start","persona":"lime","title":"雑談枠","ts":1000}
{"op":"participate","user_id":7,"ts":1010}
{"op":"current"}
{"op":"end","summary":"楽しかった","ts":2000}
```

- `start` → 新しい配信枠を切る(id 自動採番、参加リセット)。`{...MemorySession...}`。
- `participate` → 現配信で user の発言を 1 件加算(配信外は無視)。`{"ok":true,"user_count":N}`。
- `current` → 現配信を返す。`{...MemorySession...}` か `null`。
- `end` → 現配信を閉じる(`ended_at`/`summary` を埋める)。`{"session":{...},"participations":[...]}`。

状態はインメモリ。EOF で終了。不正な行は stderr に警告して読み飛ばす。

## 開発

```sh
almide check src/main.almd
almide test
almide build src/main.almd -o build/fizz-session-tracker
```

ツールチェーン: [almide](https://github.com/almide/almide) v0.26.6+。

## 契約

[fizz-protocol](https://github.com/Aid-On/fizz-protocol) の `memory` モジュール
(`MemorySession` / `SessionParticipation` とその Codec)に依存。
