# Twitch Community Intelligence

Public research dataset: what Twitch viewers and creators are actually saying about community features and notifications.

**Live dashboard:** [schlacter.me/twitch-community](https://schlacter.me/twitch-community)

## What is this?

1,183+ data points scraped from public sources, clustered into 10 pain-point themes structured as a funnel:

- **Layer 1 — Creator-Viewer Connection** (broad signal): Discovery, viewer retention, creator burnout, community tools, platform competition
- **Layer 2 — Notifications** (specific pain): Missed live alerts, notification fatigue, mobile push reliability, confusing settings, email spam

Every data point links to its exact source URL. Every quote is traceable. No LLMs in the data pipeline — all analysis is deterministic keyword matching.

## Data Sources

| Source | Method | Count |
|--------|--------|-------|
| Reddit | Public JSON API (`/search.json`) | 673 |
| Hacker News | Algolia API | 419 |
| Google Play Store | `google-play-scraper` npm | 190 |
| Apple App Store | `app-store-scraper` npm | 52 |
| Curated | Hand-sourced high-signal items | 20 |

**Subreddits scraped:** r/Twitch, r/streaming, r/Twitch_Startup, r/twitchstreams, r/twitchplayground

**Freshness guardrail:** All data is filtered to the last 12 months. Items without reliable dates are flagged `dateConfidence: "low"` and excluded from timeline views.

## Data Files

| File | Description |
|------|-------------|
| [`data/raw-feedback.json`](data/raw-feedback.json) | Full 1,183+ item dataset with source URLs, dates, perspective tags |
| [`data/snapshot.json`](data/snapshot.json) | Latest analysis snapshot — theme frequencies, perspective splits, monthly distribution |
| [`data/curated-feedback.json`](data/curated-feedback.json) | 20 hand-sourced high-signal items from top Reddit threads, tech press, creator tweets |
| [`data/twitch-launches.json`](data/twitch-launches.json) | Recent Twitch feature launches for timeline correlation |

## Schema: Raw Feedback Item

```json
{
  "id": "reddit-abc123",
  "source": "reddit",
  "text": "The actual feedback text...",
  "author": "username",
  "url": "https://reddit.com/r/Twitch/comments/abc123/...",
  "score": 156,
  "date": "2025-11-15T00:00:00.000Z",
  "dateConfidence": "high",
  "perspective": "creator",
  "subreddit": "Twitch"
}
```

### Fields

- **source**: `reddit` | `hackernews` | `playstore` | `appstore` | `curated`
- **score**: Reddit upvotes, HN points, App Store stars, or curated signal strength
- **dateConfidence**: `high` (reliable date from API) or `low` (estimated)
- **perspective**: `creator` (streamer feedback), `viewer` (watcher feedback), or `both` (ambiguous)

## Theme Definitions

Each feedback item is matched against themes using keyword lists. One item can match multiple themes.

### Community Layer
| ID | Name | Sample Keywords |
|----|------|----------------|
| `discovery` | Discoverability & New Streamer Discovery | discover, browse, recommend, algorithm, small streamer, directory |
| `viewer-retention` | Viewer Drop-Off & Re-Engagement | stopped watching, used to watch, lost interest, YouTube better |
| `creator-retention` | Creator Burnout & Churn | quit streaming, burnout, no viewers, not worth it, growth |
| `community-tools` | Community Building Features | raid, host, chat, emotes, clips, channel points, mod |
| `competitor-pull` | Platform Competition | YouTube, Kick, TikTok, switching, moved to, leaving Twitch |

### Notification Layer
| ID | Name | Sample Keywords |
|----|------|----------------|
| `notif-missing` | Missed Live Notifications | didn't get notified, missed stream, went live, no notification |
| `notif-fatigue` | Notification Overload & Fatigue | too many notifications, spam, turn off, annoying, overwhelming |
| `notif-mobile` | Mobile Push Reliability | push notification, phone, mobile, delayed, late, never arrives |
| `notif-settings` | Confusing Notification Controls | settings, can't find, how to, customize, where is, confusing |
| `notif-email` | Email Notification Spam | email, unsubscribe, inbox, marketing, too many emails |

## Perspective Tagging

Every item is tagged as `creator`, `viewer`, or `both` using keyword matching:

- **Creator signals**: "my stream", "my viewers", "my channel", "as a streamer", "growing my audience"
- **Viewer signals**: "I watch", "my favorite streamer", "I follow", "I missed", "as a viewer"
- **Default**: `both` if ambiguous or matches neither/both

## How to Use This Data

```javascript
// Load and filter
const data = require('./data/raw-feedback.json');

// All notification-related feedback from creators
const creatorNotifPain = data.filter(item =>
  item.perspective === 'creator' &&
  item.text.toLowerCase().includes('notification')
);

// Top Reddit posts by score
const topReddit = data
  .filter(item => item.source === 'reddit')
  .sort((a, b) => b.score - a.score)
  .slice(0, 20);
```

## Methodology

Full methodology: [schlacter.me/twitch-community/methodology](https://schlacter.me/twitch-community/methodology)

- All analysis is deterministic keyword matching — no LLMs in the pipeline
- Severity is manually assessed (1-5 scale) based on user impact
- Themes ranked by severity x frequency
- 12-month freshness guardrail applied at scraper level and as final pass

## Built With

- [Claude Code](https://claude.com/claude-code) (Anthropic) — built the entire project
- Next.js, React, Upstash Redis, Vercel

## License

This dataset is provided for research and analysis purposes. All data was scraped from publicly available sources. Each item links to its original source.
