# Robot - LVGL Pro UI Project

240×240 embedded display UI for an AI-powered robot/speaker device.

## Project Structure

```
robot/
  project.xml                      # LVGL Pro project config (240×240)
  globals.xml                      # Shared constants, styles, subjects, fonts
  screens/
    screen_ai_main.xml             # AI main page (PERMANENT — dual mode)
    screen_settings.xml            # Settings page (scrollable, QR code)
    screen_music.xml               # Local music player page
  components/
    chat_bubble/chat_bubble.xml    # Chat message bubble component
    music_item/music_item.xml      # Song list item component
```

## Screens

### 1. AI Main Page (`screen_ai_main.xml`)
- **Permanent screen** — state persists across page switches
- **Expression Mode** (`ai_mode = 0`): Animated robot face with 7 expressions
  - neutral (0), happy (1), sad (2), thinking (3), speaking (4), surprised (5), listening (6)
  - Each expression shows/hides different mouth shapes via `bind_flag_if_eq`
- **Dialog Mode** (`ai_mode = 1`): WeChat-style chat history
  - Scrollable chat container for up to 25 messages
  - AI messages: left-aligned, gray background
  - User messages: right-aligned, blue background
- Top bar: Music button (♪) + Settings button (⚙)
- Mode switching happens in Settings → resets the main page

### 2. Settings Page (`screen_settings.xml`)
Long scrollable container with 5 sections:
1. **Volume** — slider (0-100%) with 🔊 icon
2. **AI Mode** — toggle button (😊 Expression ↔ 💬 Dialog)
3. **Brightness** — slider (0-100%) with ☀ icon
4. **WiFi** — toggle button (📶 ON/OFF)
5. **QR Code** — WiFi provisioning QR (uses `lv_qrcode` widget)

### 3. Local Music Page (`screen_music.xml`)
- **Sort Order** button: Sequential (📋) ↔ Random (🔀)
- **Play Mode** button: Single Repeat (🔂) ↔ List Play (🔁)
- **Song list**: Scrollable, with placeholder items for preview
- **Highlight selection**: currently playing item shown with blue border + ▶ indicator
- Scroll-stop-3s auto-play logic handled by C code

## Key Subjects (Data Bindings)

| Subject | Type | Default | Description |
|---------|------|---------|-------------|
| `ai_mode` | int | 0 | 0=expression, 1=dialog |
| `expression` | int | 0 | 0-6 face expression |
| `volume` | int | 50 | 0-100 |
| `brightness` | int | 80 | 0-100 |
| `wifi_on` | int | 0 | 0=off, 1=on |
| `sort_order` | int | 0 | 0=sequential, 1=random |
| `play_mode` | int | 0 | 0=single_repeat, 1=list_play |
| `current_song_index` | int | 0 | Selected song index |
| `current_song_name` | string | "No song" | Now playing display |

## C Code Integration Notes

### Dialog Mode Chat Messages
Dynamically create chat bubbles as children of `chat_container`:
```c
// AI message (left-aligned)
lv_obj_t *row = lv_obj_create(chat_container);
lv_obj_set_flex_flow(row, LV_FLEX_FLOW_ROW);
lv_obj_set_flex_align(row, LV_FLEX_ALIGN_START, LV_FLEX_ALIGN_CENTER, LV_FLEX_ALIGN_CENTER);
lv_obj_t *bubble = lv_label_create(row);
lv_label_set_text(bubble, message_text);
lv_obj_add_style(bubble, &style_bubble_ai, 0);
```

### Music Song List
Dynamically create song items as children of `song_list_container`:
```c
lv_obj_t *item = lv_obj_create(song_list_container);
lv_obj_set_size(item, LV_PCT(100), LV_SIZE_CONTENT);
lv_obj_t *label = lv_label_create(item);
lv_label_set_text(label, filename);
// Apply selected style if index == current_song_index
```

### Mode Switching
When `ai_mode` changes, C code should reset the main page:
```c
lv_subject_set_int(ai_mode_subject, new_mode); // 0 or 1
// Force screen_ai_main to refresh its visible mode
```

### WiFi QR Code
Update at runtime with provisioning data:
```c
lv_qrcode_set_data(wifi_qrcode, wifi_provisioning_json, strlen(wifi_provisioning_json));
```

### Expression Changes
Set expression based on conversation state:
```c
lv_subject_set_int(expression_subject, EXPRESSION_HAPPY); // 0-6
```

## Fonts

Font files should be placed in `robot/fonts/Geist/`:
- `Geist-Regular.ttf` → font_10, font_12, font_14
- `Geist-SemiBold.ttf` → font_16, font_20, font_24
<!-- - `Geist-Bold.ttf` → font_36, font_48 -->

Copy from tutorials fonts directory or download from Google Fonts.
