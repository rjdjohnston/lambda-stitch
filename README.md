# lambda-stitch

A Lambda function that utilizes the `@eyevinn/hls-splice` library to stitch in another HLS VOD (e.g. ads) into an HLS VOD.

## POST `/stitch/`

Example body (application/json):

```
{
  "uri": "https://maitv-vod.lab.eyevinn.technology/VINN.mp4/master.m3u8",
  "breaks": [
    { "pos": 0, "duration": 15000, "url": "https://maitv-vod.lab.eyevinn.technology/ads/apotea-15s.mp4/master.m3u8" }
  ]
}
```

The above will generate a new HLS VOD where an ad is placed at position 0 in the generated VOD (pre-roll). The request will return the following JSON:

```
{
    "uri": "/stitch/master.m3u8?payload=ewogICAgICAidXJpIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvVklOTi5tcDQvbWFzdGVyLm0zdTgiLAogICAgICAiYnJlYWtzIjogWwogICAgICAgIHsgInBvcyI6IDAsICJkdXJhdGlvbiI6IDE1MDAwLCAidXJsIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvYWRzL2Fwb3RlYS0xNXMubXA0L21hc3Rlci5tM3U4IiB9CiAgICAgIF0KfQ=="
}
```

## HLS Interstitial

Instead of stitching in the ad on the server side this Lambda function can return an HLS with DATERANGE interstitial with an assetlist URI pointing back to the Lambda. Add the query param `i=1` to use interstitial:

```
/stitch/master.m3u8?payload=ewogICAgICAidXJpIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvVklOTi5tcDQvbWFzdGVyLm0zdTgiLAogICAgICAiYnJlYWtzIjogWwogICAgICAgIHsgInBvcyI6IDAsICJkdXJhdGlvbiI6IDE1MDAwLCAidXJsIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvYWRzL2Fwb3RlYS0xNXMubXA0L21hc3Rlci5tM3U4IiB9CiAgICAgIF0KfQ==&i=1
```

The resulting media playlist will then for example contain the following DATERANGE tag:

```
#EXT-X-DATERANGE:ID="1",CLASS="com.apple.hls.interstitial",START-DATE="1970-01-01T00:00:00.000Z",X-ASSET-LIST="/stitch/assetlist/eyJhc3NldHMiOlt7InVyaSI6Imh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvYWRzL2Fwb3RlYS0xNXMubXA0L21hc3Rlci5tM3U4IiwiZHVyIjoxNX1dfQ%3D%3D"
```

The assetlist URI contains an escaped base64 encoded payload containing the list of ads to be inserted, and the assetlist endpoint returns a JSON according to the X-ASSET-LIST attribute in the HLS Interstitial specification: https://developer.apple.com/streaming/GettingStartedWithHLSInterstitials.pdf

To both insert HLS interstitial tags and splice in the ad segments you can provide the query param `c=1`:

```
/stitch/master.m3u8?payload=ewogICAgICAidXJpIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvVklOTi5tcDQvbWFzdGVyLm0zdTgiLAogICAgICAiYnJlYWtzIjogWwogICAgICAgIHsgInBvcyI6IDAsICJkdXJhdGlvbiI6IDE1MDAwLCAidXJsIjogImh0dHBzOi8vbWFpdHYtdm9kLmxhYi5leWV2aW5uLnRlY2hub2xvZ3kvYWRzL2Fwb3RlYS0xNXMubXA0L21hc3Rlci5tM3U4IiB9CiAgICAgIF0KfQ==&c=1
```
