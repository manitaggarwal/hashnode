---
title: "Wallet Service in Spring Boot"
datePublished: Wed Apr 05 2023 08:11:23 GMT+0000 (Coordinated Universal Time)
cuid: clg4uc5z900050al47irlb6r4
slug: wallet-service
tags: kotlin, springboot, wallet, loyaltyprograms

---

Loyalty programs have become a popular way for businesses to retain customers and keep them engaged with their products or services. These programs are designed to reward loyal customers for their repeat business, which in turn, helps businesses increase customer retention rates, sales, and overall revenue.

In a Loyalty Management System (LMS), a wallet is a digital account that holds loyalty points or rewards earned by a customer. Loyalty program wallets are used to track and manage a customer's loyalty rewards, which can be redeemed for discounts, special offers, or other incentives.

In this blog post, we'll discuss how to write a wallet for LMS, using a Spring Boot program written using Reactor Netty.

The program has the following features:

1. The program allows businesses to create a loyalty program that rewards customers for their repeat business. Customers can earn these points by making purchases or performing certain actions.
    
2. Once customers have enrolled in the loyalty program, and request is received to award the points the program creates a wallet for them with a certain expiry date. The wallet holds the customer's earned points or rewards, which can be redeemed for discounts or other incentives.
    
3. The program automatically maintains a summary of the wallet for each loyalty program, which makes it easier for businesses to keep track of their customers' balances.
    

### Creating a Wallet

```kotlin
override fun createWallet(
    createWalletRequest: CreateWalletRequest,
    program: Program
): Mono<Wallet> {
    validateCreateWalletRequest(createWalletRequest)
    return walletRepository.save(
        Wallet(
            createWalletRequest.userId,
            createWalletRequest.earned,
            createWalletRequest.spent,
            0,
            WalletStatus.ACTIVE,
            program.id,
            createWalletRequest.expiryDate,
            createWalletRequest.transactionId
        )
    ).doOnSuccess(this::summarizeWallet)
        .onErrorResume(DuplicateKeyException::class.java) {
            Mono.error(
                IllegalArgumentException(
                    "Wallet already exists for transaction ${createWalletRequest.transactionId}"
                )
            )
        }
}
```

### Creating a Wallet Summary

```kotlin
fun summarizeWallet(wallet: Wallet) {
    walletSummaryRepository
        .findWalletSummaryByUserIdAndProgram(wallet.userId, wallet.program)
        .flatMap { walletSummary ->
            walletSummary.balance += (wallet.earned - wallet.spent)
            walletSummaryRepository.save(walletSummary)
        }
        .switchIfEmpty(
            walletSummaryRepository.save(
                WalletSummary(
                    wallet.userId,
                    wallet.earned - wallet.spent,
                    WalletStatus.ACTIVE,
                    wallet.program
                )
            )
        )
        .subscribe()
}
```

### Consuming from Wallet and updating Wallet Summary

```kotlin
override fun consumeWallet(
    consumeWalletRequest: ConsumeWalletRequest,
    program: Program
): Flux<Wallet> {
    validateConsumeWalletRequest(consumeWalletRequest, program)
    return walletRepository.findByUserIdAndProgramAndWalletStatusOrderById(
        consumeWalletRequest.userId,
        program.id,
        WalletStatus.ACTIVE
    ).collectList()
        .flatMapMany { wallets ->
            val consumedAmount = consumeWalletRequest.amount
            var remainingAmount = consumedAmount
            val consumedWallets = mutableListOf<Wallet>()
            wallets.forEach { wallet ->
                if (remainingAmount > 0) {

                    val amountToConsume =
                        remainingAmount.coerceAtMost((wallet.earned - wallet.spent))
                    wallet.spent += amountToConsume
                    if (wallet.spent == wallet.earned) {
                        wallet.walletStatus = WalletStatus.INACTIVE
                    }
                    remainingAmount -= amountToConsume
                    consumedWallets.add(wallet)
                }
            }
            if (remainingAmount > 0) {
                throw IllegalArgumentException("Insufficient balance")
            }
            walletRepository.saveAll(consumedWallets)
        }.doOnComplete { updateSummaryWallet(consumeWalletRequest, program) }
}
```

So, why are loyalty programs important for businesses? Here are some of the key benefits:

1. Increased Customer Retention: Loyalty programs encourage customers to come back to a business and continue purchasing their products or services. This results in increased customer retention rates, which can help businesses build a loyal customer base.
    
2. Increased Sales: When customers are rewarded for their repeat business, they are more likely to continue purchasing from a business. This can lead to increased sales and revenue for the business.
    
3. Better Customer Insights: Loyalty programs can provide businesses with valuable insights into their customers' behavior, preferences, and needs. This information can be used to improve products and services and to create more targeted marketing campaigns.
    
4. Competitive Advantage: Businesses that offer loyalty programs have a competitive advantage over those that do not. Customers are more likely to choose a business that offers rewards for their repeat business, which can help businesses stand out in a crowded marketplace.
    

In conclusion, loyalty programs are a powerful tool for businesses to retain customers, increase sales, and gain valuable customer insights. By leveraging the benefits of loyalty programs, businesses can build strong, lasting relationships with their customers and grow their business over time.

As always working code can be found on [github](https://github.com/manitaggarwal/wallet-service).