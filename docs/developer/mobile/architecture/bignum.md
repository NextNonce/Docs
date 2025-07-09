

As an application that handles financial data, ensuring numerical accuracy is paramount. To prevent floating-point inaccuracies, the app strictly avoids using `Double` or `Float` for any calculations involving monetary values.

### The `BigDecimal` Implementation

All financial data, including token balances, prices, and historical changes, is represented using **`BigDecimal`**. This aligns with the backend API, which also provides financial data in a precise decimal format.

Finding a well-maintained Kotlin Multiplatform library for `BigDecimal` was a challenge. After research, the project adopted **`com.ionspin.kotlin:bignum`**. This library provides a reliable, cross-platform implementation of `BigDecimal` and, crucially, includes a serialization module (`bignum-serialization`) that integrates seamlessly with `kotlinx.serialization` for network operations.

### Centralized Formatting

To ensure consistency and maintainability, all logic for formatting `BigDecimal` values for display is centralized. This is achieved using Kotlin Multiplatform's `expect`/`actual` mechanism, which defines a common formatting API in `commonMain` while allowing for platform-specific implementations.

#### `expect` Declarations (in `commonMain`)

A set of `expect` functions defines the public API for formatting numbers. This allows any `ViewModel` or Composable in the shared codebase to format numbers without needing to know the underlying platform-specific details.

```kotlin
// commonMain/utils/Formatter.kt

// Formats a quote currency amount (e.g., $1,234.56)
expect fun formatQuote(amount: BigDecimal, showDecimal: Boolean = true): String

// Formats the change in a quote currency (e.g., +$12.34)
expect fun formatQuoteChange(amount: BigDecimal, showDecimal: Boolean = true): String

// Formats a balance of token (e.g., 1.524 ETH)
expect fun formatBalanceNative(amount: BigDecimal, symbol: String): String

// Formats a percentage value (e.g., +5.21%)
expect fun formatPercentage(amount: BigDecimal): String
```

-----

#### `actual` Implementations

Each platform provides its own `actual` implementation of these functions, using the best tools available on that platform.

##### Android Implementation

On Android, the `actual` functions use `java.text.DecimalFormat`, which offers powerful, pattern-based formatting. The logic dynamically selects the appropriate pattern based on the magnitude of the number.

```kotlin
// androidMain/utils/Formatter.android.kt
actual fun formatQuote(amount: BigDecimal, showDecimal: Boolean): String {
    val amountJvmBd = JvmBigDecimal(amount.toString())
    val pattern = when {
        showDecimal.not() -> "#,###"
        amountJvmBd >= JvmBigDecimal.ONE -> "#,###.00"
        amountJvmBd >= JvmBigDecimal("0.01") -> "0.00"
        amountJvmBd.compareTo(JvmBigDecimal.ZERO) == 0 -> "0.00"
        else -> "0.00000000"
    }

    val formatter = DecimalFormat(pattern)
    return "$" + formatter.format(amountJvmBd)
}
```

##### iOS Implementation

On iOS, the `actual` functions leverage the native `NSNumberFormatter`. This ensures that all numbers are formatted according to the user's specific locale and settings on their device, providing a more natural and platform-idiomatic experience.

```kotlin
// iosMain/utils/Formatter.ios.kt
actual fun formatQuote(amount: BigDecimal, showDecimal: Boolean): String {
    val formatter = NSNumberFormatter().apply {
        numberStyle = NSNumberFormatterDecimalStyle

        when {
            !showDecimal -> {
                minimumFractionDigits = 0uL
                maximumFractionDigits = 0uL
            }
            amount >= 0.01 || amount.compareTo(BigDecimal.ZERO) == 0  -> {
                minimumFractionDigits = 2uL
                maximumFractionDigits = 2uL
            }
            else -> {
                minimumFractionDigits = 8uL
                maximumFractionDigits = 8uL
            }
        }
    }

    val decimalNumber = NSDecimalNumber(string = amount.toString())
    return formatter.stringFromNumber(decimalNumber)?.let { "$$it" } ?: ""
}
```

This approach provides the best of both worlds: the guaranteed precision of `BigDecimal` for all calculations and the best possible platform-native user experience for displaying data.