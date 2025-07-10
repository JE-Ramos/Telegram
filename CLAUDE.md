# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the official Telegram Android app source code. It's a complex messaging application with native C++ components for performance-critical operations and extensive custom UI framework.

## Common Development Commands

### Building the Project
```bash
# Build debug version
./gradlew assembleDebug

# Build release version  
./gradlew assembleRelease

# Build specific variant (HA_private, HA_public, standalone)
./gradlew assembleHA_private
./gradlew assembleHA_public
./gradlew assembleStandalone

# Clean build
./gradlew clean
```

### Running Tests
```bash
# Run all tests
./gradlew test

# Run specific test variant
./gradlew testDebug
```

### Code Analysis
```bash
# Run lint checks
./gradlew lint

# Generate lint report
./gradlew lintDebug
```

### Native Code
```bash
# Build native components (requires NDK)
./gradlew externalNativeBuild
```

## High-Level Architecture

### Core Components

**Multi-Account Architecture**: The app supports multiple simultaneous user accounts through `AccountInstance` which provides account-specific access to all major controllers.

**Main Application Flow**:
- `ApplicationLoader` - Main application initialization
- `LaunchActivity` - Primary entry point and fragment host
- `AccountInstance` - Account-specific service locator
- `BaseFragment` - Foundation for all UI screens

**Key Packages**:
- `org.telegram.messenger` - Core business logic (MessagesController, ContactsController, etc.)
- `org.telegram.tgnet` - Network layer and MTProto protocol implementation
- `org.telegram.ui` - UI components and fragments
- `org.telegram.SQLite` - Database abstraction layer

### Architectural Patterns

**Observer Pattern**: `NotificationCenter` provides event-driven communication between components.

**Singleton Controllers**: Most business logic controllers follow singleton pattern with account-specific instances.

**Fragment-Based UI**: Custom `ActionBarLayout` manages fragment transitions with `BaseFragment` as the foundation.

**Hybrid Native/Java**: Performance-critical operations (networking, media processing, cryptography) implemented in C++ with JNI bridge.

### Key Controllers

- `MessagesController` - Core messaging logic
- `MessagesStorage` - Database operations
- `ConnectionsManager` - Network connections
- `MediaController` - Media handling
- `NotificationsController` - Push notifications
- `ContactsController` - Contact management

### Data Flow

1. User Interaction → UI Components → Business Logic Controllers
2. Controllers → Network Layer (tgnet) → MTProto Protocol
3. Data Persistence → MessagesStorage → SQLite Database
4. Events → NotificationCenter → UI Updates

## Native Code Integration

The app includes extensive native C++ code in `TMessagesProj/jni/`:

- **tgnet/**: MTProto protocol implementation
- **ffmpeg/**: Video/audio processing
- **opus/**: Audio codec
- **mozjpeg/**: JPEG processing
- **boringssl/**: Cryptographic operations
- **rlottie/**: Lottie animation rendering

## Build Variants

- `debug` - Development build with debugging enabled
- `release` - Production build
- `HA_private` - Private beta build
- `HA_public` - Public beta build
- `standalone` - Standalone version without Google Services

## Development Setup Requirements

- Android Studio 3.4+
- Android NDK rev. 20+
- Android SDK 8.1+
- CMake 3.10.2+
- Custom release.keystore and google-services.json required for building

## Code Organization

**Main Module**: `TMessagesProj/` contains the core library code
**App Modules**: `TMessagesProj_App/`, `TMessagesProj_AppStandalone/`, etc. are different app variants

**Source Structure**:
- `src/main/java/org/telegram/` - Main application code
- `jni/` - Native C++ code
- `config/` - Build configuration files

## Important Configuration

- `BuildVars.java` - Contains API keys and configuration (requires setup)
- `local.properties` - Local build configuration
- Multiple build variants with different configurations
- Proguard rules for release builds

## Development Notes

- The app uses a custom UI framework rather than standard Android components
- Extensive theming system with dynamic theme support
- Multi-account support is built into the architecture from the ground up
- Performance optimizations include custom memory management and efficient data structures
- Security features include end-to-end encryption and secure local storage

# Memories

## ProfileActivity Avatar Expansion Animation

The ProfileActivity uses a sophisticated layered layout system for the avatar expansion animation:

### Layout Hierarchy
1. **ListView** - Added first with 88dp top padding (`PROFILE_HEADER_COLLAPSED_HEIGHT_DP`)
2. **TopView** - Background color layer
3. **Avatar Container** - Overlay positioned above ListView's padded area

### Expansion Mechanism
The ListView doesn't move - instead the system uses coordinate-based animation:

**Key Variable: `extraHeight`**
- Tracks how much the first ListView item (position 0) has moved up when pulled down
- When `extraHeight > 0`, triggers avatar expansion animation

**Scroll Detection (ProfileActivity.java:7123-7167)**
- `checkListViewScroll()` monitors the first ListView item's position
- When first item is pulled down (`top > 0`), `extraHeight = top`
- Triggers `needLayout(true)` to recalculate avatar positioning

**Avatar Animation Response (ProfileActivity.java:7509-7551)**
- `avatarAnimationProgress` controls the expansion transition
- Avatar scales from 42x42dp to fill expanded space
- Corner radius changes from round to square
- Position animates from (64,0) to center of expanded area
- Container resizes: `(extraHeight + newTop) / avatarScale`

**Visual Effect**: Avatar appears to expand with pull gesture, but it's actually an overlay growing to fill the space revealed by the pulled-down ListView content.

## ProfileActivity Write Button States

The write button (60x60dp circular FAB) adapts its appearance and behavior based on profile context:

### Button States by Profile Type

**Own Profile (Self) - Edit Mode:**
- **Icon**: `R.raw.camera_outline` (Lottie animation)
- **Action**: Opens camera/gallery picker for profile picture
- **Description**: "Change profile picture"

**Other User Profile:**
- **Icon**: `R.drawable.profile_newmsg` (message icon)
- **Action**: Opens private chat with user
- **Description**: "Open chat"

**Channel/Group Profile:**
- **Icon**: `R.drawable.profile_discuss` (discussion icon)
- **Action**: Opens linked discussion chat
- **Description**: "View discussion"

### Visibility Logic (ProfileActivity.java:7267-7269)
Shows when: avatar expansion > 20%, not in search mode, and context-specific conditions
Hides when: avatar collapsed, search active, or profile editing with avatar row visible

### Animation Transitions
- **Appear**: Scale 0.2f→1.0f, Alpha 0.0f→1.0f (DecelerateInterpolator, 150ms)
- **Disappear**: Scale 1.0f→0.2f, Alpha 1.0f→0.0f (AccelerateInterpolator, 150ms)

### Background Styling (ProfileActivity.java:13627-13647)
All states use circular background with shadow, adapting colors based on theme and peer color scheme.

## ProfileActivity Profile Info Section Spacing

Added consistent spacing above the profile info section to provide visual buffer below the write button area.

### Implementation (ProfileActivity.java:11662-11668)
**Method**: Header padding approach - adds 76dp top padding to `infoHeaderRow` in `onBindViewHolder()`

### Universal Application
- **User Profiles**: 76dp spacing above "Info" header
- **Channel Profiles**: 76dp spacing above "Description" header  
- **Topic Profiles**: 76dp spacing above topic header
- **Always Applied**: Consistent spacing regardless of write button presence

### Technical Details
- **Location**: `VIEW_TYPE_HEADER` case in ListAdapter.onBindViewHolder()
- **Padding**: `headerCell.setPadding(left, AndroidUtilities.dp(76), right, bottom)`
- **Scope**: Only applied to `position == infoHeaderRow`
- **Other Headers**: Explicitly reset to 0 top padding to prevent layout issues
- **Spacing Calculation**: 76dp = 60dp (write button height) + 16dp (visual padding)

### Rationale
Provides visual separation between the floating write button area and profile content, improving UI hierarchy and readability across all profile types (users, channels, topics).

## ProfileActivity Profile Button Container System

Implemented a comprehensive floating action button system that replaces the single write button with a full-width container holding multiple action buttons.

### Container Implementation (ProfileActivity.java:5330-5355)
**Method**: `profileButtonContainer` - Full-width FrameLayout overlay that follows same scroll behavior as write button
**Position**: 76dp height (60dp button + 16dp padding) positioned above profile info section
**Animation**: Squeeze animation using `scaleY` and `translationY` when approaching action bar
**Background**: Dynamic background matching avatar colors with proper transparency

### Button System (ProfileActivity.java:1859-1910)
**Class**: `ProfileActionButton` - Custom button view with icon and text
**Layout**: LinearLayout with ImageView (24dp icon) and TextView (10dp text)
**Styling**: 
- Light mode: `0x33000000` (20% black background)
- Dark mode: `0x1AFFFFFF` (10% white background for lighter gray appearance)
- Corner radius: 12dp
- Height: 45dp for compact appearance

### Dynamic Button Population (ProfileActivity.java:13872-13952)
**Method**: `updateProfileActionButtons()` - Dynamically adds 2-4 buttons based on profile type
**Button Types**:
- **User Profiles**: Message, Call, Video, Share
- **Bot Profiles**: Message, Share, Mute, Block
- **Channel Profiles**: Message, Share, Mute, Leave
- **Business Profiles**: Message, Call, Video, Share

### Button Actions (ProfileActivity.java:13954-14008)
**Method**: `handleProfileActionClick()` - Handles button click events
**Actions**: Opens appropriate dialogs/activities for messaging, calling, sharing, etc.

### Write Button Integration
**Original Write Button**: Hidden with `setVisibility(View.GONE)` to prevent conflicts
**Replacement**: ProfileButtonContainer provides same functionality with expanded options

### Dark Mode Color Optimization
**Issue**: Initial `0x80FFFFFF` (50% white) was too dark in dark mode
**Solution**: Changed to `0x1AFFFFFF` (10% white) for lighter, more subtle gray appearance
**Result**: Better visual hierarchy and readability in dark theme

## ProfileActivity Avatar Expansion Animation Details

### Avatar Movement During Expansion

**Initial State (Collapsed)**:
- Size: 42x42dp circular avatar
- Position: 64dp from left, centered in action bar
- Left margin: 64dp

**Expanded State**:
- Size: Fills expanded header area
- Position: Centered horizontally (0 left margin)
- Shape: Square (corner radius animates to 0)

### Animation Path

**Horizontal Movement (ProfileActivity.java:7515)**:
```java
avatarX = -AndroidUtilities.dpf2(47f) * diff;
```
- Moves 47dp to the left as `diff` progresses from 0 to 1

**Vertical Movement (ProfileActivity.java:7516)**:
```java
avatarY = (actionBar.getOccupyStatusBar() ? AndroidUtilities.statusBarHeight : 0) + 
          ActionBar.getCurrentActionBarHeight() / 2.0f * (1.0f + diff) - 
          21 * AndroidUtilities.density + 
          27 * AndroidUtilities.density * diff + 
          actionBar.getTranslationY();
```
- Complex formula creates curved trajectory
- Combines action bar height scaling with linear movement

### Scaling Phases

**Phase 1 (0-88dp pull)**:
```java
avatarScale = (42 + 18 * diff) / 42.0f;
```
- Scales from 42dp to 60dp (1.43x)

**Phase 2 (beyond 88dp)**:
```java
avatarScale = AndroidUtilities.lerp((42f + 18f) / 42f, (42f + 42f + 18f) / 42f, Math.min(1f, expandProgress * 3f));
```
- Scales from 60dp to 102dp (2.43x)

### Container Positioning (ProfileActivity.java:7742-7746)
```java
avatarContainer.setScaleX(avatarScale);
avatarContainer.setScaleY(avatarScale);
avatarContainer.setTranslationX(avatarX);
avatarContainer.setTranslationY((float) Math.ceil(avatarY));
```

### Why the Curved Movement
The curved trajectory results from the non-linear Y position formula that combines:
- Action bar height multiplication by `(1.0f + diff)`
- Linear movement component `27 * density * diff`
- This creates a parabolic path where the avatar moves down-left in a smooth curve

## ProfileActivity Avatar Circular Shape and Scaling

### How Avatar Becomes Circular

The avatar's circular shape is controlled by the `setRoundRadius()` method:

**Round Radius Setting (ProfileActivity.java:5015)**:
```java
avatarImage.setRoundRadius(getSmallAvatarRoundRadius());
```

**Default Radius Calculation (ProfileActivity.java:5878-5886)**:
```java
private int getSmallAvatarRoundRadius() {
    if (chatId != 0) {
        TLRPC.Chat chatLocal = getMessagesController().getChat(chatId);
        if (ChatObject.isForum(chatLocal)) {
            return AndroidUtilities.dp(needInsetForStories() ? 11 : 16);
        }
    }
    return AndroidUtilities.dp(12);  // Half of 24dp avatar size for perfect circle
}
```

### Avatar Size and Scaling

**Container Setup**: 24dp × 24dp initial size
**Image View**: Uses `MATCH_PARENT` to fill container
**Minimum Scale**: ~1.43x applied even when collapsed (`avatarScale = (42 + 18) / 42`)

### Why 48dp Radius Was Needed

When the user increased avatar size but it became "squarish circle", this was because:
1. The avatar container has a minimum scale of 1.43x applied
2. Container: 24dp × 1.43 = ~34dp minimum display size
3. The avatar can scale up to 2.43x during expansion
4. The actual rendered size with scaling is much larger than the base 24dp
5. A 48dp radius creates a perfect circle, suggesting ~96dp actual rendered size

**Key Insight**: The avatar's displayed size is affected by `avatarScale` transformations, making it appear larger than its defined container size.

## ProfileActivity Centered Header Layout System

Implemented a centered layout system for the ProfileActivity header to improve visual balance and modern design aesthetics.

### Avatar Container Positioning (ProfileActivity.java:4992)
**Original**: Left-aligned at 64dp with 42x42dp size
**Updated**: Centered with `Gravity.CENTER_HORIZONTAL` and 24x24dp size
```java
int avatarSize = AndroidUtilities.dp(24);
int centerX = avatarSize + AndroidUtilities.dp(12);  // 36dp total offset
avatarContainer2.addView(avatarContainer, LayoutHelper.createFrame(avatarSize, avatarSize, 
    Gravity.CENTER_HORIZONTAL, 
    centerX / AndroidUtilities.density,  // 36dp left margin
    -centerX / AndroidUtilities.density, // -36dp top margin
    0, 0));
```

### Name and Status Text Centering (ProfileActivity.java:5207-5210)
**Problem**: nameTextView used 118dp left margin designed for left-aligned layout
**Solution**: Updated to match avatar's 36dp base offset for consistent centering
```java
int baseOffset = AndroidUtilities.dp(36); // Same as avatar's centerX calculation
int nameLeftMargin = baseOffset - (nameTextView[a].getSideDrawablesSize() / 2);
```

### Animation Coordinate Updates (ProfileActivity.java:7879-7882)
**Issue**: Animation translations used -21dp offset for old left-aligned system
**Fix**: Updated to -47dp offset to account for positioning difference (118dp - 36dp = 82dp, adjusted for centering)
```java
nameX = AndroidUtilities.dp(-47f) + avatarContainer.getMeasuredWidth() * (avatarScale - (42f + 18f) / 42f);
onlineX = AndroidUtilities.dp(-47f) + avatarContainer.getMeasuredWidth() * (avatarScale - (42f + 18f) / 42f);
```

### Key Benefits
- **Visual Consistency**: Avatar and text elements properly centered
- **Modern Layout**: Improved visual hierarchy and balance
- **Maintained Animations**: All expand/collapse animations preserved
- **Drawable Compensation**: Side drawable positioning still properly compensated

### Technical Notes
- Both `avatarContainer` and `nameTextView` use `Gravity.CENTER_HORIZONTAL` with matching 36dp base offset
- Animation calculations adjusted to maintain smooth transitions during header expansion
- Works well for communities and channels where centered layout improves readability

## ProfileActivity NameTextView Centering Fix

### Problem: Manual Margin Calculation
**Issue**: Originally used `getSideDrawablesSize()` for margin calculation during view creation, but this returned 0 because drawables weren't set yet.

```java
// WRONG - Called during view creation before drawables are set
int nameLeftMargin = baseOffset - (nameTextView[a].getSideDrawablesSize() / 2);
```

### Solution: Proper Centering
**Fix**: Removed manual margin calculation and used proper `Gravity.CENTER_HORIZONTAL` like the avatar container.

```java
// CORRECT - Let framework handle centering
avatarContainer2.addView(nameTextView[a], LayoutHelper.createFrame(
    a == 0 ? initialTitleWidth : LayoutHelper.WRAP_CONTENT, 
    LayoutHelper.WRAP_CONTENT, 
    Gravity.CENTER_HORIZONTAL | Gravity.TOP, 
    0, 44, // leftMargin = 0, let gravity handle it
    (a == 0 ? rightMargin - (hasTitleExpanded ? 10 : 0) : 0), 0
));
```

### Key Insights
- **Timing Issue**: View creation happens before drawable assignment in `updateRowsIds()`
- **Framework Centering**: `Gravity.CENTER_HORIZONTAL` handles centering automatically
- **Drawable Positioning**: SimpleTextView handles drawable positioning internally
- **Consistency**: Both avatar and nameTextView now use the same centering approach

### Debug Features Added
- **Debug colors** in SimpleTextView for drawable areas (red=left, green=right, blue=right2)
- **PROFILEDEBUG comments** for easy searching of drawable-related code
- **Logging** for bot verification and encrypted chat margin adjustments

## Native Audio Components

### CallAudioTone.h (TMessagesProj/jni/voip/tgcalls/platform/darwin/iOS/)

**Purpose**: Handles audio tone playback for VoIP calls on iOS platform

**Key Features**:
- **Sample Storage**: `std::vector<int16_t> _samples` - Audio sample data
- **Sample Rate**: Default 48kHz (`_sampleRate = 48000`)
- **Loop Control**: `_loopCount` for repeating tones
- **Playback Position**: `_offset` for current playback position

**Methods**:
```cpp
// Getters
std::vector<int16_t> const samples() const;
int sampleRate() const;
int loopCount() const;
size_t offset() const;

// Setters
void setLoopCount(int loopCount);
void setOffset(size_t offset);
```

**Usage Context**: Part of Telegram's native VoIP implementation using tgcalls library for iOS call audio management.

## ProfileActivity NameTextView Gravity Fix

### Problem: Left-Aligned Loading Appearance
**Issue**: During initial loading, `nameTextView[0]` appeared left-aligned despite container centering.

**Root Cause**: Conflicting gravity settings:
- **Container**: `Gravity.CENTER_HORIZONTAL` (centers TextView within container)
- **TextView**: `Gravity.LEFT` (aligns text left within TextView)
- **Result**: Centered TextView with left-aligned text = appears left-positioned

### Solution: Align Text Content Gravity
**Fix**: Changed TextView internal gravity to match container centering:
```java
// Before: Text aligned left within centered TextView container
nameTextView[a].setGravity(Gravity.LEFT);

// After: Text aligned center within centered TextView container  
nameTextView[a].setGravity(Gravity.CENTER);
```

### Impact
- **nameTextView[0]**: Animation layer starts from correct centered position
- **nameTextView[1]**: Visible layer displays centered from initial load
- **Loading State**: No more leftmost positioning during data loading
- **Consistency**: Both layers use unified centering approach

### Technical Details
- **Timing**: View creation happens before `updateRowsIds()` data population
- **Gravity Hierarchy**: Container gravity + TextView gravity = final positioning
- **Animation**: `nameTextView[0]` alpha transitions from correct centered start position

## ProfileActivity NameTextView Animation Alignment Fix

### Problem: Misaligned Animation Layers
**Issue**: `nameTextView[0]` (animation layer) and `nameTextView[1]` (visible layer) had different layout parameters causing misalignment during animations.

**Root Cause**: Different layout constraints:
```java
// nameTextView[0]: Fixed width + right margin
avatarContainer2.addView(nameTextView[a], LayoutHelper.createFrame(
    a == 0 ? initialTitleWidth : LayoutHelper.WRAP_CONTENT,  // Different widths
    LayoutHelper.WRAP_CONTENT, 
    Gravity.CENTER_HORIZONTAL | Gravity.TOP, 
    0, 44, 
    (a == 0 ? rightMargin - (hasTitleExpanded ? 10 : 0) : 0), 0  // Different margins
));
```

### Solution: Identical Layout Parameters
**Fix**: Both layers now use identical constraints for synchronized animation paths:
```java
// Both layers: Same width + same margins
avatarContainer2.addView(nameTextView[a], LayoutHelper.createFrame(
    LayoutHelper.WRAP_CONTENT,  // Same width for both
    LayoutHelper.WRAP_CONTENT, 
    Gravity.CENTER_HORIZONTAL | Gravity.TOP, 
    0, 44, 
    0, 0  // Same margins for both
));
```

### Impact
- **Same Animation Path**: Both views follow identical movement trajectories
- **Perfect Overlap**: Animation and visible layers align perfectly when content matches
- **Smooth Transitions**: No jitter or position jumps during alpha crossfades
- **Consistent Centering**: Both layers center using same calculation method

### Key Principle
For smooth multi-layer animations, **all layers must have identical layout constraints** to ensure they move along the same path during transformations.