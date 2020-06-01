---
title: Paytm Gateway Integration
date: 2020-01-02 14:43:06
author: Ashish Jajoria
categories:
- ["Android"]
- ["Ruby on Rails"]
tags: 
- paytm
- gateway
- payment
- backend
- android
- rails
- ror
- Ashish Jajoria
---
A complete guide on adding payments to your Android app with backend as RoR

{% asset_img img_flow_android_ios_sdk.png %}

## Steps :-

1. Install SDK
2. Add Static SMS Permission (for SMS autoread)
3. Add Runtime SMS Permission (for SMS autoread)
4. Add Proguard Rules
5. Get Order Checksum from Server (our Server)
6. Generate and send Checksum (from Our Server)
7. Start Payment Transaction
8. Send Payment Response to Server
9. Confirm with Paytm Gateway about payment status
10. Update Order Status
11. Show Order Status on App
&nbsp;
&nbsp;

### Step 1: Install SDK

Add the following dependency to your **app level** `build.gradle`.

```groovy
dependencies {
 ...

 // Paytm SDK
 implementation('com.paytm:pgplussdk:1.4.4') {
  transitive = true
 }
}
```

### Step 2: Add Static SMS Permission (for SMS autoread)

Add the following permissions to your `AndroidManifest.xml`.

```xml
<uses-permission android:name=”android.permission.READ_SMS”/>
<uses-permission android:name=”android.permission.RECEIVE_SMS”/>
```

### Step 3: Add Runtime SMS Permission (for SMS autoread)

We used [Dexter](https://github.com/Karumi/Dexter) library for handling runtime permissions. To install that add the following dependency to your **app level** `build.gradle`.

```groovy
dependencies {
 ...

 // Dexter runtime permissions
 implementation 'com.karumi:dexter:4.2.0'
}
```

Add the following code before starting your transaction process.

```kotlin
Dexter.withActivity(this)
      .withPermissions(
       android.Manifest.permission.READ_SMS”,
       android.Manifest.permission.RECEIVE_SMS”
      )
      .withListener(object : MultiplePermissionsListener {
       override fun onPermissionsChecked(report: MultiplePermissionsReport?) {
        report?.let {
         if (it.areAllPermissionsGranted()) {
          beginPaytmTransaction()
         } else {
          showMessage("Permission Denied")
         }
        }
       }

       override fun onPermissionRationaleShouldBeShown(permissions: MutableList<PermissionRequest>?, token: PermissionToken?) {
        token?.continuePermissionRequest()
       }
      })
      .withErrorListener {
       showMessage("Error occurred! $it")
      }
      .onSameThread()
      .check()
```

### Step 4: Add Proguard Rules

Add the following rules to your `proguard-rules.pro`.

```pro
-keepclassmembers class com.paytm.pgsdk.paytmWebView$PaytmJavaScriptInterface {
 public *;
}
```

### Step 5: Get Order Checksum from Server (our Server)

Make an API call to your server to get the order checksum

```kotlin
interface OurService {
 @GET("/subscriptions/new")
 suspend fun newSubscription(): NewSubscriptionResponse
}
```

```kotlin
class UserRepository(private val ourService: OurService) {
 suspend fun createNewSubscription(): NewSubscriptionResponse = ourService.newSubscription()
}
```

```kotlin
open class ActivityState
```

```kotlin
class ProfileViewModel(private val userRepository: UserRepository) : ViewModel() {
 private val _paytmState: MutableLiveData<ActivityState> = MutableLiveData(InitialState)
 val paytmState: LiveData<ActivityState> = _paytmState

 ...

 fun createNewSubscription() {
  viewModelScope.launch {
   try {
    _paytmState.value = ProgressState
    _paytmState.value = PaytmChecksumState(userRepository.createNewSubscription())
   } catch (e: HttpException) {
    _paytmState.value = ErrorState(e)
   } catch (e: IOException) {
    _paytmState.value = ErrorState(e)
   }
  }
 }

 object ProgressState : ActivityState()
 data class PaytmChecksumState(val checksumResponse: NewSubscriptionResponse) : ActivityState()
 data class ErrorState(val exception: Exception) : ActivityState()
}
```

```kotlin
class ProfileFragment : Fragment() {
 private val profileViewModel: ProfileViewModel by viewModel()

 ...

 private fun beginPaytmTransaction() {
  profileViewModel.createNewSubscription()
 }
}
```

### Step 6: Generate and send Checksum (from Our Server)

To your project directory add a package named **paytm**.

Add `checksum_tool.rb` and `encryption_new_pg.rb` to the **paytm** package from [Paytm_App_Checksum_Kit_Ruby](https://github.com/Paytm-Payments/Paytm_App_Checksum_Kit_Ruby/tree/master/paytm)

We will be creating order for a PaymentRequest and we follow model heavy approach for business logic. So we added a static method to generate checksum for our order in PaymentRequest model itself.

```ruby
class PaymentRequest < ApplicationRecord
 ...

 def self.create_checksum(user, order_id)
  require './paytm/encryption_new_pg.rb'
  require './paytm/checksum_tool.rb'
  require 'uri'

  paytm_hash = Hash.new

  is_staging = 'true' == ENV['PAYTM_STAGING']
  merchant_id = is_staging ? ENV['STAGING_PAYTM_MERCHANT_ID'] : ENV['PAYTM_MERCHANT_ID']
  industry_type = is_staging ? ENV['STAGING_PAYTM_INDUSTRY_TYPE'] : ENV['PAYTM_INDUSTRY_TYPE']
  paytm_website = is_staging ? ENV['STAGING_PAYTM_WEBSITE'] : ENV['PAYTM_WEBSITE']
  paytm_callback = is_staging ? ENV['STAGING_PAYTM_CALLBACK'] : ENV['PAYTM_CALLBACK']

  paytm_hash["REQUEST_TYPE"] = 'DEFAULT'
  paytm_hash["MID"] = merchant_id #Provided by Paytm
  paytm_hash["ORDER_ID"] = order_id; #unique OrderId for every request\
  paytm_hash["CUST_ID"] = user.id.to_s #unique customer identifier
  paytm_hash["INDUSTRY_TYPE_ID"] = industry_type #Provided by Paytm
  paytm_hash["CHANNEL_ID"] = 'WAP'; #Provided by Paytm
  paytm_hash["TXN_AMOUNT"] = '1'; #transaction amount
  paytm_hash["WEBSITE"] = paytm_website #Provided by Paytm
  paytm_hash["EMAIL"] = user.email; #customer email id
  if user.phone_number.present?
   paytm_hash["MOBILE_NO"] = user.phone_number; #customer 10 digit mobile no.
  end
  paytm_hash["CALLBACK_URL"] = paytm_callback + "#{order_id}"

  checksum_hash = ChecksumTool.new.get_checksum_hash(paytm_hash).gsub("\n", '')
  paytm_hash["CHECKSUMHASH"] = checksum_hash
  paytm_hash
 end
end
```

First create a subscription for current user, then create a payment request for that subscription and create checksum treating that payment request as your Order.

```ruby
class SubscriptionsController < ApplicationController
 before_action :authenticate_user!

 def new
  user = current_user
  subscription = user.subscriptions.create!
  payment_request = subscription.payment_requests.create!

  checksum = PaymentRequest.create_checksum(user, payment_request.id)
  render json: { paytm_params: checksum, is_staging: 'true' == ENV['PAYTM_STAGING']}, status: :ok
 end
end
```

### Step 7: Start Payment Transaction

With checksum response from server, initiate the paytm purchase.

```kotlin
class ProfileFragment : Fragment() {
 ...

 private fun initiatePaytmPurchase(checksumResponse: NewSubscriptionResponse) {
  val order = PaytmOrder(checksumResponse.paytmParams)
  val service = if (checksumResponse.isStaging)
     PaytmPGService.getStagingService(null)
    else
     PaytmPGService.getProductionService()
  
  service.initialize(order, null)

  service.startPaymentTransaction(context, true, true, object : PaytmPaymentTransactionCallback {
   override fun onTransactionResponse(inResponse: Bundle?) {
   }

   override fun clientAuthenticationFailed(inErrorMessage: String?) {
   }

   override fun someUIErrorOccurred(inErrorMessage: String?) {
   }

   override fun onTransactionCancel(inErrorMessage: String?, inResponse: Bundle?) {
   }

   override fun networkNotAvailable() {
   }

   override fun onErrorLoadingWebPage(iniErrorCode: Int, inErrorMessage: String?, inFailingUrl: String?) {
   }

   override fun onBackPressedCancelTransaction() {
   }
  })
 }
}
```

![Demo of Paytm checkout flow in app](merchant_pg_android.gif)

### Step 8: Send Payment Response to Server

On Transaction response from paytm payments Activity, send the response to your server to update payment status.

```kotlin
interface OurService {
 @POST("/payment_requests/{requestId}/update_status")
 suspend fun updatePaymentStatus(@Path(value = "requestId") requestId: Int,
                                 @Body transactionResponse: JsonObject): UpdatePaymentResponse
}
```

```kotlin
class UserRepository(private val ourService: OurService) {
 suspend fun updatePaymentResponse(requestId: Int, transactionResponse: JsonObject): UpdatePaymentResponse = ourService.updatePaymentStatus(requestId, transactionResponse)
}
```

```kotlin
open class ActivityState
```

```kotlin
class ProfileViewModel(private val userRepository: UserRepository) : ViewModel() {
 private val _paytmState: MutableLiveData<ActivityState> = MutableLiveData(InitialState)
 val paytmState: LiveData<ActivityState> = _paytmState

 ...

 fun updatePaymentStatus(requestId: Int, transactionResponse: JsonObject) {
  viewModelScope.launch {
   try {
    _paytmState.value = ProgressState
    val response = userRepository.updatePaymentResponse(requestId, transactionResponse)
    _paytmState.value = PaytmStatusState(response)
    _paytmState.value = PaytmIdleState
   } catch (e: HttpException) {
    _paytmState.value = ErrorState(e)
   } catch (e: IOException) {
    _paytmState.value = ErrorState(e)
   }
  }
 }

 object ProgressState : ActivityState()
 data class PaytmStatusState(val updatePaymentResponse: UpdatePaymentResponse) : ActivityState()
 object PaytmIdleState : ActivityState()
 data class ErrorState(val exception: Exception) : ActivityState()
}
```

```kotlin
class ProfileFragment : Fragment() {
 ...

 override fun onTransactionResponse(inResponse: Bundle?) {

  val orderId = inResponse?.getString("ORDERID")
  orderId?.let {
   val responseJson = JsonObject()
   inResponse.keySet()?.forEach {
    responseJson.addProperty(it, inResponse.getString(it))
   }
   val responseJsonWrapper = JsonObject()
   responseJsonWrapper.add("gateway_response", responseJson)

   profileViewModel.updatePaymentStatus(Integer.parseInt(orderId), responseJsonWrapper)
  }
 }

 ...
}
```

### Step 9: Confirm with Paytm Gateway about payment status

Confirm with Paytm gateway using [Transaction Status API](https://developer.paytm.com/docs/transaction-status-api/).

```ruby
class PaymentRequest < ApplicationRecord
 ...

 def confirm_with_gateway(user)
  is_staging = 'true' == ENV['PAYTM_STAGING']
  status_api_url = is_staging ? ENV['STAGING_PAYTM_STATUS'] : ENV['PAYTM_STATUS']
  merchant_id = is_staging ? ENV['STAGING_PAYTM_MERCHANT_ID'] : ENV['PAYTM_MERCHANT_ID']
  order_id = self.id

  response = HTTParty.post(status_api_url,
                           body: {
                                  MID: merchant_id,
                                  ORDERID: order_id,
                                  CHECKSUMHASH: PaymentRequest.create_checksum(user, order_id)["CHECKSUMHASH"]
                                 }.to_json,
                           multipart: false,
                           headers: {
                                     'Content-Type' => 'application/json'
                                    },
                           timeout: 10000)
  update_status(response) # We will learn about this in next step
 end
end
```

After confirming with gateway, send back the response to App.

```ruby
class PaymentRequestsController < ApplicationController
 before_action :authenticate_user!

 def update_status
  payment_request = PaymentRequest.find(params[:id])
  payment_request.initiate! #We are using aasm gem for this https://github.com/aasm/aasm
  payment_request.confirm_with_gateway(current_user)
  payment_request.reload

  render json: {
                message: get_status_message(payment_request),
                status: payment_request.aasm_state
               },
         status: :ok
 end

 def get_status_message(payment_request)
  case payment_request.aasm_state
  when :gateway_confirmation_pending
   'Payment is still under process, please wait until the status of transaction is updated'
  when :success
   'Payment was successful'
  else
   'Payment has failed'
  end
 end
end
```

### Step 10: Update Order Status

Based on response codes, update the payment status of order. Response codes and statuses can be found [Transaction Status API's Response codes and Messages section](https://developer.paytm.com/docs/transaction-status-api/) and [Transaction response codes and messages](https://developer.paytm.com/assets/Transaction%20response%20codes%20and%20messages.pdf)

```ruby
class PaymentRequest < ApplicationRecord
 ...

 private
 def update_status(response)
  response = response.symbolize_keys
  response_code = response[:RESPCODE]
  if response_code == "01"
   self.update(transaction_reference: response[:TXNID], metadata: response)
   self.mark_as_succeed!
  elsif response_code == "400" || response_code == "402"
  elsif response_code == "294"
   self.mark_as_expired!
  else
   self.update(transaction_reference: response[:TXNID], error_message: response[:RESPMSG], metadata: response)
   self.mark_as_failed!
  end
 end
end
```

### Step 11: Show Order Status on App

Based on server response, show messages on UI.

```kotlin
class ProfileFragment : Fragment() {
 private val paytmStateObserver: Observer<ActivityState> = Observer {
  when (it) {
   is ProfileViewModel.PaytmStatusState -> {
    when (it.updatePaymentResponse.status) {
     "success" -> {
                   Snackbar.make(progressBar, "Payment success", Snackbar.LENGTH_LONG).show()
                  }
     "failed" -> {
                  Snackbar.make(progressBar, "Payment failed. ${it.updatePaymentResponse.message}", Snackbar.LENGTH_LONG).show()
                 }
     "expired" -> {
                   Snackbar.make(progressBar, "Payment expired. Please try again", Snackbar.LENGTH_LONG).show()
                  }
     "gateway_confirmation_pending" -> {
                                        Snackbar.make(progressBar, "Payment pending. ${it.updatePaymentResponse.message}", Snackbar.LENGTH_LONG).show()
                                       }
     else -> {}
    }
   }
  }
 }
}
```

## References:-

1. [Add payments to your Android app with Paytm SDK](https://developer.paytm.com/docs/v1/android-sdk/)
2. [Paytm_App_Checksum_Kit_Ruby](https://github.com/Paytm-Payments/Paytm_App_Checksum_Kit_Ruby)
3. [Android Payment Gateway Integration Guide: PAYTM](https://medium.com/the-zalonin/android-payment-gateway-integration-guide-paytm-fa2ee01286e)
4. [Transaction Status API](https://developer.paytm.com/docs/transaction-status-api/)

## Some good reads you may like:-

1. [Override Devise Auth Token Controllers](https://nayan.co/blog/Ruby-on-Rails/override-devise-auth-token-controllers/)
2. [Generating Pdf in Ruby on Rails using Prawn](https://nayan.co/blog/Ruby-on-Rails/generating-pdf-in-ruby-on-rails/)
