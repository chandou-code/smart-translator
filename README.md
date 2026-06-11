# Smart Translator - English Learning Assistant

A lightweight UniApp-based English learning assistant with keyword annotations, smart sentence selection, and text-to-speech capabilities. **Pure frontend implementation - no backend required.**

## Features

### 1. Keyword Annotations
- Paste English text with translations in brackets
- Example: `expanded(拓展)` or `expanded(拓展)`
- Keywords displayed with purple dashed underline
- Tap keywords to show/hide translations
- **100% frontend - no backend needed**

### 2. Smart Sentence Selection
- Long-press any sentence to select the entire sentence
- Automatically expands to complete sentences
- Supports Chinese and English punctuation
- **100% frontend - no backend needed**

### 3. Custom Long-Press Menu
- Custom popup menu on long-press
- Read aloud and copy functionality
- Menu positioned above touch point

### 4. Text-to-Speech
- Browser TTS (H5) / Native TTS (App)
- Read selected sentences aloud

## Quick Start

### Development

1. Open `翻译前端uniapp` folder in HBuilderX

2. Run:
   - H5: Run → Run to Browser → Chrome
   - Android: Run → Run to Device/Emulator

3. **Works standalone - no backend needed**

### APK Installation

Latest APK: `翻译前端uniapp/unpackage/release/apk/`

## Tech Stack

- **Framework**: UniApp (Vue 3)
- **UI**: Modern card design with purple gradient theme
- **TTS**: Web Speech API / Native TTS
- **Platforms**: H5, Android App, WeChat Mini Program

## Usage

1. Paste English text with `word(translation)` format
2. Tap "解析文本" (Parse) button
3. Keywords show purple dashed underlines
4. Tap keywords to show/hide translations
5. Long-press any sentence to highlight and show menu
6. Tap "朗读" to read, "复制" to copy

## License

MIT License