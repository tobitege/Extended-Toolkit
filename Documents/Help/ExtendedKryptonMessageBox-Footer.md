# ExtendedKryptonMessageBox - Expandable Footer Feature

## Overview

The `ExtendedKryptonMessageBox` now supports an **expandable footer** feature, similar to Windows TaskDialog. This allows developers to display additional information (such as error details, stack traces, or help text) in a collapsible footer area that users can expand or collapse as needed.

## Table of Contents

<!-- Start Document Outline -->

* [Overview](#overview)
* [Features](#features)
* [API Reference](#api-reference)
  * [Show Method Overloads](#show-method-overloads)
  * [Properties](#properties)
* [Usage Examples](#usage-examples)
  * [Basic Usage](#basic-usage)
  * [Error with Stack Trace](#error-with-stack-trace)
  * [Warning with Additional Context](#warning-with-additional-context)
  * [Information with Technical Details](#information-with-technical-details)
  * [Question with Help Text](#question-with-help-text)
  * [Validation Errors](#validation-errors)
* [Implementation Details](#implementation-details)
  * [Footer Panel Structure](#footer-panel-structure)
  * [Toggle Mechanism](#toggle-mechanism)
  * [Sizing Behavior](#sizing-behavior)
* [Best Practices](#best-practices)
* [Compatibility](#compatibility)
* [See Also](#see-also)

<!-- End Document Outline -->

## Features

### Key Features

- **Expandable/Collapsible Footer** - Footer can be shown or hidden by the user via a toggle link
- **Configurable Initial State** - Footer can start expanded or collapsed based on developer preference
- **Automatic Sizing** - Form automatically adjusts its size when footer is expanded or collapsed
- **Optional Feature** - Footer is completely optional; if not specified, message box behaves as before
- **Seamless Integration** - Works with all existing message box features (icons, buttons, timeout, etc.)
- **User-Friendly** - Toggle link text automatically changes between "Show details" and "Hide details"

### Visual Design

- Footer appears below the button panel
- Border edge separates footer from main content
- Toggle link appears at the top of the footer area
- Footer text uses the same font as the main message (configurable via `MessageBoxTypeface`)
- Footer panel uses alternate panel style for visual distinction

## API Reference

### Show Method Overloads

The expandable footer feature is available through new `Show` method overloads that include `footerText` and `footerExpanded` parameters.

#### Basic Footer Overload

```csharp
public static DialogResult Show(
    string text, 
    string caption,
    MessageBoxButtons buttons, 
    KryptonMessageBoxIcon icon, 
    string? footerText, 
    bool footerExpanded = false,
    bool? showCtrlCopy = null, 
    bool topMost = true, 
    Font? messageboxTypeface = null
)
```

**Parameters:**
- `text` - The main message text to display
- `caption` - The title bar caption
- `buttons` - Button configuration (OK, YesNo, YesNoCancel, etc.)
- `icon` - Icon to display (Error, Warning, Information, Question, None)
- `footerText` - **NEW**: Text content to display in the footer. If `null` or empty, footer will not be shown
- `footerExpanded` - **NEW**: If `true`, footer starts expanded; if `false`, starts collapsed (default: `false`)
- `showCtrlCopy` - Show "Ctrl+C to copy" hint in title bar
- `topMost` - Display message box on top of other windows
- `messageboxTypeface` - Custom font for message and footer text

**Returns:** `DialogResult` indicating which button was clicked

#### Owner-Specified Overload

```csharp
public static DialogResult Show(
    IWin32Window owner,
    string text, 
    string caption,
    MessageBoxButtons buttons, 
    KryptonMessageBoxIcon icon, 
    string? footerText, 
    bool footerExpanded = false,
    bool? showCtrlCopy = null, 
    bool topMost = true, 
    Font? messageboxTypeface = null
)
```

**Additional Parameters:**
- `owner` - Owner window for the modal dialog

### Properties

The following properties are available on `ExtendedKryptonMessageBox` instances (though typically accessed through static methods):

#### FooterText

```csharp
public string FooterText { get; set; }
```

Gets or sets the footer text content to display in the expandable footer area.

- **Type:** `string`
- **Default:** `null` (footer not shown)
- **Remarks:** If `null` or empty, the footer will not be displayed

#### FooterExpanded

```csharp
public bool FooterExpanded { get; set; }
```

Gets or sets a value indicating whether the footer is initially expanded.

- **Type:** `bool`
- **Default:** `false`
- **Remarks:** 
  - `true` - Footer starts expanded and visible
  - `false` - Footer starts collapsed (only toggle link visible)

#### IsFooterExpanded

```csharp
public bool IsFooterExpanded { get; }
```

Gets a value indicating whether the footer is currently visible and expanded.

- **Type:** `bool` (read-only)
- **Remarks:** Returns `true` if footer is visible and expanded, `false` otherwise

## Usage Examples

### Basic Usage

#### Simple Error with Collapsed Footer

```csharp
using Krypton.Toolkit.Suite.Extended.Settings;
using System.Windows.Forms;

// Error message with collapsed footer containing stack trace
string mainMessage = "An error occurred while processing your request.";
string footerText = @"Stack Trace:
   at Examples.MyClass.ProcessData()
   at Examples.MyClass.Main(String[] args)
   
Exception: System.InvalidOperationException
Message: The operation cannot be completed.";

DialogResult result = ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Error",
    MessageBoxButtons.OK,
    KryptonMessageBoxIcon.Error,
    footerText,
    footerExpanded: false  // Footer starts collapsed
);
```

#### Warning with Expanded Footer

```csharp
// Warning with important information that should be visible immediately
string mainMessage = "This operation will modify system settings.";
string footerText = @"Additional Information:
• This change affects all users on this system
• A system restart may be required
• Previous settings will be backed up automatically";

DialogResult result = ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Warning",
    MessageBoxButtons.YesNo,
    KryptonMessageBoxIcon.Warning,
    footerText,
    footerExpanded: true  // Footer starts expanded
);
```

### Error with Stack Trace

Display comprehensive error information in a collapsed footer to avoid overwhelming users:

```csharp
try
{
    // Your code here
}
catch (Exception ex)
{
    string mainMessage = "An unexpected error occurred.";
    string footerText = $@"Exception Details:
Type: {ex.GetType().FullName}
Message: {ex.Message}
Source: {ex.Source}

Stack Trace:
{ex.StackTrace}

{(ex.InnerException != null ? $"\nInner Exception:\n{ex.InnerException}" : "")}";

    ExtendedKryptonMessageBox.Show(
        this,
        mainMessage,
        "Error",
        MessageBoxButtons.OK,
        KryptonMessageBoxIcon.Error,
        footerText,
        footerExpanded: false
    );
}
```

### Warning with Additional Context

Provide additional context for warnings without cluttering the main message:

```csharp
string mainMessage = "This operation will modify system settings.";
string footerText = @"Additional Information:
• This change affects all users on this system
• A system restart may be required
• Previous settings will be backed up automatically
• You can restore previous settings from the Settings menu

For more information, visit: https://example.com/help";

DialogResult result = ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Warning",
    MessageBoxButtons.YesNo,
    KryptonMessageBoxIcon.Warning,
    footerText,
    footerExpanded: true
);
```

### Information with Technical Details

Show technical details that may be useful for advanced users or troubleshooting:

```csharp
string mainMessage = "Your file has been successfully processed.";
string footerText = $@"Processing Details:
File: {filePath}
Size: {fileSize:N0} bytes
Processing Time: {processingTime.TotalSeconds:F2} seconds
Pages Processed: {pageCount}
Format: {fileFormat}
Checksum: {checksum}
Timestamp: {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC";

ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Information",
    MessageBoxButtons.OK,
    KryptonMessageBoxIcon.Information,
    footerText,
    footerExpanded: false
);
```

### Question with Help Text

Provide guidance for user decisions:

```csharp
string mainMessage = "Do you want to save your changes before closing?";
string footerText = @"Help:
• Click 'Yes' to save your changes and close the document
• Click 'No' to close without saving (changes will be lost)
• Click 'Cancel' to return to the document and continue editing

Keyboard Shortcuts:
• Ctrl+S: Save
• Ctrl+W: Close
• Esc: Cancel";

DialogResult result = ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Save Changes?",
    MessageBoxButtons.YesNoCancel,
    KryptonMessageBoxIcon.Question,
    footerText,
    footerExpanded: false
);
```

### Validation Errors

Display multiple validation errors in an organized format:

```csharp
var validationErrors = new List<string>
{
    "Email Address: Invalid format. Expected format: user@example.com",
    "Phone Number: Required field cannot be empty",
    "Date of Birth: Date cannot be in the future",
    "Password: Must be at least 8 characters and contain uppercase, lowercase, and numbers",
    "Confirm Password: Passwords do not match"
};

string mainMessage = "Please correct the following validation errors:";
string footerText = "Validation Errors:\n" + 
    string.Join("\n", validationErrors.Select((err, idx) => $"{idx + 1}. {err}")) +
    "\n\nPlease review each field and correct the errors before proceeding.";

ExtendedKryptonMessageBox.Show(
    this,
    mainMessage,
    "Validation Failed",
    MessageBoxButtons.OK,
    KryptonMessageBoxIcon.Warning,
    footerText,
    footerExpanded: true  // Expanded to show all errors immediately
);
```

## Implementation Details

### Footer Panel Structure

The footer consists of the following components:

1. **Footer Panel** (`KryptonPanel`)
   - Uses `PaletteBackStyle.PanelAlternate` for visual distinction
   - Docked to top of form (below button panel)
   - Height adjusts dynamically based on expanded state

2. **Border Edge** (`KryptonBorderEdge`)
   - Separates footer from main content
   - Uses `PaletteBorderStyle.HeaderPrimary`
   - Docked to top of footer panel

3. **Toggle Link** (`LinkLabel`)
   - "Show details" when collapsed
   - "Hide details" when expanded
   - Clickable link to toggle footer state
   - Styled with blue link color

4. **Footer Text** (`KryptonWrapLabel`)
   - Displays the footer content
   - Uses same font as main message (configurable)
   - Automatically wraps text
   - Visible only when footer is expanded

### Toggle Mechanism

When the user clicks the toggle link:

1. Footer expanded state is toggled
2. Footer text visibility is updated
3. Toggle link text changes ("Show details" ↔ "Hide details")
4. Footer panel height is recalculated
5. Form size is recalculated to accommodate new footer height

### Sizing Behavior

The form automatically adjusts its size when the footer is expanded or collapsed:

- **Collapsed State**: Footer height is minimal (~25 pixels) - just enough for the toggle link
- **Expanded State**: Footer height is calculated based on:
  - Text content length
  - Available width (matches message box width)
  - Minimum height of 50 pixels
  - Padding for toggle link and borders

The `UpdateSizing()` method accounts for:
- Message panel height
- Button panel height
- Footer panel height (varies based on state)
- Maximum width across all panels

## Best Practices

### When to Use the Footer

✅ **Good Use Cases:**
- **Error Details**: Stack traces, exception information, technical error codes
- **Additional Context**: Extra information that doesn't fit in the main message
- **Help Text**: Guidance for user decisions, keyboard shortcuts
- **Validation Errors**: Multiple field validation errors
- **System Information**: Diagnostic information, file details, processing statistics
- **Troubleshooting**: Steps to resolve issues, contact information

❌ **Avoid Using Footer For:**
- Critical information that users must see immediately (use main message instead)
- Very short text that fits easily in the main message
- Information that changes the meaning of the main message

### Footer Text Guidelines

1. **Keep it Relevant**: Footer should provide additional context, not replace the main message
2. **Structure Clearly**: Use bullet points, numbered lists, or clear sections for readability
3. **Appropriate Length**: Very long footer text may require scrolling; consider breaking into sections
4. **Technical vs. User-Friendly**: Consider your audience - technical details are fine for developer tools, but keep user-facing messages simple

### Initial State Recommendations

- **Start Collapsed** (`footerExpanded: false`):
  - Stack traces and error details
  - Technical information
  - Optional help text
  - Information that most users won't need

- **Start Expanded** (`footerExpanded: true`):
  - Important warnings that users should read
  - Validation errors that need immediate attention
  - Critical additional context
  - Information that affects the user's decision

### Integration with Other Features

The footer feature works seamlessly with all existing `ExtendedKryptonMessageBox` features:

- ✅ **Icons**: All icon types (Error, Warning, Information, Question, None)
- ✅ **Button Configurations**: All button types (OK, YesNo, YesNoCancel, etc.)
- ✅ **Custom Fonts**: Footer uses the same `MessageBoxTypeface` as the main message
- ✅ **Timeout**: Footer works with timeout functionality
- ✅ **Do Not Show Again**: Footer works with the "Do not show again" checkbox
- ✅ **Custom Button Text**: Footer works with custom button text
- ✅ **TopMost**: Footer respects the `topMost` setting

## Compatibility

### Framework Support

The expandable footer feature is available in all supported framework versions:
- .NET Framework 4.7.2 and later
- .NET 8.0 and later
- .NET 9.0 and later
- .NET 10.0 and later
- .NET 11.0 and later

### Backward Compatibility

- **Fully Backward Compatible**: Existing code continues to work without modification
- **Optional Feature**: If `footerText` is `null` or empty, the footer is not shown and the message box behaves exactly as before
- **No Breaking Changes**: All existing `Show` method overloads remain unchanged

### Namespace

```csharp
using Krypton.Toolkit.Suite.Extended.Settings;
```

The `ExtendedKryptonMessageBox` class is located in the `Krypton.Toolkit.Suite.Extended.Settings` namespace.

## See Also

- [ExtendedKryptonMessageBox API Documentation](../../Source/Krypton%20Toolkit/Krypton.Toolkit.Suite.Extended.Settings/Classes/Other/ExtendedKryptonMessageBox.cs)
- [MessageBox Footer Example](../../Source/Krypton%20Toolkit/Examples/MessageBoxFooterExample.cs)
- [Issue #511 - Feature Request](https://github.com/Krypton-Suite/Extended-Toolkit/issues/511)
- [Changelog](Changelog.md)

---

**Last Updated:** January 2026  
**Feature Version:** 1.0  
**Namespace:** `Krypton.Toolkit.Suite.Extended.Settings`
