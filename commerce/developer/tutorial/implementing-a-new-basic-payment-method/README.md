# Implementing a New Basic Payment Method

This tutorial will show you how to add a new basic payment method by implementing  the `CommercePaymentMethod` interface.

Liferay Commerce provides several out of the box payment methods including _Authorize.Net_, _Mercanet_, _Money Order_, and _PayPal_.

![Out of the box payment methods](./images/01.png "Out of the box payment methods")

## Overview

1. **Deploy an Example**
1. **Walkthrough the Example**
1. **Additional Information**

## Deploy an Example

In this section, we will get an example payment method up and running on your instance of Liferay Commerce. Follow these steps:

1. Start Liferay Commerce.

    ```bash
    docker run -it -p 8080:8080 liferay/commerce:2.0.1
    ```

1. Download and unzip the [Acme Commerce Payment Method Sample]() to your project directory.

    ```bash
    curl liferay-b1c3.zip
    ```

    ```bash
    unzip liferay-b1c3.zip
    ```

1. Go to `b1c3-impl`.

    ```bash
    cd b1c3-impl
    ```

1. Build and deploy the sample.

    ```bash
    ./gradlew deploy -DdockerContainerId=$(docker ps -lq)
    ```

    >Note: This command is the same as copying the deployed jars to /opt/liferay/osgi/modules on the Docker container.

1. Confirm the deployment in the Liferay Docker container console.

    ```bash
    STARTED com.acme.b1c3.internal.commerce.payment.method_1.0.0
    ```

1. Verify that the example new basic payment method was added. Open your browser to `https://localhost:8080` and navigate to Payment Methods.

   ![Acme Commerce Payment Method Added](./images/02.png "Acme Commerce Payment Method Added")

Congratulations, you've successfully built and deployed a new basic payment method that implements `CommercePaymentMethod`.

Next, let's dive deeper to learn more.

## Walkthrough the Example

In this section, we will take a more in-depth review of what happened in the above sample. First, we will annotate the class for OSGi registration; second we will implement and understand the required methods; and third, we will implement and understand the optional methods.

### Annotate Your Class for OSGi Registration

```java
@Component(
	immediate = true,
	property = "commerce.payment.engine.method.key=" + B1C3CommercePaymentMethod.KEY,
	service = CommercePaymentMethod.class
)
public class B1C3CommercePaymentMethod implements CommercePaymentMethod {

	public static final String KEY = "b1c3-commerce-payment-method";
```

>It is important to provide a distinct key for your payment method so that Liferay Commerce can distinguish your new payment method from others in the payment method registry. Reusing a key that is already in use will override the existing associated method.

### Required Methods

The following five methods are required to implement `CommercePaymentMethod`:

```java
	public String getDescription(Locale locale);
```

```java
	public String getKey();
```

```java
	public String getName(Locale locale);
```

```java
	public int getPaymentType();
```

```java
	public String getServletPath();
```

To better understand each of the required methods mentioned above, let's look at [B1C3CommercePaymentMethod.java](./liferay-b1c3.zip/b1c3-impl/src/main/java/com/acme/b1c3/internal/commerce/payment/method/B1C3CommercePaymentMethod.java). We will review the implementation of each of the five required methods in sequence.

1. `public String getDescription(Locale locale);`

    ```java
		@Override
		public String getDescription(Locale locale) {
			ResourceBundle resourceBundle = ResourceBundleUtil.getBundle("content.Language", locale, getClass());

			return LanguageUtil.get(resourceBundle, "b1c3-commerce-payment-method-description");
		}
    ```

    >Populates the "Description" column in the Payment Methods administration page. `ResourceBundleUtil` is a Liferay class that provides support for multiple locales. Visit [Supporting Multiple Locales on Liferay]() for more information.

1. `public String getKey();`

    ```java
		@Override
		public String getKey() {
			return "b1c3-commerce-payment-method-registry-key";
		}
    ```

    >Provides a unique identifier for the payment method in the registry. The key can be used to fetch the payment method from the registry programmatically if necessary. Reusing a key that is already in use will override the existing associated method.

1. `public String getName(Locale locale);`

    ```java
		@Override
		public String getName(Locale locale) {
			ResourceBundle resourceBundle = ResourceBundleUtil.getBundle("content.Language", locale, getClass());

			return LanguageUtil.get(resourceBundle, "b1c3-commerce-payment-method-name");
		}
    ```

    >Populates the "Name" column in the Payment Methods administration page. `ResourceBundleUtil` is a Liferay class that provides support for multiple locales. Visit [Supporting Multiple Locales on Liferay]() for more information.

1. `public int getPaymentType();`

    ```java
		@Override
		public int getPaymentType() {
			return CommercePaymentConstants.COMMERCE_PAYMENT_METHOD_TYPE_OFFLINE;
		}
    ```

    >Payment Type identifies the way in which the Payment Engine will use a given Payment Method. In this example, `COMMERCE_PAYMENT_METHOD_TYPE_OFFLINE` informs the payment engine that there are no online processing requirements for this Payment Method. There are two other payment type constants available out of the box: `COMMERCE_PAYMENT_METHOD_TYPE_ONLINE_STANDARD` and `COMMERCE_PAYMENT_METHOD_TYPE_ONLINE_REDIRECT`. For more information see [Implementing a New Online Payment Method]().

1. `public String getServletPath()`

    ```java
		@Override
		public String getServletPath() {
			return null;
		}
    ```

    >When implementing an online payment method, a servlet may be required to enable proper interfacing with an external payment provider. Since no servlet path is required for an offline payment method, `getServletPath` should return `null`.

### Optional Methods

The following methods are not required to implement the interface but are required to implement your custom payment logic. There are many more methods that can be implemented that provide additional functionality - such as subscriptions, recurring payments, and refunds. These can be seen in the [`CommercePaymentMethod`](https://github.com/liferay/com-liferay-commerce/blob/7.1.x/commerce-api/src/main/java/com/liferay/commerce/payment/method/CommercePaymentMethod.java) interface. These methods come in pairs: one method to enable and the other to implement a given piece of functionality.

1. `public boolean isCompleteEnabled()`

    ```java
		@Override
		public boolean isCompleteEnabled() {
			return true;
		}
    ```

    >Because we are defining our own custom payment logic, this must be set to true.

1. `public CommercePaymentResult completePayment()`

    ```java
		@Override
		public CommercePaymentResult completePayment(
				CommercePaymentRequest commercePaymentRequest)
			throws Exception {

			return new CommercePaymentResult(
				null, commercePaymentRequest.getCommerceOrderId(),
				CommerceOrderConstants.PAYMENT_STATUS_PAID, false, null, null,
				Collections.emptyList(), true);
		}
    ```

    >This is where we have defined our custom payment logic.

1. `public boolean isProcessPaymentEnabled()`

    ```java
		@Override
		public boolean isProcessPaymentEnabled() {
			return true;
		}
    ```

    >Because we are defining our own custom payment processing logic, this must be set to true.

1. `public CommercePaymentResult processPayment()`

    ```java
		@Override
		public CommercePaymentResult processPayment(
				CommercePaymentRequest commercePaymentRequest)
			throws Exception {

			return new CommercePaymentResult(
				null, commercePaymentRequest.getCommerceOrderId(),
				CommerceOrderConstants.PAYMENT_STATUS_AUTHORIZED, false, null, null,
				Collections.emptyList(), true);
		}
    ```

    >This is where we have defined our custom payment processing logic.

## Conclusion

Congratulations! You now know the basics for implementing the `CommercePaymentMethod` interface and have added a new basic payment method to Liferay Commerce.

## Additional Information

* [Supporting Multiple Locales on Liferay]()
* [Implementing a New Online Payment Method]()