---
description: App Store submission preparation, review rejections, metadata, screenshots.
---

# App Store Submission Sub-Skill

> Load this skill when preparing for App Store submission, handling app review, or dealing with app rejections.

## Core Principle

**App Review is not your enemy.** They enforce Apple's guidelines to maintain App Store quality. Follow the rules, be honest, and respond professionally to rejections.

## Pre-Submission Checklist

### Technical Requirements

- [ ] App builds and runs without crashes on physical devices
- [ ] Tested on latest iOS version and one version back
- [ ] Tested on smallest supported device (iPhone SE if supporting iOS 16)
- [ ] No placeholder content or "Lorem ipsum" text
- [ ] All features work as described
- [ ] Sign in flows work (provide test account if required)
- [ ] In-app purchases work in sandbox environment
- [ ] App works in Airplane Mode (or handles offline gracefully)
- [ ] No console warnings or excessive logging
- [ ] Location services request permission with clear usage description
- [ ] Camera/Photo access request permission with clear usage description
- [ ] Push notifications work (if implemented)

### App Store Connect Setup

- [ ] App name is unique and not trademarked by others
- [ ] Subtitle is concise (max 30 characters)
- [ ] Description is clear and accurate (avoid superlatives like "best")
- [ ] Keywords are relevant (100 character limit, comma-separated)
- [ ] Screenshots for all required device sizes
- [ ] App icon (1024x1024px, no transparency, no rounded corners)
- [ ] Privacy policy URL (required for almost all apps)
- [ ] Support URL
- [ ] Marketing URL (optional)
- [ ] App category is accurate
- [ ] Age rating is correct (use App Store Connect questionnaire)
- [ ] Pricing and availability configured

### Privacy Requirements

- [ ] Privacy policy published and linked
- [ ] Privacy nutrition labels filled out accurately
- [ ] Data collection explained in app and privacy policy
- [ ] User consent obtained before tracking (ATT framework)
- [ ] Third-party SDKs declared in privacy labels
- [ ] Kids apps comply with COPPA (if applicable)

### Legal & Content

- [ ] App doesn't violate copyright/trademark
- [ ] No misleading claims or false advertising
- [ ] Content rating is accurate
- [ ] Gambling apps have proper licenses (if applicable)
- [ ] Medical apps have disclaimers (if applicable)
- [ ] No inappropriate content for rated age

### Build Configuration

- [ ] Archive builds successfully in Xcode
- [ ] Build number incremented from previous submission
- [ ] Version number follows semantic versioning (1.0.0, 1.1.0, etc.)
- [ ] All architectures included (arm64)
- [ ] Bitcode disabled (deprecated by Apple)
- [ ] App Thinning enabled
- [ ] Minimum iOS version set correctly

## Common Rejection Reasons & How to Avoid

### 1. Guideline 2.1 - App Completeness

**Rejection:** App crashes, has bugs, or incomplete features.

**How to Avoid:**

- Test thoroughly on real devices
- Fix all crashes reported in Xcode Organizer
- Remove beta/demo labels
- Ensure all features work as described
- Don't submit if critical features are broken

**If Rejected:** Fix bugs, upload new build, resubmit.

### 2. Guideline 2.3.10 - Accurate Metadata

**Rejection:** Screenshots or description don't match actual app.

**How to Avoid:**

- Screenshots must show actual app interface
- Don't use marketing graphics instead of screenshots
- Description must accurately describe features
- Don't promise features that aren't implemented
- Version-specific features should be in "What's New"

**If Rejected:** Update screenshots/description in App Store Connect, resubmit (no new build needed).

### 3. Guideline 3.1.1 - In-App Purchase

**Rejection:** Using external payment system for digital goods.

**How to Avoid:**

- Use Apple's In-App Purchase for digital content
- Don't link to external payment sites for subscriptions
- RevenueCat/StoreKit is required
- Physical goods can use external payment (Stripe, etc.)

**If Rejected:** Remove external payment links, implement IAP, upload new build.

### 4. Guideline 4.2 - Minimum Functionality

**Rejection:** App is too simple or just a repackaged website.

**How to Avoid:**

- Add native iOS features (notifications, widgets, offline mode)
- Don't submit single-page web view apps
- Provide unique value beyond website
- Polish UI with native components

**If Rejected:** Add more features, make it feel native, resubmit.

### 5. Guideline 4.3 - Spam / Duplicate Apps

**Rejection:** Too similar to your other apps or other apps on store.

**How to Avoid:**

- Don't create multiple apps with minor variations
- Don't copy other apps
- Add unique features that differentiate your app
- Consider making one app with multiple themes instead of separate apps

**If Rejected:** Consolidate apps or add significant differentiation.

### 6. Guideline 5.1.1 - Privacy / Data Collection

**Rejection:** Privacy policy missing or data collection not disclosed.

**How to Avoid:**

- Always include privacy policy URL
- Accurately fill out privacy nutrition labels
- Disclose all data collection in app and policy
- Get user consent before collecting data
- Implement ATT (App Tracking Transparency) if tracking users

**If Rejected:** Add privacy policy, update nutrition labels, upload new build if code changes needed.

### 7. Guideline 5.1.2 - Privacy / Permission Requests

**Rejection:** Permission request strings are missing or unclear.

**How to Avoid:**

Add clear usage descriptions in Info.plist:

```xml
<key>NSCameraUsageDescription</key>
<string>We need camera access to let you take photos for your profile</string>

<key>NSPhotoLibraryUsageDescription</key>
<string>We need photo library access to let you choose a profile picture</string>

<key>NSLocationWhenInUseUsageDescription</key>
<string>We use your location to show nearby events</string>

<key>NSUserTrackingUsageDescription</key>
<string>We use tracking to show you personalized ads</string>
```

**If Rejected:** Update Info.plist with clear descriptions, upload new build.

### 8. Guideline 2.3.8 - Metadata Rejection (Test Account)

**Rejection:** App requires login but no test account provided.

**How to Avoid:**

- Provide working demo account in App Review Information
- Include detailed instructions if login flow is complex
- Make sure test account has access to all features
- Don't let test account expire during review

**If Rejected:** Provide test account in App Store Connect, resubmit (no new build needed).

### 9. Guideline 2.3.1 - Hidden Features

**Rejection:** App has features not visible to reviewers.

**How to Avoid:**

- Don't hide features behind server-side flags during review
- Provide demo mode or test account with full access
- Document all features in review notes
- Don't enable features only in certain regions if reviewers can't access

**If Rejected:** Enable all features for reviewers, provide instructions in review notes.

### 10. Guideline 1.1.6 - False Information

**Rejection:** Providing false info or impersonating another company.

**How to Avoid:**

- Use accurate developer name
- Don't claim affiliation with companies you're not affiliated with
- Don't use trademarked names without permission
- Be honest about app purpose and functionality

**If Rejected:** Update metadata with accurate information, may need legal documentation.

## Screenshot Requirements

### Sizes Required (as of 2026)

- **6.7" display (iPhone 15 Pro Max)**: 1290 x 2796 pixels
- **6.5" display (iPhone 14 Plus)**: 1284 x 2778 pixels
- **iPad Pro (12.9")**: 2048 x 2732 pixels (if supporting iPad)

### Screenshot Best Practices

1. **Show actual app interface** - No mockups or marketing graphics
2. **Highlight key features** - Show main screens and unique functionality
3. **Use text overlays sparingly** - Apple allows it but don't overdo it
4. **Localize screenshots** - Provide localized versions if supporting multiple languages
5. **Order matters** - First 2-3 screenshots are most important (visible without scrolling)

### Tools for Screenshots

**Xcode Simulator:**

```
1. Run app in simulator
2. Navigate to screen
3. Cmd + S to save screenshot
4. Screenshots saved to Desktop
```

**Fastlane Snapshot:**

```bash
# Install
gem install fastlane

# Setup snapshot
fastlane snapshot init

# Configure Snapfile
fastlane snapshot
```

**Third-party Tools:**

- [App Store Screenshot Generator](https://www.appscreenshot.com/) - Add text overlays
- [Figma](https://www.figma.com/) - Design screenshot frames
- [Sketch](https://www.sketch.com/) - Design screenshot frames

## App Review Times

**Typical Review Times (2026):**

- First submission: 24-48 hours
- Updates: 12-24 hours
- Expedited review: 2-4 hours (rare, only for critical bugs)

**Expedite Review Request:**

Use only for:

- Critical bug fixes affecting many users
- Time-sensitive events (Olympics coverage, holiday features)
- Legal requirements

Don't use for:

- Missed deadlines
- New features
- Marketing launches

Request via App Store Connect → App Review → Request Expedited Review

## App Review Notes

**Always include:**

- Test account credentials (if login required)
- Special instructions for testing features
- Explanation of non-obvious features
- Regional limitations (if any)
- Demo mode instructions (if applicable)

**Example:**

```
Test Account:
Email: reviewer@example.com
Password: TestPass123!

Instructions:
1. Sign in with provided test account
2. Navigate to "Premium Features" tab to test subscription flow
3. Use sandbox environment for IAP testing
4. Push notifications require "Allow Notifications" on first launch

Notes:
- Live chat feature is only available 9am-5pm PST
- Some content is regional (US only)
- Test account has premium subscription active
```

## Handling Rejections

### Step 1: Read Carefully

- Read rejection message completely
- Identify specific guideline violated
- Check attached screenshot (if provided)

### Step 2: Understand the Issue

- Google the guideline number
- Read Apple's documentation
- Check forums (Reddit r/iOSProgramming, Apple Developer Forums)

### Step 3: Fix the Issue

- Make required changes
- Test thoroughly
- Update metadata if needed (sometimes no new build required)

### Step 4: Respond Appropriately

**If you agree:**

- Fix issue
- Upload new build (if code change needed)
- Add resolution note in Resolution Center
- Resubmit

**If you disagree:**

- Politely explain in Resolution Center
- Provide evidence (screenshots, documentation)
- Reference specific guidelines
- Stay professional

**Example Response:**

```
Thank you for your review. We believe this may be a misunderstanding.

Our app uses In-App Purchase for all digital content purchases, as shown in [attached screenshot]. The "Buy" button mentioned in the rejection note actually triggers the Apple StoreKit payment flow, not an external payment system.

We've attached screenshots showing the complete IAP flow:
1. User taps "Buy Premium"
2. Apple's payment sheet appears
3. User authenticates with Face ID
4. Transaction completes via StoreKit

Please let us know if you need additional information or a demo account.
```

### Step 5: Appeal (if necessary)

- Use App Review Board appeal only if genuinely wrongly rejected
- Provide detailed explanation
- Include evidence (screenshots, code snippets, documentation)
- Reference specific guidelines
- Stay professional and factual

## Metadata Guidelines

### App Name

- Max 30 characters
- No generic terms ("The Best App")
- No emoji or special characters
- Can't include price or App Store category
- Can include subtitle after name ("MyApp - Photo Editor")

### Subtitle

- Max 30 characters
- Describe app succinctly
- No promotional text

### Keywords

- Max 100 characters total
- Comma-separated (no spaces after commas)
- No app name, category, or company name
- Use singular and plural (Apple indexes both)
- Research competitor keywords

**Example:**

```
photo,editor,filter,collage,camera,effect,beauty,retouch,enhance,professional
```

### Description

- Max 4000 characters
- First 2-3 sentences most important (visible before "more")
- Use bullet points for features
- No spam or keyword stuffing
- No competitor names
- No future promises ("Coming soon")

**Good Example:**

```
PhotoPro is a powerful photo editor designed for iOS.

KEY FEATURES:
• Professional filters and effects
• Advanced retouching tools
• Batch editing for multiple photos
• Cloud sync across devices
• Share to Instagram, Facebook, Twitter

PERFECT FOR:
- Portrait photography
- Landscape editing
- Social media content
- Professional photographers

PREMIUM SUBSCRIPTION:
Unlock all features with PhotoPro Premium. Try free for 7 days.

Download PhotoPro today and transform your photos!
```

### What's New

- Max 4000 characters
- Describe changes in this version
- Use bullet points
- Be specific ("Fixed crash when uploading photos" not "Bug fixes")

## Post-Approval

### Monitor Crashes

Check Xcode Organizer → Crashes regularly:

```
Xcode → Window → Organizer → Crashes
```

### Respond to Reviews

- Reply to user reviews professionally
- Thank positive reviews
- Address negative reviews with solutions
- Encourage users to update if issue was fixed

### Update Regularly

- Fix bugs quickly
- Add requested features
- Update for new iOS versions
- Refresh screenshots annually

## Testing Builds

### TestFlight

Before submitting to App Review, test with TestFlight:

```
1. Archive build in Xcode
2. Upload to App Store Connect
3. Add internal testers (automatic, no review)
4. Add external testers (requires beta review, faster than app review)
5. Gather feedback
6. Fix issues
7. Submit for App Review
```

### Internal Testing

- Up to 100 internal testers
- No review required
- Immediate availability after upload
- 90-day build expiry

### External Testing

- Up to 10,000 external testers
- Requires beta app review (faster than full review)
- Public link option available
- 90-day build expiry

## Phased Release

Enable phased release to gradually roll out updates:

```
App Store Connect → App → Versions → Phased Release
```

- Day 1: 1% of users
- Day 2: 2%
- Day 3: 5%
- Day 4: 10%
- Day 5: 20%
- Day 6: 50%
- Day 7: 100%

Pause or halt if critical bugs discovered.

## App Store Optimization (ASO)

### Icon

- Simple and recognizable at small sizes
- Consistent with brand
- No text (icon should be self-explanatory)
- Test at different sizes

### Screenshots

- First 2-3 are most important
- Show unique features
- Add captions to explain features
- Use actual app interface
- Update seasonally if relevant

### Localization

- Localize metadata for key markets
- Use native speakers for translations
- Don't use Google Translate for app descriptions
- Localize screenshots if possible

### Reviews & Ratings

- Ask for reviews at right time (after positive experience)
- Use `SKStoreReviewController` (limited to 3 prompts per year)
- Never incentivize reviews (against guidelines)
- Respond to reviews professionally

```swift
import StoreKit

func requestReview() {
    if let scene = UIApplication.shared.connectedScenes.first as? UIWindowScene {
        SKStoreReviewController.requestReview(in: scene)
    }
}

// Call after user completes meaningful action
```

## Bottom Line

Read the guidelines. Test thoroughly. Provide test accounts. Respond professionally to rejections. Keep metadata accurate. Use TestFlight before submitting. Monitor crashes post-launch. Update regularly.

**Key Resources:**

- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [App Store Connect Help](https://help.apple.com/app-store-connect/)
