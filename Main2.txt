package com.abdulrauf.myapplication;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.view.MenuItem;
import android.view.Menu;

import com.abdulrauf.myapplication.ui.login.LoginActivity;
import com.google.android.material.snackbar.Snackbar;
import com.google.android.material.navigation.NavigationView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import androidx.navigation.NavController;
import androidx.navigation.fragment.NavHostFragment;
import androidx.navigation.ui.AppBarConfiguration;
import androidx.navigation.ui.NavigationUI;
import androidx.drawerlayout.widget.DrawerLayout;
import androidx.appcompat.app.AppCompatActivity;

import com.abdulrauf.myapplication.databinding.ActivityMainBinding;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.messaging.FirebaseMessaging;

public class MainActivity2 extends AppCompatActivity {

    private static final int REQUEST_NOTIFICATION_PERMISSION = 1001;

    private AppBarConfiguration mAppBarConfiguration;
    private ActivityMainBinding binding;
    private NavController navController;
    private FirebaseAuth mAuth;  // اضافه کردن FirebaseAuth

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // مقداردهی FirebaseAuth
        mAuth = FirebaseAuth.getInstance();

        // چک کردن وضعیت ورود
        FirebaseUser currentUser = mAuth.getCurrentUser();
        if (currentUser == null) {
            // اگر کاربر وارد نشده باشد، به صفحه ورود هدایت می‌شود
            Intent intent = new Intent(MainActivity2.this, LoginActivity.class); // به صفحه ورود بروید
            startActivity(intent);
            finish();
            return;
        }

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        setSupportActionBar(binding.appBarMain.toolbar);

        binding.appBarMain.fab.setOnClickListener(view -> {
            Intent intent = new Intent(MainActivity2.this, TawkActivity.class);
            startActivity(intent);
        });

        DrawerLayout drawer = binding.drawerLayout;
        NavigationView navigationView = binding.navView;

        NavHostFragment navHostFragment = (NavHostFragment) getSupportFragmentManager()
                .findFragmentById(R.id.nav_host_fragment_content_main);
        if (navHostFragment != null) {
            navController = navHostFragment.getNavController();
        } else {
            throw new IllegalStateException("NavHostFragment یافت نشد.");
        }

        // اضافه کردن آیتم‌های جدید به AppBarConfiguration
        mAppBarConfiguration = new AppBarConfiguration.Builder(
                R.id.nav_home,R.id.nav_gallery2,R.id.nav_slideshow2,R.id.nav_about,R.id.nav_services,R.id.nav_prices)
                .setOpenableLayout(drawer)
                .build();

        NavigationUI.setupActionBarWithNavController(this, navController, mAppBarConfiguration);
        NavigationUI.setupWithNavController(navigationView, navController);

        // نمایش دادن آیتم‌های مربوط به نقش جدید
        if (navigationView.getMenu() != null) {
            navigationView.getMenu().findItem(R.id.nav_gallery).setVisible(false);
            navigationView.getMenu().findItem(R.id.nav_slideshow).setVisible(false);
            navigationView.getMenu().findItem(R.id.nav_service).setVisible(false);
            navigationView.getMenu().findItem(R.id.nav_price).setVisible(false);
        }

        // مجوز اعلان‌ها برای Android 13+
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS)
                    != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.POST_NOTIFICATIONS}, REQUEST_NOTIFICATION_PERMISSION);
            } else {
                subscribeToTopic();
            }
        } else {
            subscribeToTopic();
        }
    }

    private void subscribeToTopic() {
        FirebaseMessaging.getInstance().subscribeToTopic("my_topic")
                .addOnCompleteListener(task -> {
                    // لاگ یا toast در صورت نیاز
                });
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_NOTIFICATION_PERMISSION) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                subscribeToTopic();
            } else {
                Snackbar.make(findViewById(android.R.id.content), "اجازه اعلان رد شد", Snackbar.LENGTH_SHORT).show();
            }
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            try {
                openSmsAppWithDraftMessage();
            } catch (Exception e) {
                e.printStackTrace();
                Snackbar.make(findViewById(android.R.id.content), "مشکلی در ارسال پیام پیش آمده است.", Snackbar.LENGTH_LONG).show();
            }
            return true;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    public boolean onSupportNavigateUp() {
        return navController != null && NavigationUI.navigateUp(navController, mAppBarConfiguration)
                || super.onSupportNavigateUp();
    }

    @Override
    public void onBackPressed() {
        if (navController != null && navController.getCurrentDestination() != null
                && navController.getCurrentDestination().getId() == R.id.nav_home) {
            new AlertDialog.Builder(this)
                    .setTitle("خروج")
                    .setMessage("آیا می‌خواهید از برنامه خارج شوید؟")
                    .setPositiveButton("خروج", (dialog, which) -> finish())
                    .setNegativeButton("ماندن", null)
                    .setIcon(android.R.drawable.ic_dialog_alert)
                    .show();
        } else if (navController != null) {
            navController.popBackStack();
        } else {
            super.onBackPressed();
        }
    }

    private void openSmsAppWithDraftMessage() {
        String message = "سلام، من از اپ برای رزرو نوبت آرایشگاه استفاده می‌کنم. اگه دوست داری تو هم نصبش کن!";
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_SUBJECT, "DevCenter");
        shareIntent.putExtra(Intent.EXTRA_TEXT, message);

        try {
            startActivity(Intent.createChooser(shareIntent, "معرفی از طریق"));
        } catch (Exception e) {
            throw new RuntimeException("خطا در ارسال پیام", e);
        }
    }
}