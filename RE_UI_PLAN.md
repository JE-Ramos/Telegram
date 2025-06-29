# ProfileActivity UI Uplift Plan

This document outlines the plan to update the `ProfileActivity` UI based on the new design mock-up.

## 1. Header Redesign

The current header (`TopView` and `avatarContainer`) needs a major overhaul to match the new immersive design.

### 1.1. Large Avatar Background
- The `TopView` should be modified to display a large version of the user's profile picture as its background.
- A blur effect should be applied to this background image.
- A dark gradient overlay should be added on top of the blurred image to ensure text readability.

### 1.2. User Info Overlay
- The `nameTextView` and `onlineTextView` should be moved to be overlaid on top of the header background.
- **Name:**
    - Font size and weight need to be increased significantly.
    - Color should be white.
    - An `ImageView` for the emoji status (e.g., the unicorn emoji) should be placed next to the name. The existing `emojiStatusDrawable` can be adapted for this.
- **Status:**
    - "online" text.
    - Font size and color need to be updated to match the mockup (white, slightly smaller than the name).
- These text views need to be part of the collapsing header animation, moving and scaling correctly as the user scrolls.

### 1.3. Collapsing Behavior
- The collapsing animation needs to be adjusted. As the user scrolls up, the large header should collapse into the standard `ActionBar`.
- The profile picture should shrink into the small avatar circle in the `ActionBar`.
- The name and status text should animate and settle next to the avatar in the `ActionBar`. The existing animation logic can be adapted.

## 2. Action Buttons Bar

A new, dedicated bar for action buttons needs to be created.

- Create a new layout (e.g., a horizontal `LinearLayout` or a `ConstraintLayout`) to hold the four action buttons: "Message", "Unmute", "Call", "Video".
- Each button should consist of an icon and a text label below it.
- This bar should be placed directly below the header section. It could be added as a static view that's part of the header or as a separate item type in the `RecyclerListView`. Given that it seems to scroll with the content, making it a `RecyclerView` item is likely the best approach.
- The existing `callItem` and `videoCallItem` from the `ActionBar` menu should be removed or hidden, and their logic should be moved to these new buttons. The "Message" button will likely open the `ChatActivity`. The "Unmute" button will toggle notification settings.

## 3. User Information Section

This section contains the user's bio and username.

- **Bio and Username:**
    - These are already implemented as rows (`bioRow`, `usernameRow`). The primary work is to update their styling (layout, fonts, colors) to match the mock-up.
    - The `AboutLinkCell` currently used for the bio will likely need style adjustments.
- **QR Code Icon:**
    - A new `ImageView` for the QR code icon needs to be added next to the username row.
    - The `onClickListener` for this icon should open the `QrActivity`, similar to the existing `qrItem` in the menu. The existing `qrItem` can be removed from the `ActionBar` menu.

## 4. Shared Media Layout

The existing `SharedMediaLayout` handles the tabs for media.

- **Tabs ("Posts", "Media", etc.):**
    - The styling of the tab strip (`scrollSlidingTextTabStrip` within `SharedMediaLayout`) needs to be updated. This includes the text color, font, and the style of the selection indicator to match the mockup.
- **Media Grid:**
    - The content within the tabs (e.g., the photo grid) is managed by `SharedMediaLayout`. No major structural changes seem necessary here, but styling might need minor tweaks.

## 5. General Styling

- **Colors:** Update all colors (backgrounds, text, icons) to match the dark theme in the mockup.
- **Fonts:** Ensure fonts and text sizes match the new design for a cleaner, more modern look.
- **Spacing:** Adjust margins and padding throughout the layout to match the spacing in the design.

## Implementation Steps

1.  **Backup/Branch:** Start by creating a new git branch for these changes.
2.  **Header:** Focus on reworking the `TopView` and the avatar-related components first. Get the large background image and text overlays working.
3.  **Action Buttons:** Implement the new action button bar and hook up the click listeners.
4.  **Info Section:** Restyle the Bio and Username rows and add the QR code icon.
5.  **Shared Media:** Update the styling of the `SharedMediaLayout` tabs.
6.  **Animations:** Refine all collapsing and scrolling animations to be smooth and match the new design's feel.
7.  **Cleanup:** Remove old, unused code, especially the `ActionBar` menu items that have been replaced.

## Text-based Mockup

```
+----------------------------------------------------------------+
|                                                                |
|  <-- [Back Arrow]                     [Three Dots Menu Icon]   |
|      (R.drawable.ic_ab_back)          (R.drawable.ic_ab_other) |
|                                                                |
|                                                                |
|                                                                |
|                                                                |
|                                                                |
|               Ronald Copper [Unicorn Emoji]                    |
|                      online                                    |
|                                                                |
+----------------------------------------------------------------+
|                                                                |
|   [Message Icon]   [Unmute Icon]    [Call Icon]    [Video Icon]|
|   (R.drawable.     (R.drawable.     (R.drawable.   (R.drawable. |
|    profile_newmsg)  msg_mute)         ic_call)      profile_video)|
|      Message         Unmute            Call           Video    |
|                                                                |
+----------------------------------------------------------------+
|                                                                |
|   25 y.o, CS streamer, San Francisco                           |
|   Bio                                                          |
|                                                                |
|   @ronald_copper                     [QR Code Icon]            |
|   Username                           (R.drawable.msg_qr_mini)  |
|                                                                |
+----------------------------------------------------------------+
|                                                                |
|  Posts | Media | Files | Links | Voice | Music                 |
|  ----------------                                              |
|                                                                |
|  +-------+  +-------+  +-------+                               |
|  |       |  |       |  |       |                               |
|  | Image |  | Image |  | Image |                               |
|  |       |  |       |  |       |                               |
|  +-------+  +-------+  +-------+                               |
|                                                                |
+----------------------------------------------------------------+
``` 