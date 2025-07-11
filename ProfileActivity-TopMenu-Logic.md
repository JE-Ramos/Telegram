# ProfileActivity Top Menu Button Logic

## Overview
The ProfileActivity implements a dynamic top menu system with context-sensitive buttons that change based on profile type, user permissions, and relationship status.

## Menu Button Types

### Primary Action Bar Buttons
Located at `ProfileActivity.java:3609-3673`

1. **QR Code Button** (`qr_button = 37`)
   - **Icon**: `R.drawable.msg_qr_mini`
   - **Visibility**: Own profile only
   - **Function**: Generates QR code for profile sharing

2. **Search Button** (`search_button = 32`)
   - **Icon**: `R.drawable.ic_ab_search`
   - **Visibility**: Avatar gallery mode
   - **Function**: Search through avatar photos

3. **Video Call Button** (`video_call_item = 16`)
   - **Icon**: `R.drawable.profile_video`
   - **Visibility**: User profiles with call capability
   - **Function**: Initiates video call

4. **Voice Call Button** (`call_item = 15`)
   - **Icon**: `R.drawable.msg_voicechat2` (chats) / `R.drawable.ic_call` (users)
   - **Visibility**: User profiles and group chats with call capability
   - **Function**: Initiates voice call

5. **Edit Button** (`edit_profile = 41` / `edit_channel = 12`)
   - **Icon**: `R.drawable.group_edit_profile`
   - **Visibility**: Own profile or admin permissions
   - **Function**: Edit profile/channel information

6. **More Button** (`10`)
   - **Icon**: `R.drawable.ic_ab_other`
   - **Visibility**: Always visible
   - **Function**: Opens dropdown menu with additional options

## More Menu Items (Secondary Actions)

### User Profile Actions
- `add_contact = 1` - Add user to contacts
- `block_contact = 2` - Block/unblock user
- `share_contact = 3` - Share user profile
- `edit_contact = 4` - Edit contact information
- `delete_contact = 5` - Delete from contacts
- `start_secret_chat = 20` - Start encrypted conversation
- `gift_premium = 38` - Send Telegram Premium gift
- `report = 24` - Report user for violations

### Bot Profile Actions
- `share = 10` - Share bot profile
- `report = 24` - Report bot
- `block_contact = 2` - Block bot

### Channel/Group Actions
- `leave_group = 7` - Leave channel/group
- `invite_to_group = 9` - Invite users to group
- `share = 10` - Share channel/group link
- `search_members = 17` - Search group members
- `statistics = 19` - View channel statistics (admin only)
- `view_discussion = 22` - View channel discussion group
- `channel_stories = 39` - View channel stories archive

### System Actions
- `add_shortcut = 14` - Add chat shortcut to home screen

## Dynamic Menu Population Logic

### Method: `createActionBarMenu(boolean animated)`
**Location**: `ProfileActivity.java:10756-10955`

The menu is dynamically populated based on:

1. **Profile Type Detection**:
   - Own profile vs. other users
   - Regular users vs. bots
   - Private chats vs. channels vs. groups

2. **Relationship Status**:
   - Contact vs. non-contact
   - Blocked vs. unblocked
   - Mutual contact vs. one-way

3. **Permissions**:
   - Admin rights in channels/groups
   - Call permissions
   - Edit permissions

4. **Feature Availability**:
   - Premium features
   - Regional restrictions
   - Account limitations

## Menu Item Click Handling

### Method: `ActionBarMenuOnItemClick.onItemClick(int id)`
**Location**: `ProfileActivity.java:2310+`

Each menu item ID is handled with specific logic:

```java
actionBar.setActionBarMenuOnItemClick(new ActionBar.ActionBarMenuOnItemClick() {
    @Override
    public void onItemClick(final int id) {
        switch (id) {
            case block_contact:
                // Block/unblock user logic
                break;
            case add_contact:
                // Add to contacts logic
                break;
            case call_item:
            case video_call_item:
                // Call initiation logic
                break;
            // ... more cases
        }
    }
});
```

## Technical Implementation Details

### Button Creation Pattern
```java
ActionBarMenu menu = actionBar.createMenu();
ActionBarMenuItem item = menu.addItem(ID, DRAWABLE_RESOURCE, resourcesProvider);
item.setContentDescription(ACCESSIBILITY_TEXT);
```

### Visibility Control
- Buttons are shown/hidden using `setVisibility()`
- Menu items are added/removed dynamically
- Animated transitions supported via `animated` parameter

### State Management
- Menu recreated when profile data changes
- Button states updated based on user actions
- Real-time permission checking

### Accessibility
- Content descriptions set for all buttons
- Proper focus handling for screen readers
- Keyboard navigation support

## Menu Update Triggers

The menu is updated when:
1. Profile data loads or changes
2. User permissions change
3. Relationship status changes (block/unblock, add/remove contact)
4. Feature availability changes
5. Theme changes

## Performance Considerations

- Menu items created on-demand to reduce memory usage
- Expensive operations (statistics, member counts) loaded asynchronously
- Button states cached to avoid repeated calculations
- Smooth animations for menu transitions