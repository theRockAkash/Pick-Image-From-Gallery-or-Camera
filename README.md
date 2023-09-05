# Pick-Image-From-Gallery-or-Camera Kotlin Android

#Manifest File
```
 <uses-feature
        android:name="android.hardware.camera"
        android:required="false" />
    <queries>
        <intent>
            <action android:name="android.media.action.IMAGE_CAPTURE" />
        </intent>
    </queries>
    <uses-permission android:name="android.permission.CAMERA" />
 <application
     ...
        >
 <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.example.app.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths"
                tools:replace="android:resource" />
        </provider>
</application>
```
#Add Service to Manifest for Backward compatibility for New Photo Picker

```
 <service
            android:name="com.google.android.gms.metadata.ModuleDependencies"
            android:enabled="false"
            android:exported="false"
            tools:ignore="MissingClass">
            <intent-filter>
                <action android:name="com.google.android.gms.metadata.MODULE_DEPENDENCIES" />
            </intent-filter>

            <meta-data
                android:name="photopicker_activity:0:required"
                android:value="" />
        </service>

```

#MainActivity.kt onCreate()
```
//call to show dialog to choose gallery or camera
showDialog()
```

#MainActivity.kt
```
private fun showDialog() {
        // setup the alert builder
        val builder = AlertDialog.Builder(this)
        val animals = arrayOf("Camera", "Gallery")
        builder.setItems(animals) { dialog, which ->
            // user checked an item
            when (which) {
                0 -> {
                    if (ContextCompat.checkSelfPermission(
                            this,
                            Manifest.permission.CAMERA
                        ) == PackageManager.PERMISSION_GRANTED
                    ) {
                        showCameraIntent()
                    } else {
                        requestPermissionLauncher.launch(Manifest.permission.CAMERA)
                    }
                }

                1 -> {
                    pickMedia.launch(PickVisualMediaRequest(ActivityResultContracts.PickVisualMedia.ImageOnly))
                }
            }

        }
        val dialog = builder.create()
        dialog.show()
    }

```

#For Camera

```
   private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            showCameraIntent()
        } else {
            Utils.toasty(this, "Camera permission denied")
        }
    }

  private var contentUri: Uri? = null
 private fun showCameraIntent() {

        // getContent.launch(Intent(MediaStore.ACTION_IMAGE_CAPTURE))
        val takePictureIntent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
        var photoFile: File? = null
        try {
            photoFile = createImageFile()
        } catch (ex: IOException) {
            println(ex)
        }
        contentUri = photoFile?.toUri()
        Log.w("TAG", "showCameraIntent: $contentUri")
        if (photoFile != null && photoFile.exists()) {
            val photoURI = FileProvider.getUriForFile(
                this,
                "com.example.app.fileprovider",
                photoFile
            )
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI)
            getContent.launch(takePictureIntent)
        }
    }
    private val getContent =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { uri: ActivityResult? ->
           contentUri?.let { uri ->
                this.contentResolver?.openInputStream(uri).use { inputStream ->
                    inputStream?.readBytes()?.let {
                          if(it.isNotEmpty()) {
                           uploadSignPic(it)
                          }
                    }
                }
            }
        }

```

#For Gallery

```
    private val pickMedia =
        registerForActivityResult(ActivityResultContracts.PickVisualMedia()) { uri ->
            if (uri != null) {
                val inputStream = this.contentResolver?.openInputStream(uri)
                val byteArray = inputStream?.readBytes()
                byteArray?.let {
                    uploadProfilePic(it)
                }

            } else {
                Log.d("PhotoPicker", "No media selected")
            }
        }

```

#uploadMediaToServer

```
    private fun uploadProfilePic(bytes: ByteArray) {

        val photoBody: MultipartBody.Part =
            MultipartBody.Part.createFormData(
                "file",
                "${System.currentTimeMillis()}.jpeg",
                bytes.toRequestBody("image/jpeg".toMediaTypeOrNull(), 0, bytes.size)
            )
       val map = HashMap<String, Int>()
        map["userId"] = userId
        viewModel.updateProfilePic(photoBody,map) //create a funtion in viewModel that will call api
    }
```

#Api 

```
  @Multipart
@POST("media/upload")
suspend fun updateProfilePic(
        @Part file: MultipartBody.Part,
        @PartMap map: Map<String, Int>
): Response<CommonResponse>
```
