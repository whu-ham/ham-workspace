# Promotion card — cloud-config (CCKV) contract

The `promotion` CCKV key drives the promotion card on the **My** tab of the iOS and
Android clients. Both clients read the same JSON payload and implement the same
fallback rules.

## Payload

```json
{
  "tabList": [
    {
      "title-content": { "zh": "推荐", "en": "Featured", "ja": "おすすめ" },
      "itemList": [
        {
          "iconUrl": "https://example.com/icon.png",
          "url": "https://example.com/target",
          "title-content": { "zh": "标题", "en": "Title" },
          "subtitle-content": { "zh": "副标题", "en": "Subtitle" }
        }
      ]
    }
  ],
  "itemList": []
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `tabList` | array of tab objects | Optional. Omit for the flat vertical list. |
| `tabList[].title-content` | map of locale to string | Supported locales: `zh` (default), `en`, `ja`. |
| `tabList[].itemList` | array of item objects | The entries shown while that tab is selected. |
| `itemList` | array of item objects | The flat list, used when `tabList` is absent or unusable. |
| `itemList[].iconUrl` | string | Rendered as a 30dp/30pt circle. |
| `itemList[].url` | string | Opened externally on tap. |
| `itemList[].title-content` | map of locale to string | Primary label. |
| `itemList[].subtitle-content` | map of locale to string | Secondary label. |

An item's tab is determined by **position**, not by a reference field: an item
listed under `tabList[1].itemList` belongs to the second tab. There is no
`tabId`, so an item cannot appear under two tabs without being duplicated in the
payload.

## Layout rules

Both clients apply these in order:

1. Tabs with no usable items are dropped.
2. **Two or more tabs remain** → render the tab bar. The bar scrolls
   horizontally and never wraps.
3. **Exactly one tab remains** → ignore the bar and render that tab's items as a
   vertical list. A one-entry tab bar is noise, and dropping the items would hide
   content.
4. **No usable tabs** → render the top-level `itemList` as a vertical list. This
   is the pre-tab layout, so every payload written before this change is
   unaffected.

A payload that yields no items at all renders nothing — the card is hidden.

## Backward compatibility

Payloads that contain only `itemList` keep working unchanged. No server-side
change is required to adopt this contract: operators add `tabList` at their own
pace, and clients that have not been updated yet ignore the unknown field and
keep showing the flat list.

## Locale fallback

A localized string resolves in this order: the user's locale → `zh` → any
non-empty translation. A missing key never renders as an empty label, and a
single malformed entry does not discard the whole card.

## Event tracking

Taps report `promotion_btn` with `title`, `subtitle`, and `url`. Tab layouts
additionally report `tab`, the title of the selected tab. Flat layouts omit
`tab`, so existing dashboards are unaffected.

## Implementations

- Android: `MyPromotionParser.kt`, `MyPromotionView.kt`, `MyPromotionTabRow.kt`
  (`android/feature/my/.../ui/component/promotion/`)
- iOS: `Ham/iOS/ui/my/component/card/MyViewPromotionCard.swift`
