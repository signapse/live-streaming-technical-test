# live-streaming-technical-test


## Low-Latency Live HLS Playlist Orchestrator

### Goal

Build a simplified backend service that manages live HLS playlists for a stream.
You are not encoding video.
You are orchestrating segments and optimizing for low latency.

Language: Go or Rust preferred.

Expected time: 3–5 hours.

## Context

We ingest live video, transcode it into segments, and deliver it via CDN (e.g. Amazon CloudFront).
Segments are continuously produced by a transcoder and sent to your service as metadata events.

Your job is to:

- Accept segment metadata
- Maintain a sliding live window
- Generate a valid HLS live playlist
- Handle concurrency safely
- Think about latency implications
- You do not need to implement encoding.

## Requirements

### 1 Register Segment

```bash
POST /streams/{stream_id}/renditions/{rendition}/segments
```

```bash
{
  "sequence": 42,
  "duration": 2.0,
  "path": "/segments/42.ts"
}
```

Requirements:
- Segments may arrive out of order.
- Duplicate sequence numbers must not corrupt state.
- State must be concurrency-safe.
- Sliding window size must be configurable (default: 6 segments).
- Assume multiple renditions (e.g., 720p, 480p).


### 2 Serve Live Playlist

```bash
GET /streams/{stream_id}/renditions/{rendition}/playlist.m3u8
```

Return a valid live HLS playlist using the sliding window.

Example:

```bash
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:2
#EXT-X-MEDIA-SEQUENCE:37

#EXTINF:2.0,
/segments/38.ts
#EXTINF:2.0,
/segments/39.ts
```

Must:
- Correct #EXT-X-MEDIA-SEQUENCE
- Correct #EXT-X-TARGETDURATION
- Only publish contiguous segments
- Avoid exposing gaps

### 3 End Stream

```bash
POST /streams/{stream_id}/end
```

After ending:
- Add #EXT-X-ENDLIST
- Reject new segments

## Latency Component (Important)
Assume:
- Segment duration = 2 seconds
- Sliding window = 6 segments

Answer the following in a short README section:
- Roughly what is the minimum achievable live latency with this configuration?
- What changes would reduce latency?
- What trade-offs would those changes introduce?

##  Bonus 
- Unit tests
- Structured logging
- Graceful shutdown
- Dockerfile
- Basic metrics endpoint
- Simple in-memory persistence abstraction
