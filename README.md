<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Tekstinio įvedimo laukas -->
    <EditText
        android:id="@+id/editTextInfo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Įrašykite informaciją čia" />

    <!-- Mygtukas, skirtas įrašyti ar patvirtinti duomenis -->
    <Button
        android:id="@+id/buttonSubmit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Pateikti" 
        android:layout_marginTop="16dp"/>
</LinearLayout>
