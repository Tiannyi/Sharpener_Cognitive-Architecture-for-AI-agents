---
description: Load when implementing in-app purchases, subscriptions, paywalls, or entitlements using RevenueCat in iOS.
---

# RevenueCat Sub-Skill

> Load this skill when implementing in-app purchases, subscriptions, paywalls, or monetization features in iOS apps.

## Core Principle

**NEVER build custom subscription management.** RevenueCat handles receipt validation, cross-platform syncing, webhooks, subscriber status, and edge cases. You handle the paywall UI.

## Why RevenueCat?

| What You Get | Why It Matters |
|--------------|----------------|
| **Receipt validation** | Server-side validation prevents fraud |
| **Cross-platform** | One user ID across iOS/Android/Web |
| **Webhook integrations** | Automatic Slack/Discord/Email notifications |
| **Analytics** | Charts, MRR, churn, cohorts out of the box |
| **Restore purchases** | One line of code |
| **Family Sharing** | Handled automatically |
| **Intro offers** | Free trials, pay-up-front, pay-as-you-go |
| **Grace periods** | Billing retry handling |

## Setup Checklist

### 1. RevenueCat Dashboard Setup

1. Create account at app.revenuecat.com
2. Create new project
3. Add iOS app
4. Get API key (Public App-specific Key)
5. Create products and entitlements

### 2. App Store Connect Setup

1. Create in-app purchase products (subscriptions)
2. Note the Product IDs (e.g., `monthly_premium`, `yearly_premium`)
3. Set up subscription groups
4. Configure pricing and availability
5. Add subscription descriptions and screenshots

### 3. Link RevenueCat to App Store Connect

1. RevenueCat Dashboard → Apps → Your App → App Store Connect
2. Upload In-App Purchase Key (create in App Store Connect)
3. RevenueCat will sync products automatically

### 4. Swift Package Dependencies

```swift
dependencies: [
    .package(url: "https://github.com/RevenueCat/purchases-ios", from: "4.0.0")
]

// In target dependencies:
.product(name: "RevenueCat", package: "purchases-ios")
```

## Basic Integration

### App Initialization

```swift
import SwiftUI
import RevenueCat

@main
struct YourApp: App {
    init() {
        // Configure RevenueCat
        Purchases.logLevel = .debug // Remove in production
        Purchases.configure(withAPIKey: "appl_YOUR_API_KEY")
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Set User ID (after authentication)

```swift
import RevenueCat
import FirebaseAuth

func configureRevenueCatUser() {
    guard let userId = Auth.auth().currentUser?.uid else { return }

    Purchases.shared.logIn(userId) { customerInfo, created, error in
        if let error = error {
            print("Error logging in to RevenueCat: \(error.localizedDescription)")
        } else {
            print("User logged in to RevenueCat. Created: \(created)")
        }
    }
}
```

### Subscription Manager

```swift
import RevenueCat
import Combine

@Observable
class SubscriptionManager {
    var customerInfo: CustomerInfo?
    var offerings: Offerings?
    var isSubscribed: Bool {
        customerInfo?.entitlements["premium"]?.isActive == true
    }

    init() {
        fetchOfferings()
        fetchCustomerInfo()
    }

    func fetchOfferings() {
        Task {
            do {
                offerings = try await Purchases.shared.offerings()
            } catch {
                print("Error fetching offerings: \(error)")
            }
        }
    }

    func fetchCustomerInfo() {
        Task {
            do {
                customerInfo = try await Purchases.shared.customerInfo()
            } catch {
                print("Error fetching customer info: \(error)")
            }
        }
    }

    func purchase(package: Package) async throws {
        let result = try await Purchases.shared.purchase(package: package)
        customerInfo = result.customerInfo
    }

    func restorePurchases() async throws {
        customerInfo = try await Purchases.shared.restorePurchases()
    }
}
```

## Paywall UI

### Simple Paywall

```swift
import SwiftUI
import RevenueCat

struct PaywallView: View {
    @Environment(\.dismiss) private var dismiss
    @Environment(SubscriptionManager.self) private var subscriptionManager

    @State private var selectedPackage: Package?
    @State private var isPurchasing = false
    @State private var errorMessage: String?

    var body: some View {
        VStack(spacing: 20) {
            // Header
            VStack(spacing: 8) {
                Image(systemName: "crown.fill")
                    .font(.system(size: 60))
                    .foregroundColor(.yellow)

                Text("Upgrade to Premium")
                    .font(.title)
                    .bold()

                Text("Unlock all features and support development")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            }
            .padding(.top, 40)

            // Features
            VStack(alignment: .leading, spacing: 12) {
                FeatureRow(icon: "checkmark.circle.fill", text: "Unlimited access")
                FeatureRow(icon: "checkmark.circle.fill", text: "No ads")
                FeatureRow(icon: "checkmark.circle.fill", text: "Premium support")
                FeatureRow(icon: "checkmark.circle.fill", text: "Exclusive features")
            }
            .padding()

            Spacer()

            // Packages
            if let offering = subscriptionManager.offerings?.current {
                VStack(spacing: 12) {
                    ForEach(offering.availablePackages, id: \.identifier) { package in
                        PackageButton(
                            package: package,
                            isSelected: selectedPackage?.identifier == package.identifier
                        ) {
                            selectedPackage = package
                        }
                    }
                }
                .padding(.horizontal)
            }

            // Purchase Button
            Button {
                purchasePackage()
            } label: {
                if isPurchasing {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: .white))
                } else {
                    Text("Subscribe Now")
                        .bold()
                }
            }
            .frame(maxWidth: .infinity)
            .padding()
            .background(selectedPackage != nil ? Color.blue : Color.gray)
            .foregroundColor(.white)
            .cornerRadius(12)
            .padding(.horizontal)
            .disabled(selectedPackage == nil || isPurchasing)

            // Restore & Terms
            HStack(spacing: 20) {
                Button("Restore Purchases") {
                    restorePurchases()
                }
                .font(.caption)

                Button("Terms") {
                    // Open terms URL
                }
                .font(.caption)

                Button("Privacy") {
                    // Open privacy URL
                }
                .font(.caption)
            }
            .padding(.bottom)

            if let error = errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
                    .padding()
            }
        }
        .onAppear {
            // Pre-select annual package
            if let offering = subscriptionManager.offerings?.current {
                selectedPackage = offering.annual ?? offering.availablePackages.first
            }
        }
    }

    private func purchasePackage() {
        guard let package = selectedPackage else { return }

        isPurchasing = true
        errorMessage = nil

        Task {
            do {
                try await subscriptionManager.purchase(package: package)
                dismiss()
            } catch {
                if let revenueCatError = error as? RevenueCat.ErrorCode {
                    if revenueCatError == .purchaseCancelledError {
                        // User cancelled, don't show error
                    } else {
                        errorMessage = error.localizedDescription
                    }
                } else {
                    errorMessage = error.localizedDescription
                }
            }
            isPurchasing = false
        }
    }

    private func restorePurchases() {
        isPurchasing = true
        errorMessage = nil

        Task {
            do {
                try await subscriptionManager.restorePurchases()
                if subscriptionManager.isSubscribed {
                    dismiss()
                } else {
                    errorMessage = "No active subscriptions found"
                }
            } catch {
                errorMessage = error.localizedDescription
            }
            isPurchasing = false
        }
    }
}

struct FeatureRow: View {
    let icon: String
    let text: String

    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: icon)
                .foregroundColor(.green)
            Text(text)
            Spacer()
        }
    }
}

struct PackageButton: View {
    let package: Package
    let isSelected: Bool
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            HStack {
                VStack(alignment: .leading) {
                    Text(package.storeProduct.localizedTitle)
                        .font(.headline)
                    if let intro = package.storeProduct.introductoryDiscount {
                        Text("Free for \(intro.subscriptionPeriod.periodTitle)")
                            .font(.caption)
                            .foregroundColor(.green)
                    }
                }

                Spacer()

                VStack(alignment: .trailing) {
                    Text(package.localizedPriceString)
                        .font(.headline)
                    if let period = package.storeProduct.subscriptionPeriod {
                        Text("per \(period.periodTitle)")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
            .padding()
            .background(isSelected ? Color.blue.opacity(0.1) : Color.gray.opacity(0.1))
            .cornerRadius(12)
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.blue : Color.clear, lineWidth: 2)
            )
        }
        .buttonStyle(.plain)
    }
}

extension SubscriptionPeriod {
    var periodTitle: String {
        let unit: String
        switch self.unit {
        case .day: unit = "day"
        case .week: unit = "week"
        case .month: unit = "month"
        case .year: unit = "year"
        @unknown default: unit = "period"
        }
        return value == 1 ? unit : "\(value) \(unit)s"
    }
}
```

### Content Gating

```swift
struct PremiumFeatureView: View {
    @Environment(SubscriptionManager.self) private var subscriptionManager
    @State private var showPaywall = false

    var body: some View {
        VStack {
            if subscriptionManager.isSubscribed {
                // Show premium content
                PremiumContentView()
            } else {
                // Show locked state
                VStack(spacing: 20) {
                    Image(systemName: "lock.fill")
                        .font(.system(size: 60))
                        .foregroundColor(.gray)

                    Text("Premium Feature")
                        .font(.title)

                    Text("Subscribe to unlock this feature")
                        .foregroundColor(.secondary)

                    Button("Upgrade to Premium") {
                        showPaywall = true
                    }
                    .buttonStyle(.borderedProminent)
                }
            }
        }
        .sheet(isPresented: $showPaywall) {
            PaywallView()
        }
    }
}
```

## Product Configuration

### Entitlements

Define entitlements in RevenueCat Dashboard:

- Entitlement ID: `premium` (what the user gets)
- Products: `monthly_premium`, `yearly_premium` (how they pay)

### Checking Entitlements

```swift
func checkEntitlement(identifier: String) -> Bool {
    guard let customerInfo = Purchases.shared.cachedCustomerInfo else {
        return false
    }
    return customerInfo.entitlements[identifier]?.isActive == true
}
```

## Receipt Validation

RevenueCat handles all receipt validation server-side. You never need to validate receipts manually.

```swift
// This is handled automatically by RevenueCat
// When you call purchase(), RevenueCat:
// 1. Initiates StoreKit transaction
// 2. Gets receipt from Apple
// 3. Validates receipt on RevenueCat servers
// 4. Returns CustomerInfo with updated entitlements
```

## Sandbox Testing

### Enable Sandbox Mode

```swift
#if DEBUG
Purchases.logLevel = .debug
#endif
```

### Testing Scenarios

1. **New purchase**: Use sandbox test account
2. **Restore purchases**: Delete app, reinstall, restore
3. **Subscription renewal**: Wait for auto-renewal (usually 5 minutes in sandbox)
4. **Cancellation**: Cancel via Settings → Apple ID → Subscriptions
5. **Refund**: Use App Store Connect sandbox tools

### Sandbox Test Users

Create in App Store Connect → Users and Access → Sandbox Testers

- Use fake email (test@example.com)
- Sign in only in Settings → App Store, not iCloud

## Common Gotchas

### 1. Sandbox Receipts Expire Fast

Sandbox subscriptions renew every 5 minutes and expire after 6 renewals (30 minutes total).

### 2. StoreKit Configuration File

For SwiftUI previews and testing without sandbox account:

```
File → New → File → StoreKit Configuration File
Add your product IDs with test pricing
```

### 3. Restore Purchases Button Required

Apple requires a "Restore Purchases" button in your paywall or settings.

### 4. Family Sharing

RevenueCat automatically handles Family Sharing. Check:

```swift
customerInfo.entitlements["premium"]?.isActive
```

### 5. Promo Codes

Promo codes work automatically. Users redeem in App Store, RevenueCat syncs.

### 6. Subscription Status Updates

Listen for customer info updates:

```swift
Purchases.shared.getCustomerInfo { customerInfo, error in
    if let customerInfo = customerInfo {
        // Update UI based on subscription status
    }
}
```

## Offering Configuration

### Define Offerings in RevenueCat

```
Offerings → Create Offering → Default (current)
Add packages:
- $rc_monthly (monthly subscription)
- $rc_annual (annual subscription)
- $rc_lifetime (one-time purchase)
```

### Access in Code

```swift
let offerings = try await Purchases.shared.offerings()
let currentOffering = offerings.current

// Get specific packages
let monthly = currentOffering?.monthly
let annual = currentOffering?.annual
let lifetime = currentOffering?.lifetime

// Or custom package
let custom = currentOffering?.package(identifier: "my_custom_package")
```

## Analytics & Webhooks

### Built-in Analytics

RevenueCat Dashboard shows:

- Active subscriptions
- Monthly Recurring Revenue (MRR)
- Churn rate
- Conversion rate
- Trial conversion

### Webhooks

Configure webhooks in RevenueCat Dashboard:

- Slack: Get notified of new subscriptions
- Discord: Post subscriber updates
- Custom webhook: POST to your server on events

### Events

RevenueCat sends events for:

- Initial purchase
- Renewal
- Cancellation
- Billing issue
- Refund
- Expiration

## Complete Example

```swift
import SwiftUI
import RevenueCat

@main
struct MyApp: App {
    @State private var subscriptionManager = SubscriptionManager()

    init() {
        Purchases.configure(withAPIKey: "appl_YOUR_API_KEY")
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(subscriptionManager)
        }
    }
}

struct ContentView: View {
    @Environment(SubscriptionManager.self) private var subscriptionManager
    @State private var showPaywall = false

    var body: some View {
        NavigationStack {
            VStack {
                if subscriptionManager.isSubscribed {
                    Text("You're a premium member!")
                        .font(.title)
                } else {
                    Button("Upgrade to Premium") {
                        showPaywall = true
                    }
                }
            }
            .sheet(isPresented: $showPaywall) {
                PaywallView()
            }
        }
    }
}
```

## Pricing Best Practices

1. **Offer annual plan**: Usually 30-40% discount vs monthly
2. **Free trial**: 7 days is standard, 3 days for higher-priced apps
3. **Intro offer**: First month discounted
4. **Lifetime option**: For users who hate subscriptions
5. **A/B test pricing**: Use RevenueCat Experiments

## When to Show Paywall

- **After onboarding**: When user understands value
- **Before premium feature**: When they try to use it
- **Settings screen**: Easy access to upgrade
- **After free tier limit**: "You've used 10/10 free credits"

## Bottom Line

Use RevenueCat. Don't validate receipts yourself. Don't build subscription management. Don't handle cross-platform syncing manually. Focus on building a paywall that converts.
