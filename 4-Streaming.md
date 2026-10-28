# Streaming

> Streaming is a way of delivering video and audio media do users over the internet

- Playback can start before the entire file is downloaded
- The data is sent in chunks.

## Types of streaming
On-demand streaming: pre-recorded content is stored on remote servers (netflix, youtube)
Live-streaming: content is created and transmitted in realtime (twitch, online lectures)


## Protocols
TPC is more reliable, UDP is faster. Some streaming protocols use UDP for this reason.

### Example: HTTP Live Streaming

Developed by Apple.

The server prepares the video:
1. Original video is encoded
2. Split into short segments (typically a few seconds)
3. Multiple version per resolution are created (e.g. 480p, 720p, 1080p)
4. A playlist/index records the order of the segments

CDN distributes the video
- Segemnts are distributed geogrpahically with CDN servers
- popular segments can be cached closer to users
- client requests segments using standard http

Client
- Uses playlist to request segments
- playback starts without downloading entire video
- Client can adapt video quality based on network conditions.



HLS uses TCP, with some exceptions. Adaptive bitrate streaming helps compensace slower data delivery of TCP. HLS does not need to be strictlyh real time, a few seconds delay does not impact user experience as much as missing compelte frames (with UDP).

Adaptibe bitrate works by having a playback buffer (introducing the few seconds lag). This prevents the user noticing shorts hiccups in hte TCP connection. when the buffer occupancy is too low, the bitrate is adapted to a lower bitrate, i.e. the quality is degraded so that the stream can continue without buffering. When ti occupancy is sufficient, it attempts a higher bitrate.


## ELK
Elasticsearch is a distributed search and analyctics engine built on Apache Lucene, support variaous languages and scheam-free JSON docs, high performance.

Logstash is an open0soruce data ingenstion tool:
- to collection data from various sources
- transform it,
- and send it to desired destination

Kibana is a data visualization and exploration tool used for:
- Log and time-series analytics
- Application monitoring
- Operantion intelligence use cases

## Multi-region kafka, Active/active consumption

Some sync job (e.g. uReplicator) asychronously replicates message from regional kafka in one region, to an aggregate kafka in the other. The offset manager keeps track of which messages have been sent to which consumer, to prevent duplicate consumptions. It enforces an at-least-once consumed pattern. Duplicates may still occur. Each message has an offset, and the consumer notifies the aggregate server until which offset has been processed. This is writen in the active-active database. This is replicated to other regions. Exactly once, should be enforce in the application layer.
