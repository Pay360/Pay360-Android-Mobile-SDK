# Pay360 Android SDK

## Summary

This is Android mobile SDK for Pay360 to integrate Pay360 to Android mobile app.

It is available for native Android application(Java/Koltin) and other hybrid mobile framework for example React Native, Xamarin, etc

## Prerequisites for running/building the project

At minimum Android 4.1 (API level 14)

## Build SDK

Gradle->Drop down library name -> tasks-> build-> assemble  
 Double click the `assemble`  
 It will generate `aar` file in `build/outputs/aar`

## Getting Started

Once you have a Bearer token from the Pay360 authentication service you can start to build a payment to process.

A payment consist of all information required to make payment :-

- Transaction - Details regarding the transaction (eg. Amount, Currency)
- Card - Card details
- Billing Address - Address of card holder
- Customer - Details of card holder
- Financial Service - Extra details of card holder
- Credentials - Merchant details required by Pay360

Once a Payment object has been populated you can then start the payment process.

There are currently 4 flows:

- processGuestPayment - Process Card Details & Stores Card Details if merchant ref supplied.
- processVerifyRequest - Validates Card Details & Stores Card Details if merchant ref supplied.
- processTokenPayment - Processes a payment with the card token.
- processGooglePayResponse - Processes Google Pay payment, requires Google Pay

Examples of all flows can be seen in the sample app.

## Making a Payment

1. First need to create a Transaction containing all the details of the value of the payment.

   ```
     PPOTransaction transaction = new PPOTransaction()
       .setCurrency("GBP")
       .setAmount(100)
       .setTransactionDescription("Sample Transaction")
       .setMerchantRef("YOUR MERCHANT REFERENCE")
       .setIsRecurring("false")
       .setIsDeferred("false");
   ```

2. Create the Card Details.

   ```
   PPOCard card = new PPOCard()
     .setPan("9903000000000017")
     .setCv2("123")
     .setExpiryDate("0117")
     .setCardHolderName("John Smith");
   ```

   If making a payment with a card token then only the CV2 will required populating as all other card details are stored within Pay360.

   ```
   PPOCard card = new PPOCard()
     .setCv2("123");
   ```

3. Add Billing Address details. These are always required and must link to the card address.

   ```
   PPOBillingAddress address = new PPOBillingAddress()
     .setLine1("YOUR ADDRESS LINE 1")
     .setLine2("YOUR ADDRESS LINE 2")
     .setLine3("YOUR ADDRESS LINE 3")
     .setLine4("YOUR ADDRESS LINE 4")
     .setCity("YOUR CITY")
     .setRegion("YOUR REGION")
     .setPostcode("YOUR POSTCODE")
     .setCountryCode("GBR");
   ```

4. Add extra card holder details

   ```
     PPOFinancialService financialService = new PPOFinancialService()
       .setDateOfBirth("12041979")
       .setSurname("YOUR SURNAME")
       .setAccountNumber("YOUR FINANCE ACCOUNT")
       .setPostCode("YOUR FINANCE POSTCODE");
   ```

5. Add the customer, merchantRef is used to identify the customer to the merchant.

   ```
   PPOCustomer customer = new PPOCustomer()
     .setEmail("YOUR EMAIL ADDRESS")
     .setDateOfBirthday("120479")
     .setTelephone("YOUR TELEPHONE");
   ```

Include the merchant ref if you would card details stored and a card token returned.

```
customer..setMerchantRef("YOUR CUSTOMER REFERENCE");
```

6. Add Credentials. This must include your client token from the MITE API and the installation id provided to you by Pay360.

   ```
   PPOCredentials credentials = new PPOCredentials("Bearer Token", "InstallationId");
   ```

7. Build Payment ready for processing.

   Once all payment detais have been taking you can build the payment object,
   setting all values to the objects built.

   ```
   PPOPayment payment = new PPOPayment(this, this)
     .setCredentials(credentials)
     .setTransaction(transaction)
     .setCard(card)
     .setBillingAddress(address)
     .setCustomer(customer);
   ```

   The constructor of the payment takes in two arguements:

   - PPOPaymentDelegate - You must implement this delegate class and pass as the first argument.
   - Context - Android Application Context

8. Taking Payment

To take a payment you need to ensure you have implemented PPOPaymentDelegate correctly.

- cardPaymentProceedWithSuccess - Fired once the payment has completed succesfully.
- cardPaymentProceedWithFailure - Fired once the payment has NOT completed succesfully with reason for failure.

To start the payment process for a Guest Card can now be started by calling:

`payment.processGuestPayment()`

The SDK will then contact Pay360, display a 3DS window if required then proceed to attempt the payment calling the relevant delgate method one completion.

## Google Pay

1. To use Google Pay instantiate the PPOGooglePay class, passing in your google pay configuration values. This can be done within the onCreate method of your activity. The example from the sample app:

```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        List<String> acceptedCards = Arrays.asList("AMEX", "DISCOVER","JCB", "MASTERCARD", "VISA" );
        List<String> shippingCountries = Arrays.asList("US", "UK");
        this.googlePay = new PPOGooglePay(this.merchantId, this.merchantName,  acceptedCards, shippingCountries, this );
    }
```

2. You will also need to override the onActivityResult as this is called when the google screen has exited. The arguments should be passed back to the SDK.

```
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        this.payment.processGooglePayResponse(requestCode, resultCode, data);
    }
```

3. To start the Google Pay flow you need to pass your payment object to the PPOGooglePay pay method. You can see an example within the authenticatedWithSuccess delegate method.

```
    outer.payment = outer.buildPaymentBody(paymentType, use3DS);
    try {
        switch (paymentType) {
            case PPOGooglePayPayment:
                outer.googlePay.pay(outer.payment);
                break;
```
