# MediaMelon JS Custom Multi-Player SDK Integration Guide

This guide is for integrating the MediaMelon Custom Multi-Player SDK for JavaScript-based web players.

- **Step 1:** [Add MediaMelon Player SDK](#step-1-add-mediamelon-sdk)
- **Step 2:** [Instantiate and Register SDK](#step-2-instantiate-and-register-sdk)
- **Step 3:** [Initialize Playback Session](#step-3-initialize-playback-session)
- **Step 4:** [Custom Metadata](#step-4-custom-metadata)
- **Step 5:** [Stream and Network Information](#step-5-stream-and-network-information)
- **Step 6:** [Chunk/Segment Information](#step-6-chunksegment-information)
- **Step 7:** [Player Events](#step-7-player-events)
- **Step 8:** [Fallback & Request Status](#step-8-fallback--request-status)
- **Step 9:** [Ad Data & Ad Events](#step-9-ad-data--ad-events)
- **Step 10:** [Custom Events](#step-10-custom-events)
- [Release Notes](#release-notes)

## Step 1: Add MediaMelon SDK

**NPM (2.2.2):**

```bash
npm i mediamelon-js-custom-sdk
```

```javascript
import { mmJSCustomAdapter, MMPlayerState, RenditionInfo, RequestStatus, MMAdEvent } from 'mediamelon-js-custom-sdk';
```

## Step 2: Instantiate and Register SDK

> **Note:** `CUSTOMER_ID` is your MediaMelon assigned Customer ID. If you do not know your Customer ID, contact MediaMelon at customer-support@mediamelon.com.

### Step 2.1: Instantiate and Report Registration Information

```javascript
var mmJSPlugin = new mmJSCustomAdapter();
mmJSPlugin.registerMMSmartStreaming(
    "PLAYER_NAME",
    "CUSTOMER_ID",
    "SUBSCRIBER_ID",
    "DOMAIN_NAME",
    "SUBSCRIBER_TYPE",
    "SUBSCRIBER_TAG",
    hashSubscriberId    // Boolean Value
);
```

> **Note:** `hashSubscriberId`: Set it to `true` to hash the subscriber ID, and to `false` to process the subscriber ID without hashing.

### Step 2.2: Report Player Information

```javascript
mmJSPlugin.reportPlayerId("PLAYER_ID");
mmJSPlugin.reportPlayerInfo("PLAYER_BRAND", "PLAYER_MODEL", "PLAYER_VERSION");
mmJSPlugin.reportBasePlayerInfo("BASE_PLAYER_NAME", "BASE_PLAYER_VERSION");
```

### Step 2.3: Report Application Information

```javascript
mmJSPlugin.reportAppInfo("APPLICATION_NAME", "APPLICATION_VERSION");
mmJSPlugin.reportAppSessionId("APP_SESSION_ID");
```

### Step 2.4: Report Device Information

```javascript
var deviceInfo = {
    "deviceName": "DEVICE_NAME",
    "deviceBrand": "DEVICE_BRAND",          // Device Brand or Manufacturer
    "deviceModel": "DEVICE_MODEL",
    "deviceId": "DEVICE_ID",
    "deviceOS": "DEVICE_OS",
    "deviceOSVersion": "DEVICE_OS_VERSION",
    "screenWidth": screen_width,            // Integer Value
    "screenHeight": screen_height,          // Integer Value
    "deviceCapabilities": "DEVICE_CAPABILITIES"
};
mmJSPlugin.reportDeviceInfo(deviceInfo);
```

### Step 2.5: Report Sub Property ID

```javascript
mmJSPlugin.reportSubPropertyId("SUB_PROPERTY_ID");
```

## Step 3: Initialize Playback Session

### Step 3.1: Initialize Session with Content Metadata

```javascript
var contentMetaData = {
    "assetName": "ASSET_NAME",
    "assetId": "ASSET_ID",
    "videoId": "VIDEO_ID",
    "contentType": "CONTENT_TYPE",
    "genre": "GENRE",
    "drmProtection": "DRM_PROTECTION",
    "drmLevel": "DRM_LEVEL",
    "episodeNumber": "EPISODE_NUMBER",
    "season": "SEASON",
    "seriesTitle": "SERIES_TITLE"
};

mmJSPlugin.initializeSession(contentMetaData, "STREAM_URL");
```

> **Warning:** All metadata fields set from **Step 3** will be reset when `initializeSession` is called. Therefore, ensure that all session-specific metadata, such as view session ID, preload, and similar fields, are reported after `initializeSession` is called for every playback session.

### Step 3.2: Report View Session ID

```javascript
mmJSPlugin.reportViewSessionId("VIEW_SESSION_ID");
```

### Step 3.3: Report Experiment Name

```javascript
mmJSPlugin.reportExperimentName("EXPERIMENT_NAME");
```

### Step 3.4: Report Preload

```javascript
mmJSPlugin.reportPreload(isPreload);    // Boolean Value
```

### Step 3.5: Report Player Resolution

```javascript
mmJSPlugin.reportPlayerResolution(width, height);   // Integer Values
```

### Step 3.6: Report Page URL

The SDK automatically records the URL of the page (`window.location.href`) when `initializeSession` is called. You don't need to do anything unless you want to report a different URL.

```javascript
mmJSPlugin.reportPageUrl("PAGE_URL");
```

> **Length limit:** Page URLs can be up to 2048 characters. Longer URLs are cut to 2048 characters and `...TRUNCATED` is added to the end.

### Step 3.7: Report User-Initiated Playback

> **Note:** This function indicates the intent for playback and must be invoked after `initializeSession`. Any metadata fields reported before this API call will be included in the first payload sent to the dashboard. To avoid `Null` values in metadata fields, it is recommended to report all available metadata before calling this API.

```javascript
mmJSPlugin.reportUserInitiatedPlayback();
```

## Step 4: Custom Metadata

Check the custom tags configuration in your [dashboard](https://smartsight3.mediamelon.com/settings) and report accordingly. If the custom tags are not configured, please configure and use them accordingly.

```javascript
mmJSPlugin.reportCustomMetadata("custom_1", "value1");
mmJSPlugin.reportCustomMetadata("custom_2", "value2");
```

## Step 5: Stream and Network Information

### Step 5.1: Report Stream Information

```javascript
mmJSPlugin.reportStreamInfo("STREAM_FORMAT", "MEDIA_TYPE", "SOURCE_TYPE");
```

### Step 5.2: Update Stream URL

```javascript
mmJSPlugin.updateStreamURL("STREAM_URL");
```

### Step 5.3: Report Presentation Info

```javascript
mmJSPlugin.reportPresentationInfo(is_live, video_duration);
```

> - `is_live`: Set to `true` for a live video stream and to `false` for a VOD stream.
> - `video_duration`: Integer value in milliseconds.

### Step 5.4: Report Track Information

Call this API with initial values, and every time there is a change in the track info — for example, when the user enables/disables subtitles or changes the audio track.

```javascript
mmJSPlugin.reportTrackinfo(is_subtitle_active, "SUBTITLE_TRACK", "AUDIO_TRACK", is_vds_active);
```

> - `is_subtitle_active`: Set it to `true` if subtitles are active; otherwise, set it to `false`.
> - `is_vds_active`: Set it to `true` if the type of audio is Virtual Dialogue Sound; otherwise, set it to `false`.

### Step 5.5: Report Rendition

At the start of the video, create a `RenditionInfo` object, assign initial rendition values to it, and report it to the SDK. For any subsequent rendition change, update only the fields that changed in the same `RenditionInfo` object, leave the unchanged fields as-is, and report the updated object to the SDK.

```javascript
var renditionInfo = new RenditionInfo();
renditionInfo.bitrate = bitrate;        // Integer Value in bps
renditionInfo.width = width;            // Integer Value
renditionInfo.height = height;          // Integer Value
renditionInfo.frameRate = frame_rate;   // Integer Value in fps
renditionInfo.aCodec = "AUDIO_CODEC";
renditionInfo.vCodec = "VIDEO_CODEC";

mmJSPlugin.reportRendition(renditionInfo);
```

### Step 5.6: Update DRM Type

> **Deprecated:** `updateDrmType(drmType)` is deprecated. Use the new API `updateDRM(drmProtection, drmLevel)` instead.

```javascript
mmJSPlugin.updateDRM("DRM_PROTECTION", "DRM_LEVEL");
```

### Step 5.7: Report Network Information

```javascript
var networkInfo = {
    "cdn": "CDN",
    "asn": asn,                             // Integer Value
    "hostName": "SOURCE_HOST_NAME",
    "networkType": "NETWORK_TYPE",          // Network Connection Type
    "networkOperator": "NETWORK_OPERATOR"
};
mmJSPlugin.reportNetworkInfo(networkInfo);
```

> **Note:** Use the `reportNetworkInfo` API to report the CDN along with other network information. If additional network information is not available, use `reportCDN` instead.

### Step 5.8: Report CDN Information

```javascript
mmJSPlugin.reportCDN("CDN");
```

### Step 5.9: Report Encoding Service

```javascript
mmJSPlugin.reportEncodingService("ENCODING_SERVICE");
```

## Step 6: Chunk/Segment Information

### Step 6.1: Report Download Rate

Report the latest chunk download rate using this method. Trigger this method for every chunk.

```javascript
mmJSPlugin.reportDownloadRate(downloadRate);    // Integer Value in bps (bits per second)
```

## Step 7: Player Events

### Step 7.1: Report Player State

```javascript
mmJSPlugin.reportPlayerState(MMPlayerState.PLAYING);
```

**Enum: `MMPlayerState`**
- `PLAYING`
- `PAUSED`
- `STOPPED`

### Step 7.2: Report Buffering

```javascript
mmJSPlugin.reportBufferingStarted();
mmJSPlugin.reportBufferingCompleted();
```

### Step 7.3: Report Seek

```javascript
mmJSPlugin.reportPlayerSeekStarted();
mmJSPlugin.reportPlayerSeekCompleted(seekEndPositionInMS);  // Integer Value in Milli Seconds
```

### Step 7.4: Report Error

```javascript
mmJSPlugin.reportError("ERROR_CODE", "ERROR_MESSAGE", "ERROR_DETAILS");
```

### Step 7.5: Report Warning

```javascript
mmJSPlugin.reportWarning("WARNING_CODE", "WARNING_MESSAGE", "WARNING_DETAILS");
```

### Step 7.6: Report Playback Position

> **Note:** Call this every 0.5 sec or 1 sec to report the playback position from the player.

```javascript
mmJSPlugin.reportPlaybackPosition(playback_position);   // Integer Value in Milli Seconds
```

## Step 8: Fallback & Request Status

### Step 8.1: Report Fallback Event

```javascript
mmJSPlugin.reportFallbackEvent("FALLBACK_MANIFEST_URL", "DESCRIPTION");
```

### Step 8.2: Report Request Status

```javascript
var requestInfo = {
    id: "ID",
    error: "ERROR",
    text: "TEXT",
    hostname: "HOSTNAME",
    url: "URL",
    // ...
};

mmJSPlugin.reportRequestStatus(RequestStatus.FAILED, "REQUEST_TYPE", requestInfo);
```

**Enum: `RequestStatus`**
- `FAILED`
- `CANCELLED`

## Step 9: Ad Data & Ad Events

### Step 9.1: Report Ad Break Start & End

> **Note:** `reportAdBreakStart` and `reportAdBreakEnd` can now be reported with an optional JSON parameter.

```javascript
var adBreakInfo = { };  // ad break information
mmJSPlugin.reportAdBreakStart(adBreakInfo);
mmJSPlugin.reportAdBreakEnd();
```

### Step 9.2: Report Ad Data, Ad Start & End

```javascript
var adInfo = {
    "adTitle": "AD_TITLE",
    "adId": "AD_ID",
    "adUniversalId": "AD_UNIVERSAL_ID",
    "adCreativeId": "AD_CREATIVE_ID",
    "adCreativeType": "AD_CREATIVE_TYPE",
    "adClient": "AD_CLIENT",
    "adPosition": "AD_POSITION",            // pre, mid, post
    "adServer": "AD_SERVER",
    "adResolution": "AD_RESOLUTION",
    "adUrl": "AD_URL",
    "adDuration": ad_duration,              // Double Value
    "adPodIndex": pod_index,                // Integer Value
    "adPositionInPod": ad_position_in_pod,  // Integer Value
    "isBumper": is_bumper                   // Boolean Value
};

mmJSPlugin.reportAdStart(adInfo);
mmJSPlugin.reportAdEnd();
```

<details>
<summary><b>AdInfo object description</b></summary>

| Field | Data Type | Description |
| --- | --- | --- |
| `adInfo.adTitle` | String | The title or name of the ad, usually provided in the VAST metadata or by the ad server. |
| `adInfo.adId` | String | A unique identifier for the ad creative, often defined by the ad server or DSP. |
| `adInfo.adUniversalId` | String | The universal ad ID of the creative. |
| `adInfo.adCreativeId` | String | The creative ID associated with the specific ad asset (video, image, etc.). Helps in tracking and reporting creative-level performance. |
| `adInfo.adCreativeType` | String | The format or type of the ad creative. For example, video/mp4, image/jpeg, etc., or linear, non-linear. |
| `adInfo.adClient` | String | The SDK or client library responsible for requesting and playing the ad. Example: Google IMA, Freewheel, SpotX. |
| `adInfo.adPosition` | String | The timing of the ad in relation to the main content: "pre" (before), "mid" (during), or "post" (after). |
| `adInfo.adServer` | String | The ad server or source that delivered the ad. Example: Google Ad Manager, Freewheel, etc. |
| `adInfo.adResolution` | String | The resolution of the ad video (e.g., 1920x1080), useful for reporting and quality monitoring. |
| `adInfo.adUrl` | String | The URL from which the ad video is fetched. Typically a media file or stream URL. |
| `adInfo.adDuration` | Integer | The total duration of the ad, in milliseconds. Example: 30000 = 30 seconds. |
| `adInfo.adPodIndex` | Integer | The index of the ad pod within the stream. Ad pods are groups of ads played together (like a commercial break). |
| `adInfo.adPositionInPod` | Integer | The position of the ad within its pod (e.g., 1st ad, 2nd ad in the group). |
| `adInfo.adPodLength` | Integer | The total number of ads in the current pod. Useful for showing "Ad 2 of 5" type of UI. |
| `adInfo.isBumper` | Boolean | Indicates whether the ad is a bumper ad (short ad, usually <6s, played at the start or end of ad breaks). |

</details>

### Step 9.3: Report Ad Buffering

Use the Ad Buffering APIs to report any buffering that occurs during ad playback. For any buffering event, either content buffering or ad buffering should be reported, but not both.

```javascript
mmJSPlugin.reportAdBufferingStarted();
mmJSPlugin.reportAdBufferingCompleted();
```

### Step 9.4: Report Ad Events

Use the `reportAdEvent` function to report all ad events.

> **Note:** `reportAdStart` and `reportAdEnd` are kept as-is for backward compatibility, but going forward, all new integrations should use the `reportAdEvent` API for all ad states. `MMAdEvent` needs to be imported by the client from the SDK.

```javascript
mmJSPlugin.reportAdEvent(MMAdEvent.AD_START, adInfo);
```

<details>
<summary><b>MMAdEvent (Enum)</b></summary>

| Ad Event | Description |
| --- | --- |
| `AD_START` | Report on ad start. |
| `AD_FIRST_QUARTILE` | Report on ad reached first quartile. |
| `AD_MIDPOINT` | Report on ad reached midpoint. |
| `AD_THIRD_QUARTILE` | Report on ad reached third quartile. |
| `AD_SKIP` | Report on ad skipped. |
| `AD_END` | Report on ad ended. |
| `AD_COMPLETE` | Report on ad ended and ad play time greater than 95%. |
| `AD_CLICK` | Report on ad clicked. |

</details>

### Step 9.5: Report Ad Errors and Warnings

Report Ad Error:

```javascript
mmJSPlugin.reportError("AD_ERROR_CODE", "AD_ERROR_MESSAGE", "AD_ERROR_DETAILS");
```

Report Ad Warning:

```javascript
mmJSPlugin.reportWarning("AD_WARNING_CODE", "AD_WARNING_MESSAGE", "AD_WARNING_DETAILS");
```

## Step 10: Custom Events

The Custom Events API can be used to report events that are not covered by the default MediaMelon-supported events. Custom events must be reported within the SDK activity lifecycle. Events reported before SDK initialization or after session end will be ignored. Please refer to the guidelines and constraints below.

```javascript
mmJSPlugin.reportCustomEvent("EVENT_NAME", "EVENT_VALUE");
```

**Guidelines & Constraints**

- **Event Name**
  - Maximum length: 35 characters
  - Allowed characters: alphanumeric and underscore (`_`) only
  - Special characters are not supported
  - Recommended format: UPPERCASE with underscores (e.g., `DRM_CHANGE`)
- **Event Limits**
  - A maximum of 30 custom events per session is allowed.

Custom events that violate these constraints (invalid event name or exceeding the per-session limit) will be dropped. Event values exceeding 1000 characters will be truncated to 1000 characters.

## Release Notes

Performance report: [JavaScript Custom SDK Performance Metrics](https://docs.mediamelon.com/mediamelon/sdk-performance-metrics/web/javascript-custom-sdk-performance-metrics)

### Current Release

#### v2.2.2 — Nov 17, 2026

- Added a Page URL dimension. The SDK picks up the page URL automatically, and clients can override it with `reportPageUrl`.

### Previous Releases

<details>
<summary>Show previous releases</summary>

#### v2.2.1 — July 13, 2026

- Fixed an internal mapping issue for payload ID.

#### v2.2.0 — July 6, 2026

- Enhanced internal SDK data reporting.
- Added batch-based data reporting with configurable reporting intervals.
- Improved reliability with exponential backoff retries, jitter, request timeouts, and Dead Letter Queue (DLQ) support.
- Enabled remote configuration for batch size, reporting interval, retry settings, and the batching feature flag.
- Added new APIs for reporting ad events, including quartiles, request/response, errors, warnings, skip, and click events.
- Added support for reporting device capabilities through DeviceInfo.

#### v2.1.1 — June 10, 2026

- Handled event reporting failure exception.

#### v2.1.0 — April 30, 2026

- Added support for multiple player instances within a single application.
- Added the following new metadata fields:
  - **Player ID** — report it using `reportPlayerId`.
  - **DRM Level** — report it as part of Content Metadata.
  - **Ad Universal ID** — report it as part of Ad Info.
- `updateDRMType(drmType)` → **Deprecated**. Use `updateDRM(drmProtection, drmLevel)` (refer to [Step 5.6](#step-56-update-drm-type)).
- Buffering events exceeding the stats interval are now included in the stats payload.
- Updated metadata lifecycle handling:
  - [Step 2](#step-2-instantiate-and-register-sdk) metadata fields persist across the SDK instance lifecycle.
  - All metadata fields from [Step 3](#step-3-initialize-playback-session) are reset for every playback session.

#### Non-Multiplayer Releases

See the [MediaMelon JS Custom SDK release notes](https://docs.mediamelon.com/mediamelon/smartsight-player-sdk-integration/web/mediamelon-js-custom-sdk-integration-document#release-notes).

</details>
