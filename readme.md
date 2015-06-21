#简单的拍照
##请求相机权限
```xml
 <uses-feature android:name="android.hardware.camera2"
        android:required="true"/>
```
##使用系统相机程序拍照
```java
 public static final int REQUEST_IMAGE_CAPTURE = 0;
 public void capturePhotos(View view){
         Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
         if (intent.resolveActivity(getPackageManager()) != null){
             startActivityForResult(intent,REQUEST_IMAGE_CAPTURE);
         }
     }
```
##得到缩略图
```java
 @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK){
            Bundle bundle = data.getExtras();
            Bitmap bitmap = (Bitmap) bundle.get("data");
            iv.setImageBitmap(bitmap);
        }
    }
```
> 注意的是，这里通过`data`得到的Bitmap，适合于icon。下面的讲下保存全尺寸的图片。

##保存全尺寸图片
保存全尺寸的图片，一种是对所有的app可见，一种是对自己的程序可见。两者的路径不同。前者通过`Environment.getExternalStoragePublicDirectory()`，将`Environment.DIRECTORY_PICTURES`作为参数传入。
后者通过`getExternalFilesDir()`，这个方法在4.3一下的机器上，需要写入的权限，4.4后不再需要的，因为只对当前的应用可见。
> 注意的是：通过`getExternalFilesDir()`这个方法，用户卸载程序后，数据也被卸载。

以时间戳创建文件:
```java
 private File createImageFile() throws IOException {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(imageFileName, ".jpg", storageDir);
        mCurrentPhotoPath = image.getAbsolutePath();
        //   /mnt/sdcard/Pictures/JPEG_20150621_044540_-87380209.jpg
        Log.e("Test", "mCurrentPhotoPath:" + mCurrentPhotoPath);
        return image;
    }
```
现在修改拍照的逻辑：
```java
 public void capturePhotos(View view) {
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (intent.resolveActivity(getPackageManager()) != null) {
            File photoFile = null;
            try {
                photoFile = createImageFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
            if (photoFile != null) {
                intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(photoFile));
                //   file:///mnt/sdcard/Pictures/JPEG_20150621_044540_-87380209.jpg
                Log.e("Test","uri:" +  Uri.fromFile(photoFile).toString());
                startActivityForResult(intent, REQUEST_IMAGE_CAPTURE);
            }
        }
    }
```
##将图片添加到图库
```java
 private void addToGallery() {
        Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        File f = new File(mCurrentPhotoPath);
        Uri uri = Uri.fromFile(f);
        mediaScanIntent.setData(uri);
        this.sendBroadcast(mediaScanIntent);
    }
```
##解析图片
这部分主要用于避免OOM，这在之前的DisplayingBitmaps部分有详细的陈述。
```java
private void setPic() {
        int targetW = iv.getWidth();
        int targetH = iv.getHeight();
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(mCurrentPhotoPath, options);
        int photoW = options.outWidth;
        int photoH = options.outHeight;
        int scaleFactor = Math.min(photoW / targetW, photoH / targetH);
        options.inJustDecodeBounds = false;
        options.inSampleSize = scaleFactor;
        options.inPurgeable = true;
        Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, options);
        iv.setImageBitmap(bitmap);
    }
```