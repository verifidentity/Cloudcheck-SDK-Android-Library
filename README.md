# Cloudcheck-SDK-Android-Library


Add the cloudcheck aar to your lib folder.

You'll need the following dependencies in your gradle

```
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
implementation "io.reactivex.rxjava2:rxjava:2.2.8"
implementation 'com.google.android.material:material:1.0.0'
implementation 'com.squareup.retrofit2:retrofit:2.5.0'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.5.0'
implementation "com.squareup.retrofit2:converter-moshi:2.5.0"
implementation("com.squareup.moshi:moshi:1.8.0")
implementation("com.squareup.moshi:moshi-adapters:1.8.0")
implementation "androidx.lifecycle:lifecycle-extensions:2.0.0"

```

Initialize the SDK

```(kotlin)
CloudcheckSdk.initialize("YOUR_API_KEY", "YOUR_API_SECRET")
```


Use the library like so:

```(kotlin)
val name = CCName("John",null, "Johnny")

val address = CCAddress("123 John St", null, "0101", "Johnston")

val driversLicence = CCNewZealandDriversLicence("AB000111", "002")

val dateOfBirth = CCDate("1994-04-04")

val details = CCVerifyDetails(name = name, address = address, dateOfBirth = dateOfBirth, newZealandDriversLicence = driversLicence)

val request = CCVerifyRequest(details, UUID.randomUUID().toString(), "Yes")

CloudcheckSdk.cloudcheckApi.verify(request, "my-user").subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(object : SingleObserver<VerifyResponse> {
        override fun onSuccess(t: VerifyResponse) {
            Log.d(TAG, "onSuccess: ${t}")
        }

        override fun onSubscribe(d: Disposable) {
            Log.d("TAG", "onSub")
        }

        override fun onError(e: Throwable) {
            Log.e("TAG", "error", e)
        }

    })
```

Run the live check by launching our activity

```(kotlin)

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    CloudcheckSdk.initialize("YOUR_API_KEY", "YOUR_API_SECRET")

    findViewById<Button>(R.id.start_button).setOnClickListener {
        val config = Config("This is an optional customisable complete message")
        val intent = CloudcheckSdk.liveIntent(this, CCLiveRequest(UUID.randomUUID().toString()), config)
        startActivityForResult(intent, 1)
    }
}


override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)

    when(resultCode) {
        COMPLETED -> {
            val cloudcheckResult = data!!.extras.getSerializable(CLOUDCHECK_RESULT) as CloudcheckResult
            Log.d(TAG, "We have a result: ${cloudcheckResult}")
        }
        CANCELLED -> {
            val cloudcheckResult = data!!.extras.getSerializable(CLOUDCHECK_RESULT) as CloudcheckResult
            Log.d(TAG, "We have a CANCELLED result: ${cloudcheckResult}")
        }
    }

}

```

To link the verification and the live check the verification reference must be captured from the verification response and passed to the live check request

```(kotlin)
val name = CCName("John",null, "Johnny")

val address = CCAddress("123 John St", null, "0101", "Johnston")

val driversLicence = CCNewZealandDriversLicence("AB000111", "002")

val dateOfBirth = CCDate("1994-04-04")

val details = CCVerifyDetails(name = name, address = address, dateOfBirth = dateOfBirth, newZealandDriversLicence = driversLicence)

val request = CCVerifyRequest(details, UUID.randomUUID().toString(), "Yes")

CloudcheckSdk.cloudcheckApi.verify(request, "my-user").subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(object : SingleObserver<VerifyResponse> {
        override fun onSuccess(t: VerifyResponse) {
            Log.d(TAG, "onSuccess: ${t}")
            
            findViewById<Button>(R.id.start_button).setOnClickListener {
                val config = Config("This is an optional customisable complete message")
                val intent = CloudcheckSdk.liveIntent(this, CCLiveRequest(UUID.randomUUID().toString(), t.verification.verificationReference), config)
                startActivityForResult(intent, 1)
            }
        }

        override fun onSubscribe(d: Disposable) {
            Log.d("TAG", "onSub")
        }

        override fun onError(e: Throwable) {
            Log.e("TAG", "error", e)
        }

    })
```
