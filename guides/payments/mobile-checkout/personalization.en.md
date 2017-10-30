---
sites_supported:
    - mla
    - mlb
    - mlm
    - mco
    - mcl
    - mpe
    - mlv
    - global
---
# **Customization**

> WARNING
>
> Prerequisites
>
> *  This guide presumes that you have already followed the steps in the introduction section of the documentation for installing the SDK and the default integration, along with the creation of the payment preference.

## Customize the payment preference

If necessary, when creating the payment preference on your server, you can specify restrictions such as specific exclusions of payment methods or types and set the maximum or the default number of installments.

#### Exclude Payment Methods

You can specify the types of payment method you do not want to accept (Cash, Credit or Debit Cards) by excluding them when creating the Checkout Preference.

In the content of the payment preference, you can add the payment methods or types of payment methods that you do not want to accept.

*Exclude a specific payment type*

```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
    "excluded_payment_types":[
        {"id":"ticket"}
    ]
  },
  ...
}
```

*Exclude more than one payment type:*


```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
    "excluded_payment_types":[
        {"id":"ticket"},
        {"id":"atm"},
        {"id":"debit_card"}
    ]
  },
  ...
}
```

You can even determine which specific payment methods (Visa, Mastercard, etc.) you want to exclude from the checkout:

*Exclude a specific payment method:*


```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
    "excluded_payment_methods":[
        {"id":"visa"}
    ]
  },
  ...
}
```

*Exclude more than one payment method:*


```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
    "excluded_payment_methods":[
        {"id":"visa"},
        {"id":"master"}
    ]
  },
  ...
}
```

#### Customize the Installments

You can specify the maximum number of installments you want to accept for your payment methods.

```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
  	 "excluded_payment_methods": [
  	   ...
  	 ],
  	 "excluded_payment_types": [
  	   ...
  	 ],
    "installments": 1
  },
  ...
}
```

You can also set the number of installments by default, which will be selected automatically, if available for the payment method selected by the user. Otherwise, the installment options will be displayed for the user to select one:


```json
{
  "items": [
    ...
  ],
  "payer": {
    ...
  },
  "payment_methods": {
  	 "excluded_payment_methods": [
  	   ...
  	 ],
  	 "excluded_payment_types": [
  	   ...
  	 ],
    "default_installments": 3
  },
  ...
}
```

## Customize the payment flow

The Flow Preference allows you to customize and set up the flow so that you can get the best payment experience.

In the `FlowPreference` class you can set up whether you want to display a screen with the summary of the payment details (Check and Confirm) or if you want to display discount campaigns, among other options.

In order to incorporate into the Checkout the options set up in the `FlowPreference` class, you must add a request for that in the beginning of the Checkout, as shown in the following code:

[[[
```android
	FlowPreference flowPreference = new FlowPreference.Builder()
                .disableReviewAndConfirmScreen()
                .disableDiscount()
                .disableBankDeals()
                .build();

        new MercadoPagoCheckout.Builder()
                .setActivity(this)
                .setPublicKey(mPublicKey)
                .setCheckoutPreference(mCheckoutPreference())
                .setFlowPreference(flowPreference)
                .startForPayment();
```
```swift
	let flowPrefernece = FlowPreference()
            flowPrefernece.disableReviewAndConfirmScreen()
            flowPrefernece.disableDiscount()
            flowPrefernece.disableBankDeals()

            MercadoPagoCheckout.setFlowPreference(flowPrefernece)

	 let checkout = MercadoPagoCheckout(publicKey: publicKey, accessToken: nil, checkoutPreference: checkoutPreference,
         navigationController: self.navigationController!)
	   checkout.start()
```
```Objective-c
	FlowPreference *flowPreference = [[FlowPreference alloc]init];
    [flowPreference disableReviewAndConfirmScreen];
    [flowPreference disableDiscount];
    [flowPreference disableBankDeals];
    [MercadoPagoCheckout setFlowPreference:flowPreference];

	-(void)startMercadoPagoCheckout:(CheckoutPreference *)checkoutPreference {
    		MercadoPagoCheckout *checkout = [[MercadoPagoCheckout alloc] initWithPublicKey: TEST_PUBLIC_KEY checkoutPreference:checkoutPreference discount:nil navigationController:self.navigationController];
		[checkout start];
	}
```
]]]

As you can see in the example, it is possible to hide the Promotions button using the `disableBankDeals` method for single-installment payments.

## Pay on your Server

If you need to perform any validation on your server at the time of making the payment, you can set up your own payment service.

In the `ServicePreference` class you can set up the URL and URI of your service together with a Map so that you can send the information you want.

At the moment of posting the payment, the SDK will do it at your service, [creating the payment](/reference/payments). and performing the validations inherent to your business. The SDK will expect to receive a payment, according to the response of MercadoPago service.

As soon as the `ServicePreference` has been created, you must start the payment flow of MercadoPago, as shown in the following code:

[[[

```android
        public void submit(View view) {
        CheckoutPreference checkoutPreference = new CheckoutPreference.Builder()
                .setSite(Sites.ARGENTINA)
                .addItem(new Item("Test Item", new BigDecimal("100")))
                .build();

        HashMap<String, Object> extraData = new HashMap<>();
        map.put("item_id", "id");

        ServicePreference servicePreference = new ServicePreference.Builder()
                .setCreatePaymentURL("https://www.tunombre.com", "/createPayment", extraData)
                .build();

        new MercadoPagoCheckout.Builder()
                .setActivity(this)
                .setServicePreference(servicePreference)
                .setPublicKey("TEST-0b74577e-863f-4a0e-9932-b87761cda03e")
                .setCheckoutPreference(checkoutPreference)
                .startForPayment();
    }
```
```swift
	let item = Item(_id: "Item_Id", title: "Remeras", quantity: [FAKER][NUMBER][BETWEEN][1,10], unitPrice: 50, description: nil, currencyId: "[FAKER][CURRENCY][ACRONYM]")
	let payer = Payer(_id: "Payer_Id", email: "sarasa@gmail.com", type: nil, identification: nil, entityType: nil)

	let checkoutPreference = CheckoutPreference()
	checkoutPreference.items = [item]
	checkoutPreference.payer = payer
	checkoutPreference.setId("MLA")

	let servicePreference = ServicePreference()
	servicePreference.setCreatePayment(baseURL: "https://your-base-URL.com/", URI: "your_create_preference_URI",
    additionalInfo: ["item_id" : "id", "quantity" : [FAKER][NUMBER][BETWEEN][1,10]])

	MercadoPagoCheckout.setServicePreference(servicePreference)

	 let checkout = MercadoPagoCheckout(publicKey: publicKey, accessToken: nil, checkoutPreference: checkoutPreference,
         navigationController: self.navigationController!)

	   checkout.start()
```
```Objective-c
	 Item *item = [[Item alloc] initWith_id:@"itemId" title:@"item title 2" quantity:[FAKER][NUMBER][BETWEEN][1,10] unitPrice:2 description:@"item description" currencyId:@"[FAKER][CURRENCY][ACRONYM]"];
    Payer *payer = [[Payer alloc] initWith_id:@"payerId" email:@"payer@email.com" type:nil identification:nil entityType:nil];

    NSArray *items = [NSArray arrayWithObjects:item, item, nil];

    self.pref = [[CheckoutPreference alloc] initWithItems:items payer:payer paymentMethods:nil];
	[self.pref setSiteId:@“MLA”];

	ServicePreference * servicePreference = [[ServicePreference alloc] init];
	 NSDictionary *extraParams = @{
                                  @"merchant_access_token" : @"mla-cards-data" };
	[servicePreference setCreatePaymentWithBaseURL:@"https://private-0d59c-mercadopagoexamples.apiary-mock.com" URI:@"/create_payment" additionalInfo:extraParams];
	[MercadoPagoCheckout setServicePreference:servicePreference];

	-(void)startMercadoPagoCheckout:(CheckoutPreference *)checkoutPreference {
		    self.mpCheckout = [[MercadoPagoCheckout alloc] initWithPublicKey: TEST_PUBLIC_KEY accessToken: nil checkoutPreference:checkoutPreference paymentData:nil discount:nil navigationController:self.navigationController paymentResult: nil];
    [self.mpCheckout start];
	}
```
]]]