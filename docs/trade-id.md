# Deterministic TradeId contract

TradeId is the shared identity that joins structured trade data, video metadata, final video files, database rows, and the review application.

TradeStatistics and TradeVideoCapture independently compute:

```text
SHA256(
    Account
    + "|"
    + SecurityId
    + "|"
    + EntryTimeUtcMs
    + "|"
    + Direction
)
```

`EntryTimeUtcMs` is the UTC Unix timestamp in milliseconds. `Direction` is exactly `Long` or `Short`. The first 16 hash bytes are rendered as 32 lowercase hexadecimal characters.

The required invariant is:

```text
TradeStatistics CSV TradeId
= TradeVideoCapture JSON trade_id
= MP4 basename and identity
```

The statistics importer preserves the same 128 bits as a database UUID. The video importer matches an existing trade only by this identity before uploading and linking its recording.

Changing the algorithm affects every producer and consumer and may orphan existing data or videos. Treat any change as a cross-project architecture and data-migration decision.
