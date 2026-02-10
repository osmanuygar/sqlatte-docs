# UI Screenshots Directory

This directory contains all screenshots for the SQLatte UI Guide documentation.

## Required Screenshots

Upload screenshots with these exact filenames to match the documentation:

### Widget Screenshots

#### Standard Widget
- [ ] `standard-widget-badge.png` - Badge button in bottom-right corner
- [ ] `standard-widget-chat.png` - Main chat interface showing conversation
- [ ] `widget-query-results.png` - SQL results displayed in table format
- [ ] `widget-sql-code.png` - Syntax-highlighted SQL code block with copy button
- [ ] `widget-chart-modal.png` - Chart visualization modal
- [ ] `widget-history-panel.png` - History slide-out panel
- [ ] `widget-favorites-panel.png` - Favorites panel
- [ ] `widget-fullscreen.png` - Widget in fullscreen mode

#### Auth Widget
- [ ] `auth-widget-login.png` - Login modal with credentials form
- [ ] `auth-widget-chat.png` - Chat interface after authentication
- [ ] `auth-session-indicator.png` - Session status and logout controls

### Dashboard Screenshots

#### Main Dashboard
- [ ] `dashboard-home.png` - Home tab landing page
- [ ] `dashboard-assistant.png` - Assistant tab with embedded widget
- [ ] `dashboard-demo.png` - Demo tab with examples
- [ ] `dashboard-analytics.png` - Analytics dashboard
- [ ] `dashboard-schedules.png` - Schedule management interface
- [ ] `dashboard-config.png` - Configuration tab
- [ ] `create-schedule-modal.png` - Schedule creation modal

### Admin Panel Screenshots

#### Admin Tabs
- [ ] `admin-panel-overview.png` - Admin panel main interface
- [ ] `admin-providers-tab.png` - LLM and database provider config
- [ ] `admin-prompts-tab.png` - Runtime prompt editor
- [ ] `admin-prompt-edit-flow.png` - Prompt editing workflow
- [ ] `admin-email-tab.png` - Email/SMTP configuration
- [ ] `admin-scheduler-tab.png` - Scheduler settings
- [ ] `admin-insights-tab.png` - Insights engine config
- [ ] `admin-export-tab.png` - Export format settings
- [ ] `admin-history-tab.png` - Configuration change history
- [ ] `admin-snapshots-tab.png` - Configuration snapshots
- [ ] `admin-snapshot-flow.png` - Snapshot creation/restore flow

### Responsive Design
- [ ] `responsive-desktop.png` - Desktop layout (1920x1080)
- [ ] `responsive-tablet.png` - Tablet layout (768x1024)
- [ ] `responsive-mobile.png` - Mobile layout (375x667)

### Theme Screenshots
- [ ] `widget-dark-theme.png` - Dark theme widget
- [ ] `dashboard-dark-theme.png` - Dark theme dashboard

## Screenshot Guidelines

### Resolution
- **Desktop**: 1920x1080 or 1440x900
- **Tablet**: 768x1024
- **Mobile**: 375x667

### Format
- Use PNG format for best quality
- Optimize file size (recommended: 200-500KB per screenshot)
- Use tools like TinyPNG or ImageOptim

### Composition
- Include relevant UI elements only
- Crop unnecessary browser chrome (keep minimal context)
- Use realistic sample data (avoid Lorem Ipsum)
- Ensure text is readable
- Show actual query results from SQLatte

### Content Guidelines
- Use professional sample data (e.g., e-commerce, analytics)
- Blur or anonymize sensitive information
- Show meaningful queries and results
- Demonstrate actual features in action

### Naming Convention
All filenames should be:
- Lowercase
- Hyphen-separated
- Descriptive
- Match references in ui-guide.md

## Taking Screenshots

### Recommended Tools

**macOS:**
```bash
# Full screen
Cmd + Shift + 3

# Selected area
Cmd + Shift + 4

# Window with shadow
Cmd + Shift + 4, then Space, click window
```

**Windows:**
```bash
# Snipping Tool
Windows + Shift + S

# Or use Greenshot (recommended)
```

**Linux:**
```bash
# Gnome Screenshot
gnome-screenshot -a

# Or use Flameshot (recommended)
flameshot gui
```

### Browser DevTools for Responsive

1. Open DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Select device preset or custom dimensions
4. Take screenshot (right-click → Capture screenshot)

## Optimization

After taking screenshots, optimize them:

```bash
# Using ImageMagick
convert input.png -quality 85 -resize 1920x1080 output.png

# Using pngquant
pngquant --quality=65-80 input.png --output output.png

# Batch optimize all PNGs in directory
for file in *.png; do
  pngquant --quality=65-80 "$file" --output "optimized-$file"
done
```

## Sample Data Suggestions

### Queries to Screenshot
1. "Show me top 10 customers by revenue"
2. "What's the trend of daily sales last month?"
3. "List products with low inventory"
4. "Compare revenue by region"
5. "Show failed login attempts today"

### Schedules to Screenshot
1. "Daily Sales Report" - 9:00 AM daily
2. "Weekly KPI Dashboard" - Monday 8:00 AM
3. "Monthly Revenue Summary" - 1st of month

### Charts to Screenshot
- Bar chart: Revenue by product category
- Line chart: Daily sales trend
- Pie chart: Market share by region

## Verification Checklist

Before uploading screenshots:

- [ ] All required screenshots captured
- [ ] Filenames match documentation exactly
- [ ] Resolution appropriate for display
- [ ] File size optimized (<500KB preferred)
- [ ] No sensitive data visible
- [ ] Text is sharp and readable
- [ ] Colors accurate (especially dark theme)
- [ ] Features clearly demonstrated
- [ ] Professional sample data used

## Integration with Documentation

After adding screenshots to this directory:

1. Images will automatically appear in ui-guide.md
2. Relative paths are pre-configured: `../assets/images/ui/filename.png`
3. No code changes needed if filenames match

## Placeholder Image

If a screenshot is not yet available, the documentation will show:
- Broken image icon (intentional)
- Alt text describing what should be shown
- This serves as a TODO list for screenshots

## Contributing Screenshots

To contribute screenshots:

1. Fork the SQLatte repository
2. Add screenshots to `docs/assets/images/ui/`
3. Ensure filenames match this README
4. Verify in local MkDocs preview: `mkdocs serve`
5. Submit pull request with description

---

**Questions?** Open an issue on GitHub or contact the maintainers.
