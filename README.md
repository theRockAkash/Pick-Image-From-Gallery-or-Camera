# Pick-Image-From-Gallery-or-Camera



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

 private fun showCameraIntent() {
        val takePicture = Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        getContent.launch(takePicture)
    }
    private val getContent =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { uri: ActivityResult? ->
            uri?.data?.extras?.let {
                try {
                    val bitmap = it["data"] as Bitmap?
                    val stream = ByteArrayOutputStream()
                    bitmap?.compress(Bitmap.CompressFormat.JPEG, 100, stream)
                    val bytes = stream.toByteArray()
                    uploadProfilePic(bytes)
                } catch (e: Exception) {
                    e.printStackTrace()
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
